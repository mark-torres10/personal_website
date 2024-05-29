---
layout: single
title:  "Accidentally deleting prod data (and the importance of backups)"
date:   2024-05-29 04:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/importance-of-backups-2024-05-28
---
# Accidentally deleting prod data (and the importance of backups).

## What went wrong
I ran a cron job to collect posts from Bluesky over the course of the past week. I needed to migrate the posts from their old schema to a new schema, so I ran a database migration. When I went to inspect the results, I saw that there were about ~900,000 rows.
![Old database counts](/assets/images/2024-05-28-importance-of-backups/old_db_counts.png)

But when I came back to look at the database again a few hours later, I saw that I only had 5,000 rows.
![New database counts](/assets/images/2024-05-28-importance-of-backups/new_db_counts.png)

What happened? I was worried that I accidentally deleted the database, and I thought that I could turn to my backup table to restore the data. Turns out that was gone as well. I also checked by connecting to sqlite directly in the command line to verify what I was seeing, and no matter what I tried, I kept seeing the same row count.

What happened? Turns out, I didn't use `git pull` very intelligently. Git overwrites, especially when forced, can be dangerous. When I wanted to fetch the latest code from my remote Github repo, I decided to overwrite the current repo via `git pull`. But as it turns out, I saved an old version of the database in version control. When I did a forced overwrite, Git did a force overwrite not only of my code, but also the database. Plus, since I hadn't committed the new SQLite DB to version control (Github begins throwing fits at around n~1M rows due to file sizes).

