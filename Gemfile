source "https://rubygems.org"

# Hello! This is where you manage which Jekyll version is used to run.
# When you want to use a different version, change it below, save the
# file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
#
#     bundle exec jekyll serve
#
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!
gem "jekyll", "~> 4.3"

gem "jekyll-theme-hydejack", "~> 9.1"

# If you are part of the ["Customers" team](https://github.com/orgs/hydecorp/teams/pro-customers), 
# you can fetch the theme from a private repository. 
# See [Deploy in the Hydejack Docs](https://hydejack.com/docs/deploy) for details.

# gem "jekyll-theme-hydejack", git: "https://github.com/hydecorp/hydejack-pro", tag: "pro/v9.2.0"

# Math rendering is handled client-side via MathJax (configured in _config.yml).
# kramdown-math-katex is not supported on GitHub Pages (requires a local JS runtime).
# gem "kramdown-math-katex"
# gem "duktape"

# Required for `jekyll serve` in Ruby 3
gem "webrick"

# Uncomment when using the `--lsi` option for `jekyll build`
# gem "classifier-reborn"

group :jekyll_plugins do
  gem "jekyll-default-layout"
  gem "jekyll-feed"
  gem "jekyll-optional-front-matter"
  gem "jekyll-paginate"
  gem "jekyll-readme-index"
  gem "jekyll-redirect-from"
  gem "jekyll-relative-links"
  gem "jekyll-seo-tag"
  gem "jekyll-sitemap"
  gem "jekyll-titles-from-headings"
  gem "jekyll-include-cache"

  # Non-Github Pages plugins:
  gem "jekyll-last-modified-at"
  gem "jekyll-compose"
end

gem 'wdm' if Gem.win_platform?
gem "tzinfo-data" if Gem.win_platform?

