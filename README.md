

# Bootstrapping Hugo under nixos

See [Getting started](https://gohugo.io/getting-started/quick-start/)

```fish
mkdir ngiger-hugo
cd ngiger-hugoo
nix-shell -p hugo --command fish
hugo new site .
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
echo theme = \"ananke\" >> config.toml
hugo new posts/my-first-post.md
echo "Something to tell you" >> new posts/my-first-post.md
hugo server -D # to show drafts, too
wget http://localhost:1313/index.html
 ```
# Bootstrapping git checkout on nuc

```bash
nix-env -iA nixos.git nixos.hugo
sudo mkdir /var/blog
sudo chown -R nginx /var/blog
sudo -u nginx git clone https://github.com/ngiger/ngiger-hugo.git  /var/blog
cd /var/blog
sudo -u nginx git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
sudo -u nginx git clone https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
```

# Publishing the blog via nuc

Using the followin Nix snippet in the file `features/blog_hugo.nix`


```nix

```
