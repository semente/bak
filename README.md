# bak -- simple, efficient, and encrypted backups

The purpose of **bak** is to be a simple, secure, easy to set up, and
use backup solution for GNU/Linux.

Although not tested on BSDs, Mac OS X, cygnus and the like, it should
work fine in any machine with *GNU coreutils, bash, rsync* and
*GnuPG*-feel free to send pull requests to increase compatibility and
ease of deploy in systems other than GNU/Linux.

It is also required that you have a PGP key configured **and know how
to handle GnuPG**.

The initial version of this tool was written in less than a hour and I
would keep it simple. I do not intend to add more features or let the
code bloated, but any good idea is very welcome. Use the link
https://github.com/semente/bak/issues to suggest any changes.


## Usage

To run `bak` you must have these softwares installed in your computer:

  - Tar and Gzip
  - GNU coreutils
  - rsync
  - GnuPG
  - OpenSSH client (ssh), if you need push backups to a remote host

### Synopsys

Local:

```sh
$ bak DEST
```

Via remote shell:

```sh
$ bak [USER@]HOST:DEST
```

(destination operand follows the same syntax as in `rsync`)


### Description

*To-Do*

I recommend you to read the source code, it is very short and clean.


### Usage examples

Backup current directory, also sign encrypted backup with PGP key and
exclude files bigger than 1M (1024kb):

```sh
$ SIGN=1 MAX_FILE_SIZE=1024 bak user@remote:backups/`hostname -s`
```

Force full backups and use a specific recipient encryption key:

```sh
$ rm .bak.last && RCPT_KEY=ABCD1234 bak /in/black
```


## Settings

Settings are defined by environment variables. The simplest way is the
inline way:

```sh
$ DEBUG=1 bak user@remote:backups
```

You can also source it from a file:

```sh
$ ( . ~/.bakrc && bak /in/black )
```

### Reference

####DEBUG

Set to non-nil to read commands but do not execute them.

####VERBOSE

Set to non-nil to tar, gpg and rsync run verbosily and to print to
default output the commands and their arguments as they are executed.

####SIGN

Also sign the encrypted backup file (GnuPG's `--sign` argument).

####SIGN_KEY

Key to be used to sign the encrypted backup file.

####RCPT_KEY

Key to be used to encrypted the backup file.

####MAX_FILE_SIZE

Exclude files bigger than `MAX_FILE_SIZE` (in kbytes). Very useful for
faster uploads if your target and most important files are smalls.


### Exclude file

*bak* will exclude from backup patterns (Tar syntax) listed in the
file `.bakignore` of the current directory. It will also exclude any
directory that contains the file `.nobackup`.

Check out the exclude file below, to ignore all `.git` directories I
had to start the pattern with a wildcard "*" as in `*/.git` but to
ignore just the directory `.local` from the root of the backup source
I had to use `./.local`. In this case, using `/home/user/.local` won't
work.

```sh
# my ~/.bakignore file
*~
*/.git
*/.tox
./.cache
./.local
./.virtualenvs
./Downloads/Torrent
```

*bak* do not respects `.bakignore` files from subdirectories, i.e., it
just load the `./.bakignore` file.


## Scheduled backups

I do not have any scheduled backup in my laptop, I have a script that
remember me from time to time to backup but you might find interesting
schedule your backups by using *cron*. Following is example I wrote
but didn't have time to test it yet, so please make sure it is running
fine before trust in your backups.

```sh
SHELL=/bin/bash

# bak preferences
RCPT_KEY=ABCD1234
#SIGN=1  # if the PGP key has a passphrase you might have problems here
#SIGN_KEY=ABCD1234
MAX_FILE_SIZE=10240  # prevent upload of big files (>10M), they aren't my target

# m  h    dom mon dow   command
#
# backup all files in $HOME every Sunday
0    0    *   *   0     cd $HOME && rm .bak.last && /path/to/bak user@remote:bak/`hostname -s`/
#
# backup all files in $HOME that size are less than 4M (4096kb), daily
0    0    *   *   *     cd $HOME && MAX_FILE_SIZE=4096 rm .bak.last && /path/to/bak user@remote:bak/`hostname -s`/
#
# backup files in $HOME that are newer than `.bak.last' and size less than 1M, every 6 hours
*/6  *    *   *   *     cd $HOME && MAX_FILE_SIZE=1024 /path/to/bak user@remote:bak/`hostname -s`/
```
