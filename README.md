# chrisjquinn.github.io

Personal website, blog.

This website runs thanks to:
- Jekyll: A blog-aware, static website runner [jekyllrb.com](http://jekyllrb.com)
- Hyde: A two-column theme. [Hyde](https://github.com/poole/hyde)

Few additions were made on top of this, such as a dark-mode response and small CSS tweaks.


## Debugging
Running this thing locally quite simply is a pain in the ass. This section is for the chris of the future to debug his own work as to why ruby is not working on his laptop.

1. Check ruby & gem PATH (and the `.zshrc` PATH homebrew recommends)
```bash
% which ruby
/usr/local/opt/ruby/bin/ruby
% which gem
/usr/local/opt/ruby/bin/gem
```
2. Now with bundle
```bash
% which bundle
/usr/local/opt/ruby/bin/bundle
```
3. Check your bundle version: `bundle -v`
4. Uninstall and re-install bundler with `gem uninstall bundler; gem install bundler`
5. Update bundler `gem update bundler`
6. Install the gems for this project `bundle install`.
7. Cry.