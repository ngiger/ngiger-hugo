---
title:  "Using rbenv under NixOS"
date:   2022-02-05 11:17:24 +0100
draft: false
tags:
  - nixops
  - ruby
---

# Adding rbenv

```bash
nix-env -iA nixos.rbenv 
echo "status --is-interactive; and source (rbenv init -|psub)" | tee --append ~/.config/fish/config.fish
mkdir -p ~/.rbenv/plugins
cd  ~/.rbenv/plugins
git clone https://github.com/rbenv/ruby-build
nix-shell -p openssl readline bundler

rbenv install 3.1.0
# autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm6 libgdbm-dev libdb-dev
```
