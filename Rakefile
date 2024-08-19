require 'time'

AVAILABLE_REVISIONS = %w[major minor patch].freeze

# task :default => [:deploy]

task :command_exists, [:command] do |_, args|
  abort "#{args.command} doesn't exists" if `command -v #{args.command} > /dev/null 2>&1 && echo $?`.chomp.empty?
end

task :has_bumpversion do
  Rake::Task['command_exists'].invoke('bumpversion')
end

task :bump, [:revision] => [:has_bumpversion] do |_, args|
  args.with_defaults(revision: 'patch')
  unless AVAILABLE_REVISIONS.include?(args.revision)
    abort "Please provide valid revision: #{AVAILABLE_REVISIONS.join(',')}"
  end

  system "bumpversion #{args.revision}"
end

desc "deploy"
task :deploy, [:revision] do |_, args|
  args.with_defaults(version: 'patch')

  now = Time.now
  current_update_time = now.strftime("%B %d, %H:%M, %Y")
  system %{
    sed -i "" 's/\\(<cite>Last update : \\)[^<]*\\(<\\/cite>\\)/\\1#{current_update_time}\\2/' index.html
  }

  unless `git status -s | wc -l`.strip.to_i.zero?
    system %{
      git add .
      git commit -m '[UPDATE] - #{current_update_time}'
      git push
    }
    abort "auto add and commit failed" if $?.exitstatus != 0
  end

  if $?.exitstatus == 0
    Rake::Task[:bump].invoke(args.revision)

    system %{
      git push
      git push --tags
      git checkout gh-pages &&
      git pull &&
      git rebase main &&
      git push origin gh-pages &&
      git checkout main
    }
    exit $?.exitstatus
  else
    abort "sed fucked up"
  end
end
