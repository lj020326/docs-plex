# Plex Media Permissions for Linux Noobies

There is no problem with being a noobie and I do not use the term to sligtht or disparage anyone.

This is **a way** to setup your permissions for running Plex in Linux. Different folks may use different methods.

The permissions concepts provided here apply to OSX, but the users and groups are controlled and modified differently, so much of this will not work properly. I think the command is `dscl`, but that could be out of date.

There are many ways to setup your permissions scheme in Linux, this methodology describes **a way** to do it, not everyone will like it, but it works for me, so whatever.

This is meant to be a super quick guide, please do some research on your own to help you with these terms, you're also welcome to reply with fix requests and/or suggestions.

## DON'T!

* Set your user and group for your media files to `root`.
* Set your permissions to `777`.

## Permissions and Ownership Commands

1. `chmod` - change file modes or Access Control Lists
2. `chown` - change file owner and group
3. `chgrp` - change group ownership

If you want more information on these or most commands in Linux, use `man`:

    man chmod
    man chown
    man chgrp

## Permissions & Access Control Brief

This is a super simplified description of Linux user and permission sets, I implore you to read up on this subject yourself.

After installing Plex, it will create a `plex` user and a `plex` group for itself.  This user and group will need to have at least read access to your media files for them to show up in your library, you can give it write access if you want to be able to delete stuff from the GUI.

There are several types of access controls we're going to be using with Plex.

### User

Basically a user is either a login account or a system account.  User accounts can login to the computer, system accounts do stuff on the computer but generally can't login.

To view your user do:

	whoami

To view all users do:

	cat /etc/passwd

### Group

Basically a group is a set of one or more users, groups can allow multiple users to have permissions to a file or directory.

To view your groups do:

	groups

To view all groups do:

	cat /etc/group

### World / Everyone / Other

The world or everyone may have certain permissions to certain files, you rarely should give this level write permissions to anything.

### Permissions

Permissions are the levels of access users, groups, or world has, they are broken up into three sets:

* `r` - Read - The ability to read a file or a directory.
* `w` - Write - The ability to create or delete a file in a directory or write to a file.
* `x` - eXecute - The ability to execute a command in a file or directory.

You can see the permissions of something by doing:

	ls -la

Here's an example from my server with the description of what the stuff is.

	drwxr-xr-x    2 plex plex 20480 Sep 20 12:53 A
	^^            ^ ^    ^    ^     ^            ^
	||            | |    |    |     |            |
	||            | |    |    |     |            +-> name of the path or file
	||            | |    |    |     +--------------> modification date
	||            | |    |    +--------------------> file size or size of the contents
	||            | |    +-------------------------> owners group
	||            | +------------------------------> owners name
	||            +--------------------------------> number of links
	|+---------------------------------------------> permissions of the path or file
	+----------------------------------------------> directory / file / link / other

The permissions are setup in 3 groups:

	rwxr-xr-x
	^  ^  ^
	|  |  |
	|  |  +-> World Permissions - Read / eXecute
	|  +----> Group Permissions - Read / eXecute
	+-------> Owner Permissions - Read / Write / eXecute

So in the example above, we can tell the user is `plex`, the group is `plex`, and the permissions are `rwxr-xr-x`.  This means:

* User `plex` can read, write, or execute stuff in the directory.
* Group `plex` can read or execute stuff in the directory.
* World can read or execute stuff in the directory.

**Permissions can be set with 3 sets of numbers 0-7.**

The sets define the user / group / world:

* First digit is for owner
* Second digit is for group
* Third digit is for world or everyone

The number defines the type of permission:

| Num | Permission  | Description            |
|:---:|:-----------:|------------------------|
|  0  |    `---`    | No permissions         |
|  1  |    `--x`    | Executable Only        |
|  2  |    `-w-`    | Write Only             |
|  3  |    `-wx`    | Write & Execute        |
|  4  |    `r--`    | Read Only              |
|  5  |    `r-x`    | Read & Execute         |
|  6  |    `rw-`    | Read & Write           |
|  7  |    `rwx`    | Read & Write & Execute |

**Permissions can also be set with letters.**

Pick what you want to give permissions to:

* `u` - user
* `g` - group
* `o` - other/world/everyone

Add or remove the permission:

* `-` - remove
* `+` - add

Add with what kind of permission:

* `r` - read
* `w` - write
* `x` - execute

Combine them:

    chmod u+x filename
    chmod g+x filename
    chmod o+x filename

## Plex Media Permissions

I keep all of my plex media in a path on my server, the path is a mount of a hardware RAID5.  I don't recommend any particular file system or RAID setup, I do recommend using what you want to learn; for example my new system uses various drives in using `btrfs`.

My drives are mounted to the `/dvr/mediastore` path, mounting is outside of the scope of this document, you can search for *How to mount a device in Linux?* for more information.

I'll be referring to my personal directory structure from here on out, you will need to substitute your structure to get this stuff to work correctly.

In my setup I have Music, Movies, TV Shows as such:

	/dvr/mediastore/Music
	/dvr/mediastore/Movies
	/dvr/mediastore/TV

The permission sets are:

* `/dvr` - Owner: `root`, Group: `root`, Permissions: `755`
* `/dvr/mediastore` - Owner: `plex`, Group: `plex`, Permissions: `775`

Everything in `mediastore` is owned by `plex` and has the group `plex`, all the files are set to `xxx`

