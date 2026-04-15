# About

A copy of technical articles from [my personal blog on Medium](https://adequatica.medium.com/).

## How to Set up Blog on GitHub Pages with Jekyll

1. Create repository by [instruction](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site):
   - Use `<username>.github.io` as the repository name for your personal blog.

2. Install Jekyll:
   - First, [switch to Ruby on Brew](https://github.com/ffi/ffi/issues/653#issuecomment-458895497):

   ```bash
   brew install ruby

   brew link --force --overwrite ruby
   ```

   At this step, I have to add this to `~/.zshrc`:

   ```bash
   echo 'export PATH="/usr/local/opt/ruby/bin:$PATH"' >> ~/.zshrc

   echo 'export PATH="/usr/local/opt/ruby/bin:$PATH"' >> ~/.zshrc
   ```

   - **Reboot the terminal!**

   - Use sudo to install Jekyll:

   ```bash
   sudo gem install bundler jekyll
   ```

3. Create a site by [instruction](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll#creating-your-site):
   - Where `git checkout --orphan gh-pages` is an excess step — you can use the «main» branch;

   - Use sudo to install Bundle:

     ```bash
     sudo bundle install
     ```

   - **Reboot the terminal!**

4. Test a site locally by [instruction](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll#building-your-site-locally):
   - Run: `sudo bundle exec jekyll serve`

5. Add, commit, and push into «main»:
   - For the «gh-pages» branch, the deployment should be set up by [instruction](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-from-a-branch);

   - GitHub Action runs after each push to build GitHub Pages [website](https://adequatica.github.io/).

   - Path to gem for adding to VS Code workspace: `../../../usr/local/lib/ruby/gems/3.2.10/gems/minima-2.5.2`

---

### Testing a site locally after a while…

If `sudo bundle exec jekyll serve` does not work and you get an error:

```bash
/usr/local/opt/ruby/bin/bundle:25:in `load': cannot load such file -- /usr/local/lib/ruby/gems/3.3.0/gems/bundler-2.4.22/exe/bundle (LoadError)
	from /usr/local/opt/ruby/bin/bundle:25:in `<main>'
```

Check the bundle version: `bundle -v`

If it is not 2.4.22, then you need to install it: `sudo gem install bundler -v 2.4.22`.

Reinstall gems: `sudo bundle install` and try to exec jekyll server again.

If you still get an error like this:

```bash
jekyll 3.9.3 | Error:  undefined method `[]' for nil
/usr/local/Cellar/ruby/3.3.6/lib/ruby/3.3.0/logger.rb:384:in `level': undefined method `[]' for nil (NoMethodError)

    @level_override[Fiber.current] || @level
                   ^^^^^^^^^^^^^^^
	from /usr/local/lib/ruby/gems/3.3.0/gems/jekyll-3.9.3/lib/jekyll/log_adapter.rb:43:in `adjust_verbosity'
   …
   etc.
```

Check the Ruby version: `ruby -v` — Jekyll 3.9.3 was tested under Ruby 3.2.10.

If Ruby's version is higher than 3.2.10 (probably 3.3.x), you need to install [Ruby version manager](https://rbenv.org/) and install and switch to a compatible version:

```bash
brew install rbenv ruby-build

echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc

source ~/.zshrc
```

```bash
rbenv install 3.2.10

rbenv global 3.2.10
```

- **Reboot the terminal!**

Recheck the Ruby version: `ruby -v` — it should be 3.2.10.

Reinstall Bundler and gems:

```bash
gem install bundler

sudo bundle install
```

Now it should run:

```bash
sudo bundle exec jekyll serve
```

A [similar article](https://ritviknag.com/tech-tips/ruby-versioning-hell-with-jekyll-&-github-pages/) about the painful process of running GitHub Pages' Jekyll.

---

Only for this repository — local Markdown files formatting:

- Set up `package.json` for [Prettier](https://prettier.io/): `npm install`
- Add [Husky](https://typicode.github.io/husky/) pre-commit: `npm run prepare`
- Run formatter when necessary: `npm run format`
