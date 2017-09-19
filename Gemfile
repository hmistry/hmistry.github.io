# frozen_string_literal: true

source "https://rubygems.org"
ruby RUBY_VERSION
gem 'json', '>= 2.0'
require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

# gem "jekyll", "~> 3.5.1"

# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
gem 'github-pages', versions['github-pages'], group: :jekyll_plugins

gem "minima", "~> 2.0"

group :jekyll_plugins do
   gem "jekyll-feed", "~> 0.6"
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
