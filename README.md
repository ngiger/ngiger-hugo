

# Bootstrapping under nixos

```fish
mkdir ngiger-jekyll
cd ngiger-jekyllo
nix-shell -p zlib bundler git ruby --command fish
jekyll new blog
Running bundle install in /opt/src/ngiger-jekyll/blog4...
Bundler: Skipping "bundle install" as it fails due to the Nix wrapper.
Bundler: Please enter the new directory and run the following commands to serve the page:
Bundler: nix-shell -p bundler --run "bundle install --gemfile=Gemfile --path vendor/cache"
Bundler: nix-shell -p bundler --run "bundle exec jekyll serve"
cd blog
bundler install
bundle exec jekyll server --port 4050
 ```
## Bootstrappin on nuc

nix-env -iA nixos.ruby.devEnv nixos.gcc nixos.libressl nixos.gnumake nixos.nixUnstable nixos.rubyPackages.ffi
 1.15.
 
 nix-shell -I nixpkgs=https://github.com/NixOS/nixpkgs/archive/d373d80b1207d52621961b16aa4a3438e4f98167.tar.gz
 nix-shell -I nixpkgs=https://github.com/NixOS/nixpkgs/archive/nixos-21.11.tar.gz
 
 Dann läuft in der nix-shell bundle install und bundle exec jekyll serve

# TODO: 

* https://kevq.uk/how-to-add-search-jekyll
* https://blog.webjeda.com/jekyll-search/
