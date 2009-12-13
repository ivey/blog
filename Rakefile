def jekyll(args=nil)
  sh %Q{ jekyll #{args}}
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


task :draft do
  print "Name: "
  foo = STDIN.gets
  puts slugify(foo)
end


def slugify(str)
  str.downcase.gsub(/ +/,'-').gsub(/[^-\w]/,'')
end
