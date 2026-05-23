# frozen_string_literal: true

source "https://rubygems.org"

gemspec

gem "webrick" # Required for Jekyll serve with Ruby 3.x
# Pin jekyll-sass-converter for Alpine/musl (Docker) to avoid sass-embedded segfault.
# On Ubuntu (GitHub Actions), the default 3.x with sass-embedded works fine.
gem "jekyll-sass-converter", "~> 2.0", platform: [:x86_64-linux-musl]
gem "html-proofer", "~> 5.0", group: :test

platforms :windows, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", :platforms => [:windows]
