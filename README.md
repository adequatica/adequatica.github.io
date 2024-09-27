# About

A copy of my personal blog on [Medium](https://adequatica.medium.com/) (only technical articles).

## How to Set up Blog on GitHub Pages with Jekyll

1. Create repository by [instruction](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site):

   - Use `<username>.github.io` as the repository name for your personal blog.

2. Install Jekyll:

   - First, [switch to Ruby on Brew](https://github.com/ffi/ffi/issues/653#issuecomment-458895497);
   - Reboot the terminal!
   - Use sudo to install Jekyll: `sudo gem install bundler jekyll`

At this step, I have to add this to `~/.zshrc`:

```
export PATH="/usr/local/opt/ruby/bin:$PATH"
export PATH="/usr/local/sbin:$PATH"
```

3. Create a site by [instruction](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll#creating-your-site):

   - Where `git checkout --orphan gh-pages` is an excess step — you can use the «main» branch;
   - Use sudo to install Bundle: `sudo bundle install`
   - Reboot the terminal!

4. Test a site locally by [instruction](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll#building-your-site-locally):

   - Run: `sudo bundle exec jekyll serve`

5. Add, commit, and push into «main»:

   - For the «gh-pages» branch, the deployment should be set up by [instruction](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-from-a-branch);
   - GitHub Action runs after each push to build GitHub Pages [website](https://adequatica.github.io/).

6. To override theme defaults, follow the [instructions](https://jekyllrb.com/docs/themes/#overriding-theme-defaults):

   - Move theme’s files required for changes to the local repository (I used `_layouts` and `_sass`);
   - Path to gem for adding to VS Code workspace: `../../../usr/local/lib/ruby/gems/3.2.0/gems/minima-2.5.1`

7. To format Markdown files **locally**:

   - Set up `package.json` for [Prettier](https://prettier.io/): `npm install`
   - Add [Husky](https://typicode.github.io/husky/) pre-commit: `npm run prepare`
   - Run formatter when necessary: `npm run format`
