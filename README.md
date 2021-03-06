# bak &ndash; simple, efficient, and encrypted backups

The purpose of **bak** is to be a simple, secure, and easy to set up
and use encrypted backup solution for GNU/Linux. It is designed for
personal and desktop backups&mdash;mainly for your laptop when roaming
around&mdash;but you can also find it useful in other situations.

Although not tested on BSDs, Mac OS X, cygnus and the like, it should
work fine in any machine with *GNU coreutils, findutils, grep, bash,
GnuPG* and *rsync* installed. Feel free to send pull requests to
increase compatibility and ease of deploy in systems other than
GNU/Linux.

It is also required that you have a PGP key configured **and know how
to handle GnuPG**.

## Usage

To run `bak` you must have the following software installed on your
computer:

  - Bash
  - GNU Tar and Gzip
  - GNU coreutils (`dirname`, `mktemp`, `rm`, `touch`)
  - GNU findutils (`find`)
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

Bak creates a backup of the current directory to one or more `DEST`
in local or remote hosts. Backups are always encrypted and signed by
GnuPG&mdash;you must have a PGP key configured for the user running `bak`.

**Incremental backups are done by default** and are based on the last
modification time of the last backup performed; i.e. only files
modified after the latest backup detected will be included into the
backup archive. However, you can force a full backup by just providing
the option `-f`.

### Usage examples

**Backup current directory and exclude files bigger than 1M (1024kb):**

```sh
$ bak -s1024 user@remote:backups/`hostname -s`
```

If a previous backup is detected, an incremental backup will
occur. You must use the option `-f` to force a full backup, if
desired, as shown in the next example.

**Force full backups and use a specific recipient encryption key:**

```sh
$ bak -f -rABCD1234 /in/black
```

If you don't have the options `default-recipient-self` and
`default-key` configured in your `gpg.conf`, using `-r` option will
prevent GnuPG from asking you for the recipient key for
encryption. However, if the option `-k` is not used the default key to
sign with is the first key found in the secret keyring.

**Backup multiple directories to multiple destinations:**

```sh
$ for dir in /etc /root $HOME
> do
>     cd $dir && bak /var/backups/$dir /media/usb0/backups/`hostname -s`/$dir
> done
```

#### Other options

```
  -f                      force a full backup
  -r RECIPIENT-KEY        encrypt for user id RECIPIENT-KEY
  -k SIGN-KEY             use SIGN-KEY as the key to sign with
  -s SIZE                 don't backup any file larger than SIZE kbytes
  -i IDENTIFY-FILE        selects a file for public key authentication over SSH
  -u                      non interactive-commonly used for unattended operations
  -v                      verbose (i.e. give more information during processing)
  -d                      print commands as they are executed (for debug)
  -V                      print version number
  -h                      show this help text
```

The option `-u` basically tells GnuPG to use the options *--batch
--tty* (see *gpg(1) manpage* for more information).

### Exclude file

Bak will exclude from backup patterns (Tar syntax) listed in the
file `.bakignore` of the current directory. It will also exclude any
directory that contains the file `.nobackup`.

Bak does not respect `.bakignore` files from subdirectories, i.e.,
it just load the `./.bakignore` file.

An example of what could be a `~/.bakignore` file:

```sh
*~
*.mp3
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

You might want to schedule your backups by using *cron* or other
tool. Once Bak **always signs** the resulting backup archive it will
prompt you for the PGP secret key password if your it is protected by
one, and it may be an inconvenient for unattended backups. I chose to
not support unsigned backups for security reasons. So, for unattended
operations, it is recommended that the PGP key used for sign (option
`-k`) is not protected by a password&mdash;you can still select a
password-protected key for encryption with the option `-r`.

Similarly, you might want use a non password-protected SSH identity
for public key authentication over SSH. Use the option `-i` to select
the desired SSH private key. You can also set the environment variable
`RSYNC_RSH` for the same purpose or to tweak your SSH connections (see
*rsync(1) manpage* for more information).

Here is an *crontab* example:

```sh
SHELL=/bin/bash
PATH="/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin"

# bak setup
BAK_ARGS="-u -r ABCD1234 -k ABCD1234 -i ~/.ssh/id_rsa.bak"
BAK_DEST="user@remote:bak/"
#RSYNC_RSH="ssh -p2222 -i somekey"


# m  h    dom mon dow   command
#
# full $HOME backup at 3AM on the 1st day of every month
0    3    1   *   *     cd ~ && bak -f $BAK_ARGS $BAK_DEST
#
# backup files that are newer than `.bak' and size are not larger than 1M;
# runs at 2AM every day
0    2    *   *   *     cd ~ && bak -s1024 $BAK_ARGS $BAK_DEST
```

# Testing

Bak use the command line tester tool
[clitest](https://github.com/aureliojargas/clitest) to perform
automatic testing.

Once `clitest` is installed in your system, just run following
commmand:

```sh
$ clitest test.txt
```

# Known issues

- It is recommended that you do full backups (`-f`) if you have
  changed your `.bakignore` file. Bak may not include in next
  incremental backups files related to removed or modified ignore file
  patterns. A similar issue will occur when using the option `-s`.
