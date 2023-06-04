## Time Machine for Developers

## The problem

Time Machine will backup your files. For a developer, and having loads of `node_modules` and `vendor` directories for projects, this is huge time suck and waste since those can be brought back with `npm`, `yarn`, `composer`, etc.

## The solution

Directories can be excluded from Time Machine using the cli.

### Adding exclusions

First, verify that you can get a list of directories that you want to exclude:

```
cd "$HOME/code" && find $(pwd) \
    -maxdepth 3 \
    -type d \( -name vendor -o -name node_modules \)
```

The first part of the above assumes you keep your code in a `code` directory in your home directory. You'll need to change that if that's not the case for you.

Executing this command should output a list of directories that have `vendor` or `node_modules` directories. These are what you want excluded from Time Machine.

Next, we can add to the above command to pipe those into the Time Machine utility tool, `tmutil`, to be excluded.

```
cd "$HOME/code" && find $(pwd) \
    -maxdepth 3 \
    -type d \( -name vendor -o -name node_modules \) \
    -prune \
    -exec tmutil addexclusion {} \; \
    -exec tmutil isexcluded {} \;
```

### Viewing exclusions

You can see all excluded files/directories with the following command:

```
sudo mdfind "com_apple_backup_excludeItem = 'com.apple.backupd'"
```

You can see if a specific file or directory is excluded with:

```
tmutil isexcluded /some/file/path.txt
```

Verify that directories we excluded earlier are in fact exluded:

```
cd "$HOME/code" && find $(pwd) \
  -maxdepth 3 \
  -type d \( -name vendor -o -name node_modules \) \
  -exec tmutil isexcluded {} + | grep -F "[Excluded]"
```

### Remove exclusions

Or remove all the exclusions we added

```
cd "$HOME/code" && find $(pwd) \
  -maxdepth 3 \
  -type d \( -name vendor -o -name node_modules \) \
  -prune \
  -exec tmutil removeexclusion {} \; \
  -exec tmutil isexcluded {} \;
```

### Automate

We can automate this so if we create a new project we don't have to worry about manually adding the exclusion with a cron job.

Create a bash file that will run the `addexclusion` commands:

```
#!/usr/bin/env bash

cd "$HOME/code" && find $(pwd) \
  -maxdepth 3 \
  -type d \( -name vendor -o -name node_modules \) \
  -prune \
  -exec tmutil addexclusion {} \; \
  -exec tmutil isexcluded {} \;
```

Remember to change the `$HOME/code` to wherever you keep your projects. I put this at `$HOME/code/machine-utils/time-machine-exclusions.sh`.

Make sure it's executuble:

```
chmod +x $HOME/code/machine-utils/time-machine-exclusions.sh
```

Add this as a cronjob to run every hour:

- `crontab -e`
- press `i` to go into "insert" mode
- paste the following line (Change according to where you saved the file on your machine. The full path is required.):
  - `0 * * * * /Users/davidadams/code/machine-utils/time-machine-exclusions.sh`
- press `esc` to exit "insert" mode
- type `:x` to save and exit

Categories: [Mac](https://programmingarehard.com/blog/categories/Mac)

Tags: [Mac](https://programmingarehard.com/blog/tags/Mac), [Time Machine](https://programmingarehard.com/blog/tags/Time Machine)

Next: [Career advice and server monitoring](https://programmingarehard.com/2022/06/14/career-advice-and-server-monitoring.html/)Previous: [Throw exceptions in repositories](https://programmingarehard.com/2015/03/13/exceptions-in-repositories.html/)