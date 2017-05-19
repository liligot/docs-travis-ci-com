#!/usr/bin/env rake

require 'html-proofer'

task :default => [:test]

desc 'Runs the tests!'
task :test => :build do
  Rake::Task['run_html_proofer'].invoke
end

desc 'Builds the site'
task :build => [:remove_output_dir, :gen_user_erb] do
  FileUtils.rm '.jekyll-metadata' if File.exist?('.jekyll-metadata')
  sh 'bundle exec jekyll build --config=_config.yml'
end

desc 'Remove the output dir'
task :remove_output_dir do
  FileUtils.rm_r('_site') if File.exist?('_site')
end

def print_line_containing(file, str)
  File.open(file).grep(/#{str}/).each do |line| puts "#{file}: #{line}" end
end

desc 'Lists files containing beta features'
task :list_beta_files do
  files = FileList.new('**/*.md')
  files.exclude("_site/*", "STYLE.md")
  for f in files do
    print_line_containing(f, '\.beta')
  end
end

desc 'Runs the html-proofer test'
task :run_html_proofer do
  # seems like the build does not render `%3*`,
  # so let's remove them for the check
  url_swap = {
    /%3A\z/ => '',
    /%3F\z/ => '',
    /-\.travis\.yml/ => '-travisyml'
  }

  tester = HTMLProofer.check_directory('./_site', {
                              :url_swap => url_swap,
                              :connecttimeout => 600,
                              :only_4xx => true,
                              :typhoeus => { :ssl_verifypeer => false, :ssl_verifyhost => 0, :followlocation => true },
                              :url_ignore => ["https://www.appfog.com/", /itunes\.apple\.com/, /coverity.com/, /articles201769485/],
                              :file_ignore => ["./_site/api/index.html", "./_site/user/languages/erlang/index.html"]
                            })
  tester.run
end

PREREQ_REMOTE_FILES = [
  'https://raw.githubusercontent.com/travis-infrastructure/terraform-config/master/aws-production-2/generated-language-mapping.json'
]

desc 'Generates ERb-templated /user documents'
task :gen_user_erb do
  require 'find'

  PREREQ_REMOTE_FILES.each { |f| `curl -OsSfL #{f}` }

  files = Find.find(File.join(File.dirname(__FILE__), 'user')) do |f|
    if File.file?(f) && File.fnmatch('*.erb', f)
      `erb -r json user/common-build-problems.md.erb | tee user/common-build-problems.md`
    end
  end
end

namespace :assets do
  task :precompile do
    puts `bundle exec jekyll build`
  end
end
