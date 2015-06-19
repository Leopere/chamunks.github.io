---
layout: post
title: Backups with duply
tagline: Easy backups with duply
category: blog
tags:
  - post
  - blog
  - linux
  - desktop
  - server
  - backup
  - duplicity
---

{% include JB/setup %}

# A frontend for Duplicity
[Duply][1] is _a frontend for the mighty [duplicity][2] magic_ and a really nifty one. Anybody that has used duplicity for backups may have noticed two things: how powerful and versatile a tool it is, and how tricky it can be to configure a backup scheme.

First of all, let's just talk a little bit about the backend, that is, Duplicity. As the [Wikipedia article][3] nicely points out, Duplicity provides encrypted, versioned, remote backups that require very little of the remote server, in fact, it just needs for the server to be accessible via any of the supported protocols (FTP, SSH, Rsync, etc).

# Configuring Duply
The first step required to use Duply is the creation of a backup profile. This can be accomplished by running `duply <profile> create` where <profile> is whatever name we want for the profile.

This creates a configuration file called `~/.duply/<profile>/conf` that we will edit. The configuration file is quite well documented but I will break down the main points.

There are several settings we should take into account when configuring Duply:
- [**Encryption**](#encryption): whether we need our backups to be encrypted, either with symmetric encryption or with a key and passphrase;
- [**Location**](#location): where to save our backup to, either a remote server or a local folder;
- [**Source**](#source): the directory we want to back up (we can exclude files to avoid backing up garbage);
- [**Age**](#age): how long our old backups should be kept.

## Encryption
There are two types of encryption Duply can use (unless we just disable it altogether), both with pros and cons.

Encryption with GPG keys is self-explanatory. You use a GPG key to encrypt each volume of the backup and both the GPG key and the passphrase are needed to decrypt the backup, giving you extra security. This also means that, if you lose the GPG key file, you will **not** be able to recover your backup, therefore you have to make sure that the `~/.gnupg/` directory is copied somewhere else and not just inside the backup... Trust me, it's happened to me:

{% highlight sh %} GPG_KEY='ADD274FA'        # Use 'gpg --list-keys' to see your keys GPG_PW='VeryStrongPass'   # Passphrase of the key {% endhighlight %}

Symmetric encryption is simplier in that it only uses a single password for encryption, meaning you can recover your backup so long as you remember this password. Obviously, it is less secure than using a key, since it's subject to bruteforce attacks:

{% highlight sh %}

# GPG_KEY='ADD274FA'       # Comment out this line
GPG_PW='ItBetterBeStr0ng' # Password to use {% endhighlight %}

## Location
Now we have to configure where Duply will save our backups. In the `conf` file there are several examples for all the supported protocols. In my case I will use FTP:

{% highlight sh %} TARGET="ftp://ftpuser:ftppass@server/$USER@$HOSTNAME" {% endhighlight %}

Notice that, if you use shell environment variables (`$USER`, `$HOSTNAME`, etc) you have to use double quotes instead of the default single quotes, otherwise the substitution won't expand.

## Source
Usually, as normal users, we would want to backup our home directory and exclude those directories/files with an exclude list. This can be done with Duply by changing the following setting in the `conf` file:

{% highlight sh %} SOURCE="$HOME" {% endhighlight %}

Again, notice the double quotes for variable substitution.

For system backups, since we can only specify one source, we should use the root folder and use exclude lists:

{% highlight sh %} SOURCE='/' {% endhighlight %}

### Excluding files
Once we have determined our source for backups, we should filter out files or directories that would make our backups too big. We do this by creating the file `~/conf/<profile>/exclude` and listing the files inside. Thankfully, these lists accept default [Unix globbing][4]. For reference, this is what's in my exclude file:

{% highlight sh %} **/_[Cc]ache_ **/_[Hh]istory_ **/_[Ss]ocket_ **/_[Tt]humb_ **/_[Tt]rash_ **/_[Bb]ackup **/_.[Bb]ak **/*[Dd]ump **/_.[Ll]ock **/_.log **/*.part **/_.[Tt]mp **/_.[Tt]emp **/*.swp **/_~ **/.adobe **/.cache **/.dbus **/.fonts **/.gnupg/random_seed **/.gvfs **/.kvm **/.local/share/icons **/.macromedia **/.obex **/.rpmdb **/.thumbnails **/.VirtualBox **/.wine *_/Downloads {% endhighlight %}

As you can see, you can specify both wildcards or certain directories/files.

It's worth noting that, even though the file is called `exclude`, it can be used to include files too. For instance, if we used the root directory as source (`SOURCE='/'`) as we talked about before, we can exclude all files except certain directories like so:

{% highlight sh %}
- /etc
- /root
- /var/lib/mysql
- /var/mail
- /var/spool/cron
- /var/www
- **
- {% endhighlight %}

That last line would tell Duply to ignore all files except those listed previously and preceded by a plus sign.

Since version v0.5.14 of Duply, there is another way to exclude directories from the backup. By creating a file called `.duplicity-ignore` inside a directory, we will force Duply to ignore this directory recursively. To enable this, we will have to uncomment these lines in our configuration file `~/.duply/<profile>/conf`:

{% highlight sh %} FILENAME='.duplicity-ignore' DUPL_PARAMS="$DUPL_PARAMS --exclude-if-present '$FILENAME'" {% endhighlight %}

## Age
Finally, we can determine the age of the backups we keep when we run the purge commands. There are a couple of settings here depending on the way we make backups.

This setting tells Duply to keep backups up to a certain time (for example 6 weeks) when we run `duply <profile> purge`:

{% highlight sh %} MAX_AGE=6W {% endhighlight %}

This other one tells Duply to keep a number of full backups when we run `duply <profile> purge-full`:

{% highlight sh %} MAX_FULL_BACKUPS=2 {% endhighlight %}

However, the most useful one for me, is the setting that uses the `--full-if-older-than` option of duplicity to automatically make a full backup when the previous full backup is older than a certain age:

{% highlight sh %} MAX_FULLBKP_AGE=1W DUPL_PARAMS="$DUPL_PARAMS --full-if-older-than $MAX_FULLBKP_AGE " {% endhighlight %}

## Scheduling backups
Finally, after everything is configured, we should run a backup to test everything is alright with the command `duply <profile> backup`. This might take a while since, not having any previous backup, it will execute a full backup.

After that, we can check the status of our backups by running `duply <profile> status`, which would give us something like this:

{% highlight console %}

## Found primary backup chain with matching signature chain:
Chain start time: Tue Apr 17 14:48:54 2012 Chain end time: Wed Apr 18 14:01:33 2012 Number of contained backup sets: 1 Total number of contained volumes: 52  Type of backup set:                            Time:      Num volumes:

```
            Full         Tue Apr 18 14:48:54 2012                52
```

--------------------------------------------------------------------------------

No orphaned or incomplete backup sets found. --- Finished state OK at 15:46:34.122 - Runtime 00:00:03.495 --- {% endhighlight %}

That looks cool and everything but we cannot rely on our memory to remember when we should make a backup. That's why we should schedule our backups using [cron][5] (or [anacron][6], or [fcron][7]) and leave the heavy lifting to them.

We can either specify a time for both a full and an incremental backup, like this:

{% highlight console %} @daily    duply <profile> backup_verify @weekly   duply <profile> full_verify_purge --force {% endhighlight %}

This will run and verify a daily incremental backup and a weekly full backup. Also, it will purge old backups weekly after completing and verifying the full backup.

However, if we configured Duply to use the `--full-if-older-than` option of duplicity like discussed above, we can just run a single command:

{% highlight console %} @daily    duply <profile> backup_verify_purge --force {% endhighlight %}

This is extremely useful for laptops and boxes that are not on 24x7.

### Pre and post scripts
Another basic requirement for any backup solution is the option to run certain commands both before and after the backup is executed. Duply, of course, has this too and will run any command inside the file `~/.duply/<profile>/pre` before the backup and any command inside `~/.duply/<profile>/post` after the backup.

This is useful to lock and flush databases before backup and unlocking them afterwards, maybe even make a [LVM snapshot][8] for consistent and quick backup. Or just to gather any other information that needs to be backed up too (f.e. installed packages, Delicious bookmarks, etc).

### Live backups
There are some drawbacks to using the system while the backup is being run. An obvious one is the impact on performance, since the backup is using the disks.

Also we have the fact that, if the backups take a while, which is very likely to happen, and the files are modified in the meantime, the verification will **fail**. That doesn't mean the backup has failed but the verification obviously will.

For this, I would recommend either a [LVM snapshot][8] as suggested above which, let's face it, is not very likely to be done on anything other than a server; or we can just disable the verification and use [ionice][9] like so:

{% highlight console %} @daily    ionice -c3 duply <profile> backup_purge --force {% endhighlight %}

This will execute the backup with low I/O priority, which means we will be able to use the computer without much impact, and cron will still send us an email with the output of the command so we can confirm that the backup was done properly.

[1]: http://duply.net/ "Duply"
[2]: http://duplicity.nongnu.org/ "Duplicity"
[3]: https://en.wikipedia.org/wiki/Duplicity_%28software%29 "Wikipedia: Duplicity (software)"
[4]: https://en.wikipedia.org/wiki/Globbing#Syntax "Wikipedia: Globbing - Syntax"
[5]: http://man.cx/cron "Cron manpage"
[6]: http://anacron.sourceforge.net/ "Anacron"
[7]: http://fcron.free.fr/ "Fcron"
[8]: http://tldp.org/HOWTO/LVM-HOWTO/snapshots_backup.html "13.4. Taking a Backup Using Snapshots"
[9]: http://man.cx/ionice "Ionice manpage"
