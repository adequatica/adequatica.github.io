# About

A copy of my personal blog on [Medium](https://adequatica.medium.com/) (only technical articles).

## How to Set up Blog on GitHub Pages with Jekyll

1. Create repository by [instruction](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site):

   - Use `<username>.github.io` as the repository name for your personal blog.

2. Install Jekyll:

   - First, [switch to Ruby on Brew](https://github.com/ffi/ffi/issues/653#issuecomment-458895497);

     At this step, I have to add this to `~/.zshrc`:

     ```
     export PATH="/usr/local/opt/ruby/bin:$PATH"

     export PATH="/usr/local/sbin:$PATH"
     ```

   - **Reboot the terminal!**
   - Use sudo to install Jekyll:

     ```
     sudo gem install bundler jekyll
     ```

3. Create a site by [instruction](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll#creating-your-site):

   - Where `git checkout --orphan gh-pages` is an excess step — you can use the «main» branch;
   - Use sudo to install Bundle:

     ```
     sudo bundle install
     ```

   - **Reboot the terminal!**

4. Test a site locally by [instruction](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll#building-your-site-locally):

   - Run: `sudo bundle exec jekyll serve`

5. Add, commit, and push into «main»:

   - For the «gh-pages» branch, the deployment should be set up by [instruction](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-from-a-branch);
   - GitHub Action runs after each push to build GitHub Pages [website](https://adequatica.github.io/).

6. To override theme defaults, follow the [instructions](https://jekyllrb.com/docs/themes/#overriding-theme-defaults):

   - Move theme’s files required for changes to the local repository (I used `_layouts` and `_sass`);
   - Path to gem for adding to VS Code workspace: `../../../usr/local/lib/ruby/gems/3.2.0/gems/minima-2.5.1`

---

### Testing a site locally after a while…

If `sudo bundle exec jekyll serve` does not work and you get an error:

```
/usr/local/opt/ruby/bin/bundle:25:in `load': cannot load such file -- /usr/local/lib/ruby/gems/3.3.0/gems/bundler-2.4.22/exe/bundle (LoadError)
	from /usr/local/opt/ruby/bin/bundle:25:in `<main>'
```

Check the bundle version: `bundle -v`

If it is not 2.4.22, then you need to install it: `sudo gem install bundler -v 2.4.22`.

Reinstall gems: `sudo bundle install` and try to exec jekyll server again.

If you still get an error like this:

```
jekyll 3.9.3 | Error:  undefined method `[]' for nil
/usr/local/Cellar/ruby/3.3.6/lib/ruby/3.3.0/logger.rb:384:in `level': undefined method `[]' for nil (NoMethodError)

    @level_override[Fiber.current] || @level
                   ^^^^^^^^^^^^^^^
	from /usr/local/lib/ruby/gems/3.3.0/gems/jekyll-3.9.3/lib/jekyll/log_adapter.rb:43:in `adjust_verbosity'
   …
   etc.
```

Check the Ruby version: `ruby -v` — Jekyll 3.9.3 was tested under Ruby 3.2.

If Ruby's version is higher than 3.2 (probably 3.3.x), you need to install [Ruby version manager](https://rbenv.org/) and install and switch to a compatible version:

```
brew install rbenv ruby-build

echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc

source ~/.zshrc
```

```
rbenv install 3.1.4

rbenv global 3.1.4
```

- **Reboot the terminal!**

Recheck the Ruby version: `ruby -v` — it should be 3.1.4.

Reinstall Bundler and gems:

```
gem install bundler

sudo bundle install
```

Now it should run: `sudo bundle exec jekyll serve`

A [similar article](https://ritviknag.com/tech-tips/ruby-versioning-hell-with-jekyll-&-github-pages/) about the painful process of running GitHub Pages' Jekyll.

---

Only for this repository: local Markdown files formatting:

- Set up `package.json` for [Prettier](https://prettier.io/): `npm install`
- Add [Husky](https://typicode.github.io/husky/) pre-commit: `npm run prepare`
- Run formatter when necessary: `npm run format`
