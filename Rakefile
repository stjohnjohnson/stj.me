task :local do
    system 'bundle exec jekyll serve -w'
end

namespace :assets do
    task :precompile do
        puts `bundle exec jekyll build`
    end
end
