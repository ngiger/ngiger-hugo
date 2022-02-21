---
title: "Borgbase Backup"
date: 2022-02-18T17:12:08+01:00
draft: false
---

# Motivation and overview

I am always concerned whether I have a backup of my data, as life teached
that things go wrong when you do not expect it.

My requirements were
  * somewhere in my local network
  * somewhere outside of my local network (eg. a fire, broken dam destroy my house)
  * encrypted
  * open source tools
  * moderate size 50GB -> 1 TB

After looking at several alternatives I decided to use borbackup and the offerings
of borgbase.com.

## Backup to locally accessible filessystem

I used just a different partition, but I could have also used a NFS-mount or
sshfs on a host inside my LAN or somewhere else.

This leads to this simple configuration, where I just want to backup my
home directory


```nix
{pkgs, ... }:
{
# To list contents use
# sudo borg list /opt/backup_niklaus/
# sudo borg list /opt/backup_niklaus/::librem-backupToOpt-2021-06-17T14:00:43
  services.borgbackup.jobs = {
      backupNiklaus = {
        paths = [ "/home/niklaus" ];
        doInit = false;
        repo = 	"/opt/backup_niklaus" ;
        encryption = {
          mode = "none";
        };
        compression = "auto,lzma";
        startAt = "hourly";
        exclude = (import ./borg_excludes.nix).borg_excludes; # ++ [ "pp:/home/niklaus/work";
      };
  };
}
```

The borg_excludes.nix file has the following content
```
{
  borg_excludes = [
        "/nix"
        "*.pyc"
        "**/tmp"
        "**/.cache"
        "**/.metadata"
        "**/.m2/repository" # but backup m2 configuration
        "**/.mozilla"
        "**/.vagrant.d"
        "**/gnome-boxes/images"
        "**/*.vdi"
        "**/Downloads"
        "**/downloads"
        "**/vendor"
        "**/*.tmp"
        "**/*.log"
        "**/*.bz2"
        "**/*.zst"
        "**/*.tgz"
        "**/*.zip"
        "**/*.gz"
        "**/*.deb"
        "**/repository"
        "**/gems"
        "**/libvirt/qemu/save"
        "chromium"
        "**/*~"
        "*gnucash*.gnucash"
        "**/Dropbox"
        "**/Trash"
        "**/*.iso"
        "**/.local/share/akonadi"
        "**/snap"
        "**/share/flatpak"
        "**/share/baloo"
        "**/.eclipse"
        "**/.gem"
        "**/.p2/pool"
        "**/.var"
        "**/*.img"
    ];
}
```
This has patterns to exclude all VM-images, *.deb/tmp/log files, snaps, flatpak,
rubygems under vendor, gnucash backups etc.

## Backup to borgbase.com

First I did set up a job to backup my /etc and /home directories to borgbase.
In order to keep everything (including passwords) in my configuration I decided
to use the `encription.passphrase` option. I could also read it from a local
file from the host running the borgbackup job, by setting `encription.passCommand`
like this  `passCommand = "cat /etc/nixos/.keys/borgbackup_passphrase";`.

The ID `j1234siy0@j1234siy0.repo.borgbase.com:repo` was taken from my borgabase.com
setup of a newly created repository.

Also I add a `preHook` to ensure that my laptop has internet connection by
trying to ping borgbase.com.

The snippet looks like this

```nix
{pkgs, ... }:
{
  programs.ssh.extraConfig = "Host *.repo.borgbase.com\n  StrictHostKeyChecking=accept-new" ;
  services.borgbackup.jobs = {

    Librem_2_borgbase = {
        paths = [ "/etc" "/home" ];
        exclude = (import ./borg_excludes.nix).borg_excludes;
        doInit = false;
        repo =  "j1234siy0@j1234siy0.repo.borgbase.com:repo"; # librem2
        encryption = {
          mode = "repokey-blake2";
          passphrase = "TopSecret";
        };
        environment = { BORG_RSH = "ssh -i /etc/nixos/.keys/id_ed25519_borgbackup"; };
        compression = "auto,lzma";
        startAt = "daily";
        preHook = ''
              # waiting for internet after resume-from-suspend
              ${pkgs.coreutils}/bin/echo Librem_Niklaus_Backup checks for borgbase.com
              until ${pkgs.iputils}/bin/ping borgbase.com -c1 -q >/dev/null; do :; done
              ${pkgs.coreutils}/bin/echo Librem_Niklaus_Backup pinged borgbase.com
            '';
        postHook = ''
              ${pkgs.coreutils}/bin/echo Librem_Niklaus_Backup finished
        '';
    };
  };
}
```

# Troubleshooting

  * Error `Failed to create/acquire the lock`

After some months suddenly a few of my repositories failed to backup.

Looking at the output of `systemd status borgbackup-job-partners_Backup` I found
lines like `Failed to create/acquire the lock /srv/repos/j1234siy0/repo/lock.exclusive (timeout).`
and did not know how to handle it. Finally found in this [post](https://medium.com/macoclock/backing-up-with-borg-on-mac-os-x-fcac50df5188) the solution

```bash
borg --verbose break-lock j1234siy0@j1234siy0.repo.borgbase.com:repo
borg check  j1234siy0@j1234siy0.repo.borgbase.com:repo
```

  * Added wrongly some big files

After some time I detected that my exclude list was not correct and included
some virtual machine images. Therefore I decided to cleanpu my repository using
the prune subcommand of borg via  `borg prune -v --list --dry-run --keep-daily=7 --keep-weekly=4 j1234siy0@j1234siy0.repo.borgbase.com:repo`

