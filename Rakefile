task :jekyll do
  sh %q{jekyll --pygments}
end

task :publish do
  sh %q{rsync -Cavz _site/ gweezlebur.com\:public_html}
  sh %q{rsync -Cavz .htaccess gweezlebur.com\:public_html}
end
