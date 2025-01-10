# frozen_string_literal: true

source "https://rubygems.org"
ruby RUBY_VERSION
# gem 'json', '>= 2.0'
# require 'json'
# require 'open-uri'
# versions = JSON.parse(::URI.open('https://pages.github.com/versions.json').read)

gem "jekyll", "~> 4.3"

# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
# gem 'github-pages', versions['github-pages'], group: :jekyll_plugins

gem "minima"

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.17"
  gem "jekyll-compose"
  gem "jekyll-timeago", "~> 0.15"
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
