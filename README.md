# bak -- simple, efficient, and encrypted backups

The purpose of **bak** is to be a simple, secure, and easy to set up
and use encrypted backup solution for GNU/Linux. It is designed for
personal and desktop backups-mainly for your laptop when roaming
around-but you can also find it useful in other situations.

I wrote this script because I found *duplicity* and other alternatives
for encrypted backups too complicated for my purposes.

Although not tested on BSDs, Mac OS X, cygnus and the like, it should
work fine in any machine with *GNU coreutils, findutils, grep, bash,
GnuPG* and *rsync* installed. Feel free to send pull requests to
increase compatibility and ease of deploy in systems other than
GNU/Linux.

It is also required that you have a PGP key configured **and know how
to handle GnuPG**.

The initial version of this tool was written in less than an hour and
I kept it very simple. I do not intend to add more features or let the
code become bloated, but any good ideas are very welcomed. Use the
link https://github.com/semente/bak/issues to suggest any changes.


## Usage

To run `bak` you must have the following software installed on your
computer:

  - Bash
  - GNU Tar and Gzip
  - dirname, mktemp, rm, touch (GNU coreutils)
  - find (GNU findutils)
  - GNU grep
  - rsync
  - GnuPG
  - openssh-client (optional, to send backups to a remote host)
  - logger (optional, to make entries in syslog)

### Synopsis

```sh
bak [OPTION]... [[USER@]HOST:]DEST...
```

### Description

*bak* creates a backup of the current directory to one or more `DEST`
in local or remote hosts. Backups are always encrypted and signed by
GnuPG (PGP)-you must have a personal key configured for the user
running `bak`.

**Incremental backups are done by default** and are based on the last
modification time of the file `.bak` (*lastfile*) located in the
current directory. This file is automatically created and its
modification time is updated after every successful backup. However,
you can force a full backup by just providing the option `-f`.

### Usage examples

**Backup current directory and exclude files bigger than 1M (1024kb):**

```sh
$ bak -s1024 user@remote:backups/`hostname -s`
```

If there is a previous backup (i.e. there is a `.bak` file in the
current directory) the next backup will ignore any files older than
the *lastfile* `.bak`. You must use the option `-f` to force a full
backup, as shown in the next example.

**Force full backups and use a specific recipient encryption key:**

```sh
$ bak -f -rABCD1234 bak /in/black
```

If you don't have the options `default-recipient-self` and
`default-key` configured in your `gpg.conf`, using `-r` option will
prevent GnuPG from asking you for the recipient key for encryption.

#### Other options

You can get a full list of options by accessing the built-in help:

```sh
$ bak -h
```

### Exclude file

*bak* will exclude from backup patterns (Tar syntax) listed in the
file `.bakignore` of the current directory. It will also exclude any
directory that contains the file `.nobackup`.

*bak* does not respect `.bakignore` files from subdirectories, i.e.,
it just load the `./.bakignore` file.

An example of what could be a `~/.bakignore` file:

```sh
*~
*/.git
*/.tox
./.cache
./.local
./.virtualenvs
./Downloads/Torrent
```

In the example above, to ignore all `.git` directories I had to start
the pattern with a wildcard "\*" as in ``*/.git``, but to ignore just
the directory `.local` from the root of the backup source I had to use
`./.local`. In this case, using `/home/user/.local` won't work.

## Scheduled and unattended backups

I do not have any scheduled backups on my laptop, I have a script that
reminds me from time to time to backup, allowing me to decide if it is
a good time for a backup and what options I should use considering
mainly the current Internet connection.

However, you might want to schedule your backups by using *cron*. For
unattended operations, it is recommended that you use the option `-u`,
it basically tells GnuPG to use the options *--batch --tty* (see
*gpg(1) manpage* for more information).

The following is an example I wrote but didn't have time to test yet,
so please make sure it is running fine before trusting in your
backups.

```sh
SHELL=/bin/bash

# m  h    dom mon dow   command
#
# backup all files in $HOME every Sunday
0    0    *   *   0     cd $HOME && /path/to/bak -uf -rABCD1234 user@remote:bak/`hostname -s`/
#
# backup all files in $HOME that size are not larger than 4M (4096kb), daily
0    0    *   *   *     cd $HOME && /path/to/bak -uf -s4096 -rABCD1234 user@remote:bak/`hostname -s`/
#
# backup files in $HOME that are newer than `.bak' and size are not larger than 1M, every 6 hours
*/6  *    *   *   *     cd $HOME && /path/to/bak -u -s1024 -rABCD1234 user@remote:bak/`hostname -s`/
```

# Known issues

- It is recommended that you do full backups (`-f`) if you have
  changed your `.bakignore` file. *bak* won't include a removed or
  modified pattern from this file that was changed before *lastfile*
  (`.bak`) in the next incremental backup. A similar issue will occur
  when using the option `-s`.

- When sending backups to multiple destinations, *lastfile*'s (`.bak`)
  last modification time won't be updated if *bak* failed for at least
  one destination. It means that the next incremental backup will have
  duplicates files in the destinations that were updated
  successfully. It is not a big deal.

- dry run (`-n`) option is not implemented yet