Were there backups or other things that I could've done to restore the data?
- SQLite backups? I overwrote the actual underlying file itself and the changes weren't handled by SQLite, so there was no undo. Plus, broadly speaking, there's no "undo" anyways in SQLite, since once a transaction is committed then it is treated as final.
- Git-related backups? I never stored the data in version control due to file limit constraints.
- Cluster-related backups? I know that many institutional compute clusters store backups of data, but according to [this](https://services.northwestern.edu/TDClient/30/Portal/KB/PrintArticle?ID=1542) and [this](https://services.northwestern.edu/TDClient/30/Portal/KB/ArticleDet?ID=1546), KLC only backs up certain directories, and my project directory doesn't fall in those categories.

Theoretically I could've pushed back the cursor to the start of last week, but that'll lead to too many posts to backfill in a short period of time plus I was already only collecting 1-2 hours worth of posts per day. In lieu of this, I'll just restart data collection, but to make sure that this doesn't happen again, I'll build an automated system to back up any data.

## Steps to fix it

### Looking at existing third-party options
There are a few third-party options that I saw (via ChatGPT), such as [DBeaver](https://github.com/dbeaver/dbeaver) and [DB Browser](https://sqlitebrowser.org/). But, it looks like these are just wrappers that would automate the task of writing the data as some converted and compressed file format on a certain schedule, so given how straightforward the task is, I'd rather just built it myself instead of relying on more dependencies.

### Create automated script to back up databases and restore from backups

First, I created a script to back up my databases. This loads the databases as pandas dataframes and then writes them to the respective format (which here will be either parquet or JSON).

#### Backing up the databases

This is the script to back up the databases.

```python
"""Back up SQL databases.

Stores in a new directory, `bluesky_research_data_backups`, outside of Git
version control.

Requires pyarrow and fastparquet.

The resulting filesystem will look something like this.

    - bluesky_research_data_backups
        - <db_name>
            - <timestamp>
                - parquet
                    - <table_name>.parquet
                - json
                    - <table_name>.json
        - <db_name>
            - <timestamp>
                - parquet
                    - <table_name>.parquet
                - json
                    - <table_name>.json
        - <db_name>
            - <timestamp>
                - parquet
                    - <table_name>.parquet
                - json
                    - <table_name>.json
"""
import gzip
import os
import sqlite3

import pandas as pd

from lib.constants import current_datetime_str

current_file_directory = os.path.dirname(os.path.abspath(__file__))

root_directory = os.path.abspath(
    os.path.join(current_file_directory, '../../../..')
)
root_data_backups_directory_name = "bluesky_research_data_backups"
root_data_backups_directory = os.path.join(
    root_directory, root_data_backups_directory_name
)

def load_db_filepaths(directory: str = current_file_directory) -> dict:
    """Loads all DB filepaths in the directory. Looks for '.db' extension.

    Returns a dictionary whose key = db filename and whose value = db filepath.
    """
    res = {}
    for root, _, files in os.walk(directory):
        for file in files:
            if file.endswith(".db"):
                res[file] = os.path.join(root, file)
    return res


def backup_db(
    db_filepath: str, export_formats: list[str], compressed: bool = True
):
    # strip ".db" from name (e.g., "foo.db" -> "foo")
    db_folder_name = os.path.basename(db_filepath)[:-3]

    # full folder path = <root backups directory>/<db folder name>/<timestamp>
    export_directory = os.path.join(
        root_data_backups_directory, db_folder_name, current_datetime_str
    )

    print(f"Exporting {db_filepath} as {' and '.join(export_formats)} to {export_directory}...") # noqa
    os.makedirs(export_directory, exist_ok=True)

    conn = sqlite3.connect(db_filepath)
    cursor = conn.cursor()

    cursor.execute("SELECT name FROM sqlite_master WHERE type='table';")
    tables = cursor.fetchall()

    for table_name in tables:
        table_name = table_name[0]
        df = pd.read_sql_query(f"SELECT * FROM {table_name}", conn)
        for export_format in export_formats:
            if export_format == "parquet":
                print(f"Exporting table {table_name} as parquet...")
                parquet_export_dir = os.path.join(export_directory, "parquet")
                parquet_filepath = os.path.join(
                    parquet_export_dir, f"{table_name}.parquet"
                )
                os.makedirs(parquet_export_dir, exist_ok=True)
                compression = 'gzip' if compressed else None
                df.to_parquet(parquet_filepath, compression=compression)
                print(f"Finished table {table_name} as parquet to {parquet_filepath}...") # noqa
            elif export_format == "json":
                print(f"Exporting table {table_name} as JSON...")
                json_export_dir = os.path.join(export_directory, "json")
                json_filepath = os.path.join(json_export_dir, f"{table_name}.json")
                os.makedirs(json_export_dir, exist_ok=True)
                json_str = df.to_json(orient='records', lines=True)
                if compressed:
                    with gzip.open(json_filepath + '.gz', 'wt', encoding='utf-8') as gz_file:
                        gz_file.write(json_str)
                else:
                    with open(json_filepath, 'w', encoding='utf-8') as json_file:
                        json_file.write(json_str)
                print(f"Finished table {table_name} as JSON to {json_filepath}...")

    conn.close()


def main(export_formats: list[str] = ["parquet", "json"]):
    db_filepaths: dict = load_db_filepaths()
    for db_filename, db_filepath in db_filepaths.items():
        print(f"Backing up {db_filename}...")
        backup_db(db_filepath=db_filepath, export_formats=export_formats)
        print(f"Backed up {db_filename}.")
    print("Completed backing up all SQL databases.")


if __name__ == "__main__":
    main()
```

When we run this, we can see that the backup is successful:

```plaintext
(bluesky_research) [jya0297@quser31 sql]$ python backup_databases.py 
Backing up synced_record_posts.db...
Exporting /projects/p32375/bluesky-research/lib/db/sql/synced_record_posts.db as parquet and json to /projects/p32375/bluesky_research_data_backups/synced_record_posts/2024-05-29-09:14:42...
Exporting table dbmetadata as parquet...
Finished table dbmetadata as parquet to /projects/p32375/bluesky_research_data_backups/synced_record_posts/2024-05-29-09:14:42/parquet/dbmetadata.parquet...
Exporting table dbmetadata as JSON...
Finished table dbmetadata as JSON to /projects/p32375/bluesky_research_data_backups/synced_record_posts/2024-05-29-09:14:42/json/dbmetadata.json...
Exporting table subscriptionstate as parquet...
Finished table subscriptionstate as parquet to /projects/p32375/bluesky_research_data_backups/synced_record_posts/2024-05-29-09:14:42/parquet/subscriptionstate.parquet...
Exporting table subscriptionstate as JSON...
Finished table subscriptionstate as JSON to /projects/p32375/bluesky_research_data_backups/synced_record_posts/2024-05-29-09:14:42/json/subscriptionstate.json...
Exporting table transformedrecordwithauthor as parquet...
Finished table transformedrecordwithauthor as parquet to /projects/p32375/bluesky_research_data_backups/synced_record_posts/2024-05-29-09:14:42/parquet/transformedrecordwithauthor.parquet...
Exporting table transformedrecordwithauthor as JSON...
Finished table transformedrecordwithauthor as JSON to /projects/p32375/bluesky_research_data_backups/synced_record_posts/2024-05-29-09:14:42/json/transformedrecordwithauthor.json...
Backed up synced_record_posts.db.
Backing up filtered_posts.db...
Exporting /projects/p32375/bluesky-research/lib/db/sql/filtered_posts.db as parquet and json to /projects/p32375/bluesky_research_data_backups/filtered_posts/2024-05-29-09:14:42...
Exporting table filteredpreprocessedposts as parquet...
Finished table filteredpreprocessedposts as parquet to /projects/p32375/bluesky_research_data_backups/filtered_posts/2024-05-29-09:14:42/parquet/filteredpreprocessedposts.parquet...
Exporting table filteredpreprocessedposts as JSON...
Finished table filteredpreprocessedposts as JSON to /projects/p32375/bluesky_research_data_backups/filtered_posts/2024-05-29-09:14:42/json/filteredpreprocessedposts.json...
Backed up filtered_posts.db.
Completed backing up all SQL databases.
```

#### Restoring from backups

This is the script to restore from backups.

```python
def load_from_backup(
    db_name: str, timestamp: str, table_name: str, export_format: str
) -> pd.DataFrame:
    """Loads a table from a backup.

    Args:
        db_name: The name of the database.
        timestamp: The timestamp of the backup.
        table_name: The name of the table.
        export_format: The format of the backup (e.g., "parquet", "json").

    Returns:
        A pandas DataFrame.
    """
    backup_directory = os.path.join(
        root_data_backups_directory, db_name, timestamp, export_format
    )
    if export_format == "parquet":
        return pd.read_parquet(os.path.join(backup_directory, f"{table_name}.parquet"))
    elif export_format == "json":
        with open(os.path.join(backup_directory, f"{table_name}.json"), 'r') as f:
            return pd.read_json(f, lines=True)
    else:
        raise ValueError(f"Unsupported export format: {export_format}")
```

We can test it with this script and verify that the resulting dataframe exists:
```python
def _test_load_data():
    """Tests loading data from a backup."""
    db_name = "filtered_posts"
    timestamp = "2024-05-29-09:14:42"
    table_name = "filteredpreprocessedposts"
    export_format = "parquet"
    df = load_from_backup(db_name, timestamp, table_name, export_format)
    print(df.head())
    assert df.shape[0] > 0

if __name__ == "__main__":
    _test_load_data()
```

We can see that this looks correct:
```plaintext
(bluesky_research) [jya0297@quser31 sql]$ python backup_databases.py 
   id                                                uri                                                cid  ... filtered_by_func        synctimestamp preprocessing_timestamp
0   1  at://did:plc:sgsx73ohiqfsb5feqdllh3we/app.bsky...  bafyreiehv35bv3vjlrhzlgzii4hizoc7wyuohlj6aux2q...  ...             None  2024-05-22-21:00:36     2024-05-28-18:16:05
1   2  at://did:plc:rje4snbb7obj6twr4gji7ssm/app.bsky...  bafyreicbhewj43k32kj7e26dp6heukxu2zrnd5ywjl5es...  ...             None  2024-05-22-21:00:36     2024-05-28-18:16:05
2   3  at://did:plc:p6yxwwdkw6huohjymf3whhx3/app.bsky...  bafyreia72xnzsdz7ghh46p2inlkv2bvpw74efqoreqfxi...  ...             None  2024-05-22-21:00:36     2024-05-28-18:16:05
3   4  at://did:plc:e2ch2347axhuxzfnssqek22k/app.bsky...  bafyreibpmngn2dpvp7ffdnv472qfhsdhlgue4iycxtpjg...  ...             None  2024-05-22-21:00:36     2024-05-28-18:16:05
4   5  at://did:plc:xrr5j2okn7ew2zvcwsxus3gb/app.bsky...  bafyreiftf634skp57bmea5nkpernfheyyg47mrbjczfac...  ...             None  2024-05-22-21:00:36     2024-05-28-18:16:05

[5 rows x 13 columns]
```


### Running backups as a cron job

Now that we have a script that can back up our databases, I want to run this on a cron job.

```bash
#!/bin/bash

# This bash script will create a cron job to back up the SQL DBs every 8 hours.
# It will execute the following command:
#   - python backup_databases.py

# Get the current directory
DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# load conda env
CONDA_PATH="/hpc/software/mamba/23.1.0/etc/profile.d/conda.sh"

# set pythonpath
PYTHONPATH="/projects/p32375/bluesky-research/:$PYTHONPATH"

# cron expression for running every 8 hours
CRON_EXPRESSION="0 */8 * * *"

# Define the cron job command
CRON_JOB="$CRON_EXPRESSION source $CONDA_PATH && conda activate bluesky_research && export PYTHONPATH=$PYTHONPATH && cd $DIR && python backup_databases.py >> /projects/p32375/bluesky-research/lib/log/logfile.log"

# Add the cron job to the current user's crontab
(crontab -l 2>/dev/null; echo "$CRON_JOB") | crontab -

echo "Cron jobs created to automatically run database backups."
```

Now this wil automatically back up the databases every 8 hours.

We can see it in the list on the cron jobs:
```plaintext
(bluesky_research) [jya0297@quser31 sql]$ crontab -l
0 */8 * * * source /hpc/software/mamba/23.1.0/etc/profile.d/conda.sh && conda activate bluesky_research && export PYTHONPATH=/projects/p32375/bluesky-research/:/projects/p32375/bluesky-research/:$PATH && cd /projects/p32375/bluesky-research/lib/db/sql && python backup_databases.py >> /projects/p32375/bluesky-research/lib/log/logfile.log
```

## Conclusion

Deleting prod is never fun, but is a rite of passage. This was a very small example of that (I've done worse accidental data deletions in industry, oops), but it is a case study of always making sure to have disaster recovery measures enabled. Given how simple my current use case is, a simple Python script, run on a cron schedule, is enough for this use case. This means that if I accidentally delete data again (which is likely to happen), I can avoid this mistake from happening again.
