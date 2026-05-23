# frozen_string_literal: true

source "https://rubygems.org"

gemspec

gem "webrick" # Required for Jekyll serve with Ruby 3.x
# sass-embedded segfaults on Alpine/musl (Docker), but sassc (v2.x) fails to
# compile on Ruby 3.4+ (CI). Use v2.x locally, v3.x on CI (Ubuntu).
gem "jekyll-sass-converter", "~> 2.0" unless ENV["CI"]
gem "html-proofer", "~> 5.0", group: :test

platforms :windows, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", :platforms => [:windows]
