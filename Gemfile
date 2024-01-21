source "https://rubygems.org"
gem "jekyll", "~> 3.4"
# This is the default theme for new Jekyll sites. You may change this to anything you like.
# gem "minima"
gem "jekyll-theme-hydeout", "~> 3.4"
# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
# gem "github-pages", group: :jekyll_plugins
# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
end

gem 'jekyll-titles-from-headings'
gem 'jekyll-multiple-languages-plugin'
gem 'kramdown-parser-gfm'

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :install_if => Gem.win_platform?

