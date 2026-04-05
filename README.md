
# 🦟 mosquito-hive-v2

Parallel toolkit for mass rebuilding, preparing and testing ALT Linux packages.

This project is inspired by the original [mosquito-hive](https://github.com/greno4ka/mosquito-hive), but it has been completely rewritten from scratch to meet new requirements and tasks. It’s mainly intended for anyone, who needs to make batch changes to packages and rebuild them.

## Background

Maintaining of Python 3 in ALT Linux long time required rebuilding a large number of dependent packages. To make this easier for maintainers, an original set of scripts has been written. While they worked for a while, using and maintaining them was always a challenge.
Some of the implemented functionality turned out to be unnecessary, while the truly essential features had to be added in a rather crude way. This project represents an attempt to organize the necessary functionality in an order.

## Purpose

`hive` is the main entry point that operates parallel processing of packages.
Each processing scenario is implemented as a separate script `mosquito[1-6]`.

### Helper Scripts

Also this project provides several scripts to prepare and maintain the environment:

- **0initialize** – This script should be run first to initialize hashers before work
- **1clean** – Use this script to ensure, that there are no built RPMs in hashers
- **2sync_packages** – Run this script after building or loading target package(s) inside your hasher to ensure that all hashers have the same environment.

### Key scenarios

**1. Rebuilding dependent packages**

Suppose you have built Python 3 or another package with many dependencies. To ensure all dependent packages rebuild correctly:

-   Initialize your hashers
-   Load or synchronize your built RPMs
-   Run `hive` with `mosquito1`

The result: two directories containing logs of *successful* and *failed* builds.

**2. Preparing a mass NMU (Non-Maintainer Upload)**

-   Clone your git repositories into a folder, e.g., `$HOME/repos`
-   Create a preparation script, e.g., `nmu.sh`:
```bash
#!/bin/bash

pill() {
    spec="$(gear-rules-print-specfile 2>/dev/null)"
    sed -i 's,Development/Python,Development/Python3,' $spec
    srpmnmu --spec $spec --changelog "- Fix package group." 2>&1
    gear-commit -a --no-edit -q
}
```
- Run `hive` with `mosquito2`

You can specify your repos directory or path to your script with environment variables.

While this could be done with a simple bash loop, running it in parallel significantly speeds up the process for hundreds of repositories.

**3. Building packages from git repositories**

After enhancing your repositories, check if they build successfully using `mosquito3`. You can also specify the repository directory for convenience.

**4. Testing installation of built packages**

Finally, verify that your built packages are installable (or identify missing dependencies) with `mosquito4`. Failed builds will not install and will show a `NOTFOUND` diagnostic.

**5. Check your git repos for freshness**

To see if any packages were updated since your last changes, run `hive` with `mosquito5`.

**6. Reset your prepared updates**

For debugging your NMU script, you may need to clear previous attempts. Run `hive` with `mosquito6` to reset all changes.

## Usage

    ./hive <mode> < package_list

## Modes

| Mode | Script    | Description                     |
|------|-----------|---------------------------------|
| 1    | mosquito1 | Rebuild source RPMs             |
| 2    | mosquito2 | Prepare git repositories        |
| 3    | mosquito3 | Build packages from git repos   |
| 4    | mosquito4 | Install built packages          |
| 5    | mosquito5 | Check git repos for updates     |
| 6    | mosquito6 | Sync/clean git repositories     |

## Environment Variables

You can override default paths:

##### For mosquito2 and mosquito3:
    REPOS=/path/to/repos

##### For mosquito2:
    PILLFILE=/path/to/pill

## Dependencies

Make sure the following tools are installed:

- `gear`
- `hasher`
- `parallel` (note: not moreutils-parallel)
- `altlinux-repolist-utils` (required for mosquito1)
- `sshfs` (required for mosquito1 for remote builds)
