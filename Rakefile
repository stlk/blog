require "rubygems"
require "tmpdir"

require "jekyll"

namespace :site do
  desc "Generate blog files"
  task :generate do
    Jekyll::Site.new(Jekyll.configuration({
      "source"      => ".",
      "destination" => "_site"
    })).process
  end


  desc "Generate and publish blog to gh-pages"
  task :publish => [:generate] do
    Dir.chdir "_site/"
    system "git config --global user.email \"josef.rousek@gmail.com\""
    system "git config --global user.name \"Josef Rousek\""
    system "git init"
    system "git add ."
    message = "Site updated at #{Time.now.utc}"
    system "git commit -m #{message.inspect}"
    # system "git remote add origin git@github.com:stlk/blog.git"
    # system "git push origin master:refs/heads/gh-pages --force"
  end
end
