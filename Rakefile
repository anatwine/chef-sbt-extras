#!/usr/bin/env rake

# needed or not? - require 'json'
require 'tailor/rake_task'
require 'rspec/core/rake_task'

task :default => 'test'
task :test => [:tailor, :foodcritic, :knife, :spec]

desc "Runs foodcritic linter"
task :foodcritic do
  Rake::Task[:prepare_sandbox].execute

  if Gem::Version.new("1.9.2") <= Gem::Version.new(RUBY_VERSION.dup)
    sh "foodcritic -f any #{sandbox_path}"
  else
    puts "WARN: foodcritic run is skipped as Ruby #{RUBY_VERSION} is < 1.9.2."
  end
end
# Variant 2:
#task :foodcritic_bis do
#  FoodCritic::Rake::LintTask.new do |t|
#    t.options = { :fail_tags => ['any'] }
#  end
#end

desc "Runs tailor linter"
task :tailor do
  Tailor::RakeTask.new do |task|
    task.file_set('attributes/**/*.rb', "attributes") do |style|
      style.max_line_length 160, level: :warn
    end
    #task.file_set('definitions/**/*.rb', "definitions")
    #task.file_set('libraries/**/*.rb', "libraries")
    task.file_set('metadata.rb', "metadata") do |style|
      style.max_line_length 80, level: :warn
    end
    #task.file_set('providers/**/*.rb', "providers")
    task.file_set('recipes/**/*.rb', "recipes") do |style|
      style.max_line_length 160, level: :warn
    end
    #task.file_set('resources/**/*.rb', "resources")

    # Template analysis is currently disabled, because I have no clue about how 'ruby -c' could support ERB markers like '<%'
    # task.file_set('templates/**/*.erb', "templates")

    task.file_set('spec/**/*.rb', "chefspec") do |style|
      style.max_line_length 160, level: :warn
    end
  end
end

desc "Runs knife cookbook test"
task :knife do
  Rake::Task[:prepare_sandbox].execute

  ENV["BUNDLE_GEMFILE"] = "test/support/Gemfile"
  sh "bundle exec knife cookbook test #{cookbook_name} -c #{sandbox_root}/knife.rb"
end

task :prepare_sandbox do
  files = %w{*.md *.rb attributes definitions files providers recipes resources templates}

  rm_rf sandbox_root
  mkdir_p sandbox_path
  mkdir_p File.join(sandbox_root, "cache")

  cp_r Dir.glob("{#{files.join(',')}}"), sandbox_path

  File.open(knife_rb, "w") do |fp|
    fp.write("cookbook_path ['#{sandbox_root}/cookbooks/']\n")
    fp.write("cache_type    'BasicFile'\n")
    fp.write("cache_options :path => '#{sandbox_root}/cache'\n")
  end

  # Add fake dependant cookbooks (put only the stuff chefspec will verify)
  cookbook_deps = %w(java)
  cookbook_deps.each do |dep|
    mkdir_p File.join(sandbox_cookbooks, dep, 'recipes')
    touch File.join(sandbox_cookbooks, dep, %w(recipes default.rb))
    touch File.join(sandbox_cookbooks, dep, 'README.md')
  end
end

RSpec::Core::RakeTask.new

private

def cookbook_name
  'sbt-extras'
end

def sandbox_root
  File.join(File.dirname(__FILE__), %w(tmp))
end

def sandbox_cookbooks
  File.join(sandbox_root, 'cookbooks')
end

def sandbox_path
  File.join(sandbox_cookbooks, cookbook_name)
end

def knife_rb
  File.join(sandbox_root, 'knife.rb')
end
