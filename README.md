# chrisjquinn.github.io

Personal website, blog.

This website runs thanks to:
- Jekyll: A blog-aware, static website runner [jekyllrb.com](http://jekyllrb.com)
- Hyde: A two-column theme. [Hyde](https://github.com/poole/hyde)

Few additions were made on top of this, such as a dark-mode response and small CSS tweaks.


## Debugging
### Jekyll / Bundler Issues
If bundle exec jekyll serve fails, it’s almost always a Ruby version mismatch.
**Symptoms**
You may see errors like:
- Could not find `concurrent-ruby-x.x.x`
- `nokogiri` requires `ruby < 3.3`
- Bundler fails even after running `bundle install`

**Root Cause**
Jekyll (and GitHub Pages) do not yet support Ruby 3.3+.
macOS may default to a newer system Ruby, even if rbenv is installed.

Quick Fix
1. Ensure the correct Ruby version is active This project expects **Ruby 3.2.x.**
```bash
rbenv local 3.2.2
ruby -v
```
Expected output:
```bash
ruby 3.2.2p...
```

If it still shows 3.3.x, rbenv is not initialised in your shell.
2. Verify rbenv is active with `which ruby`. Expected: `~/.rbenv/shims/ruby` 

If not, ensure your shell config `(~/.zshrc)` includes:
```bash
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init - zsh)"
```
Then reload:
```bash
source ~/.zshrc
```
3. Reinstall dependencies cleanly
Once the correct Ruby version is active:
```bash
rm -f Gemfile.lock
bundle install
bundle exec jekyll serve
```

**Notes**
Warnings about `DidYouMean` deprecations are harmless
Conda environments (`(base)` in prompt) **can override** Ruby — if issues persist, consider disabling auto-activation:
`conda config --set auto_activate_base false`

**Sanity Check**
```bash
ruby -v        # should be 3.2.x
bundle -v
bundle exec jekyll serve
```