### Directory Structure

In movies I have a set of folders, my TV and Music are setup similarly.

	/dvr/mediastore/Movies/#
	/dvr/mediastore/Movies/A
	/dvr/mediastore/Movies/B
	/dvr/mediastore/Movies/C
	### you can figure it out ###
	/dvr/mediastore/Movies/X
	/dvr/mediastore/Movies/Y
	/dvr/mediastore/Movies/Z

In each folder I keep movies which have titles starting with the listed character, with the exception of movies which start with *The* which I use the first letter of the second word, thus *The Matrix (1999)* is stored in `M`. Movies which start with numbers are all stored in `#` as they are uncommon.

## Set Up

Let's setup from scratch.

* Download and Install Plex - [https://www.plex.tv/media-server-downloads/](https://www.plex.tv/media-server-downloads/)

* Add your user to the plex group, I'm guesssing your name is probably not `pjobson`.

		sudo usermod -a -G plex pjobson

* Setup your drives and mount them if they're not.

* Create some directories.

		sudo mkdir -p /dvr/mediastore/Movies/#
		sudo mkdir -p /dvr/mediastore/Movies/A
		sudo mkdir -p /dvr/mediastore/Movies/B
		sudo mkdir -p /dvr/mediastore/Movies/C
		sudo mkdir -p /dvr/mediastore/Movies/D
		sudo mkdir -p /dvr/mediastore/Movies/E
		# ...etc...
		# same convention for for TV & Music

* Setup the OWNERSHIP of the paths.

		sudo chown -R plex.plex /dvr/mediastore/Movies
		sudo chown -R plex.plex /dvr/mediastore/TV
		sudo chown -R plex.plex /dvr/mediastore/Music

* Setup the permissions of the paths.

		sudo find /dvr/mediastore/Movies -type d -exec chmod 775 {} \;
		sudo find /dvr/mediastore/TV -type d -exec chmod 775 {} \;
		sudo find /dvr/mediastore/Music -type d -exec chmod 775 {} \;

* Go to the plex webgui [http://localhost:32400](http://localhost:32400) and/or [http://127.0.0.1:32400](http://127.0.0.1:32400).

* Setup your Plex and tell it where the libraries are.

* ???

* PROFIT!

Seriously that is it.  With this setup you can add media with either your `plex` user or your personal user account.

## Maintence

When copying files and directories around be sure to keep your permissions up to date.

Directories should be `775` or `rwxrwxrx-` and files should be `664` or `rw-rw-r--`.

I have my process automated, but what it would look like if it were manual would be:

Copy the movie recursively to the correct path:

	cp -r "Movie I Totally Own (1999)" /dvr/mediastore/Movies/M/

Change the permissions of the directory of the movie to rwxrwxrw-

	chmod 775 "/dvr/mediastore/Movies/M/Movie I Totally Own (1999)"

Change the permissions of the files in the movie's path to rw-rw-r--

	chmod 664 "/dvr/mediastore/Movies/M/Movie I Totally Own (1999)/*"

Change the ownership of the movie and all contents using recursively to plex and the group to plex.

	sudo chown -R plex.plex "/dvr/mediastore/Movies/M/Movie I Totally Own (1999)"

I automate this process by creating a cron job which runs every fifteen minutes and checks the permissions and modifies them as needed.

### Cron Script


You can save the script below as `plexown.sh` or whatever you want, then add a cronjob for root to run it automatically every 10 minutes or so.  You'll also want to chmod the script to executable.  I save my scripts in `/home/pjobson/bin`, so that's what I'm using in this example.  You can of course put yours wherever you like.

    chmod 755 plexown.sh
    sudo crontab -e

Select nano or whatever editor you like, then add this where `*/10` means run every 10 minutes:

    */10 * * * * /home/pjobson/bin/plexown.sh >/dev/null 2>&1

To break it down:

    */10 * * * * /home/pjobson/bin/plexown.sh >/dev/null 2>&1
     ^   ^ ^ ^ ^ ^                            ^
     |   | | | | |                            |
     |   | | | | |                            +-> Send output to /dev/null
     |   | | | | +------> Path to your script, you'll want to change this. 
     |   | | | +--------> Run on every day of the week
     |   | | +----------> Run on every month of the year
     |   | +------------> RUn on every day of the month
     |   +--------------> Run on every hour of the day
     +------------------> Run every 10 minutes

#### Script

    #!/bin/bash
    
    # chmod files
    find /dvr/media/Movies           -type f \! -perm 664 -exec chmod 664 {} \; -print
    find /dvr/media/TV               -type f \! -perm 664 -exec chmod 664 {} \; -print
    find /dvr/media/Music            -type f \! -perm 664 -exec chmod 664 {} \; -print
    
    # chmod directories
    find /dvr/media/Movies           -type d \! -perm 775 -exec chmod 775 {} \; -print
    find /dvr/media/TV               -type d \! -perm 775 -exec chmod 775 {} \; -print
    find /dvr/media/Music            -type d \! -perm 775 -exec chmod 775 {} \; -print
    
    # chown everything
    find /dvr/media/Movies           \! -user plex -exec chown plex.plex {} \; -print
    find /dvr/media/TV               \! -user plex -exec chown plex.plex {} \; -print
    find /dvr/media/Music            \! -user plex -exec chown plex.plex {} \; -print
