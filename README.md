# snapshotter
Python script to execute rsnapshot using the required retain-level as needed
making it possible to create a new snapshot at will, without any concern to
which retain-level must be used.


## Description
rsnapshot is configured to use one or more "retain-levels". Each specifies
the number of snapshots a given level will retain.

The first retain-level is assumed to be the most frequent and each successive
level gets "pulled up" from the level below it. So if we configured rsnapshot
to retain 7 "daily", 4 "weekly", and 6 "monthly" snapshots, every seventh
snapshot would become a weekly snapshot, and *of those*, every fourth snapshot
would become a monthly snapshot.

Typically, rsnapshot execution is automated using cron. This ensures that
snapshots are created to a regular schedule, e.g.:
 * retain daily 7
 * retain weekly 4
 * retain monthly 6

For this to work, the crontab needs to ensure the larger retain level is run
_before_ the lower level. In the above example, "monthly" must be run before
"weekly" which in turn must be run before "daily".


## Motivation
Slightly more burdensome and the motivation for this script, is when using
rsnapshot in a desktop environment where the computer isn't running 24/7.

In this situation, the user would be required to know which rsnapshot levels
to use, and must run them in the correct order.

FWIW: anacron is not ideal here as it can't run anything more frequently than
once per day, and if you don't use your computer daily, anacron will run all
the missed snapshots at once.

This script provides the user with a single command to execute without any
concern for which rsnapshot levels need to be run.


## Implementation
See Python source for details. The short description is, snapshotter reads
the retain-levels from the rsnapshot configuration file and stores the product
of these as the `cycle_total`. It then tracks the current snapshot as
`cycle_index` and tests that against a moduli dictionary for each retain level
to know which rsnapshot level needs to be executed.


## Installation
`snapshotter` is a single-file Python script and can be installed by copying
it to a location in your `$PATH`. E.g.: `/usr/local/bin`

NOTE: You _will_ need to configure rsnapshot first. You can use a separate
rsnapshot configuration file and specify that on the command line, otherwise
the default `/etc/rsnapshot.conf` file will be parsed.


## Usage
```
usage: snapshotter [-h] [-c rsnapshot_config_file] [--cycle-file path] [--inhibit hours]
                   [--symlink-dir path] [--symlink-suffix relative-path] [-n] [-t] [-v] [-D] [-q]
                   [--version]

Execute appropriate rsnapshot command(s) based on current point in retain cycle

options:
  -h, --help            show this help message and exit
  -c rsnapshot_config_file, --config rsnapshot_config_file
  --cycle-file path     file used to retain current cycle count
  --inhibit hours       number of hours to prevent another snapshot being run
  --symlink-dir path    directory into which symlinks to each snapshot will be created
  --symlink-suffix relative-path
                        relative path from rsnapshot-root to target directory
  -n, --dryrun          dryrun, show shell commands that would be executed
  -t                    alias for --dryrun
  -v, --verbose         verbose, increase verbosity
  -D                    debug, outputs debugging/diagnostic info
  -q                    quiet, surpress non-fatal warnings
  --version             show program's version number and exit
```

The `-q` and `-D` are identical to rsnapshot and are also passed to the
rsnapshot command(s) being executed. Verbosity is slightly different to
rsnapshot:

 * `-v` enables verbose snapshotter output
 * `-vv` enables verbose snapshotter and rsnapshot output
 * `-vvv` enables verbose snapshotter and more verbose rsnapshot output

The `--inhibit` option is useful if you set snapshotter run execute via cron
`@reboot`. In this situation, you may wish to ensure new snapshots are not
created in quick succession by preventing new runs within (say) 10 hours of
the last

The `--symlink-dir` and (optionally) `--symlink-suffix` options can be used to
create symlinks directly to snapshots based on their time-stamp.

