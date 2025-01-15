---
layout: single
title:  "Setting up zsh on an HPC environment"
date:   2025-01-15 05:00:00 +0800
classes: wide
toc: true
categories:
- personal
- all_posts
permalink: /personal/2025-01-15-setting-up-zsh-hpc
---

# Setting up zsh on an HPC environment

I have `zsh` set up locally, but my production code runs in an HPC environment.

There's an argument that you should set up your environment in a way that makes it
as easy to use as possible. I've already set up tooling like `zsh` and `warp` locally,
but the production environment is a bit of pain to use.

So, I finally, at long last, set up `zsh` on the HPC environment.

This is annoying to do and I'll likely forget how to do it again, so I'm writing this
down to save future Mark the trouble.

The first step is installing `zsh` on the HPC environment from source (since you
don't have root access on the HPC environment). [This](https://stackoverflow.com/questions/15293406/install-zsh-without-root-access) StackOverflow post was very helpful,
and I just needed to copy and paste the commands of the top-rated answer.

Once that was set up, I needed to switch from the default bash shell to zsh. [This](https://stackoverflow.com/questions/10341271/switching-from-zsh-to-bash-on-os-x-and-back-again) StackOverflow post was also very helpful.

Finally, I needed to get this to work on startup, so that whenever I connected to the
remote environment from my IDE, the default shell would be `zsh`. To do this, I added
the following to my `~/.bash_profile` file (courtesy of Claude):

```bash
# Only source bashrc if we're actually in bash
if [ -n "$BASH_VERSION" ]; then
    # Get the aliases and functions
    if [ -f ~/.bashrc ]; then
        . ~/.bashrc
    fi
fi

# Check if zsh exists and we're not already running it.
# Goal is to run zsh on startup instead of bash
if [ -x "$(command -v zsh)" ] && [ "$SHELL" != "$(command -v zsh)" ]; then
    export SHELL="$(command -v zsh)"
    exec zsh -l
fi
```

Now, whenever I connect to the HPC environment, it starts using `zsh` instead of `bash`. I personally find `zsh` to be much easier to use, since it includes helpful features like syntax highlighting, auto-completion, and a bunch of other stuff.

![Example zsh shell on HPC](/assets/images/2025-01-15-setting-up-zsh-hpc/example_shell.png)
