def jekyll(args=nil)
  sh %Q{ jekyll --pygments #{args}}
end

task :build do
  jekyll
end

task :server do
  jekyll "--server 3100 --auto"
end

task :push do
  sh %q{git push origin}
end

task :rsync do
  sh %q{rsync -Cavz _site/ gweezlebur.com\:public_html}
  sh %q{rsync -Cavz .htaccess gweezlebur.com\:public_html}
end

task :publish => [:push, :build, :rsync]
