+++
title = "Migrate issues from GitHub to gitlab using CLI" 
date = 2024-08-16
[extra]
toc = true
+++


## Prerequisites

For this guide you would need both [`gh`] and [`glab`] installed in your system
and logined too, including GNU core utils like: `xargs`.

[`gh`]: https://cli.github.com/
[`glab`]: https://docs.gitlab.com/ee/editor_extensions/gitlab_cli/

## Quering issues in GitHub

Particularly in my case, I was intereseted in migrating only titles and descriptions
(bodies in GitHub's terminology). This could easily be done using `--json` flag:

```shell
gh issue list --json title,body
```

## Creating Issue in GitLab

For that `glab` has `issue create` command with `--title` and `--description` flags:

```shell
glab -R owner/repo issue create --title "Issue title" --description "Issue description"
```

The problem here, is that we will pipe output of `gh` to `glab`, when `glab` accepts
only position arguments. That's where `xargs` come in handy:

```shell
echo '"Issue title" "Issue description"' | xargs -l bash -c 'glab -R owner/repo issue create --title "$0" --description "$1"'
```

> Note that in MacOS you need to use `xargs` with `-L1` flag instead of `-l`:

Also this works with arguments that have newlines in it:

```shell
echo '"Issue title" "Issue description\r\nanother line"' | xargs -l bash -c 'glab -R owner/repo issue create --title "$0" --description "$1"'
```
