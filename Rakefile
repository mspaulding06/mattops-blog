require "rubygems"
require "bundler/setup"
require "jekyll"

namespace :site do
  desc "Generate blog files"
  task :generate do
    Jekyll::Site.new(Jekyll.configuration({
      "source"      => ".",
      "destination" => "_site"
    })).process
  end

  desc "Generate and publish blog to s3 bucket"
  task :publish => [:generate] do
    system "aws s3 sync _site/. s3://mattops.io/ --delete --acl public-read"
  end
end
