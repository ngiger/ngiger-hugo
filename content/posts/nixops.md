---
title:  "Notes about nixops"
date:   2022-02-01 11:17:24 +0100
draft: false
tags:
  - nixops
  - nix
---

# Bootstrapping nixops version 2.0

```bash
nixops create -d nuc
cd /etc/nixos/hosts/nuc
nixops deploy -d nuc --test
# test the system and if everythins is okay install and reboot
nixops deploy -d nuc --boot
nixops deploy -d nuc --force-reboot
```

Hatte die Fehlermeldung `nuc> warning: unable to download 'https://cache.nixos.org/nix-cache-info': Couldn't resolve host name (6); retrying in 276 mss`
Gelöst mit `rm $HOME/.cache/nix/binary-cache-v*.sqlite*` auf `nuc`
https://oddco.de/post/private-server/

## Ideas for storing secrets

See https://christine.website/blog/nixos-encrypted-secrets-2021-01-20
    pkgs.age # https://github.com/FiloSottile/age 
    age is a simple, modern and secure file encryption tool, format, and Go library.
