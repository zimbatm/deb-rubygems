# -*- ruby -*-

$:.unshift 'lib'

require 'rubygems'
require 'rubygems/package_task'

require 'hoe'

Hoe.plugin :minitest

hoe = Hoe.spec 'rubygems-update' do
  self.rubyforge_name = 'rubygems'
  self.author         = ['Jim Weirich', 'Chad Fowler', 'Eric Hodel']
  self.email          = %w[rubygems-developers@rubyforge.org]
  self.readme_file    = 'README'
  self.need_zip       = false
  self.need_tar       = false

  spec_extras[:required_ruby_version] = Gem::Requirement.new '> 1.8.3'
  spec_extras[:executables]           = ['update_rubygems']

  clean_globs.push('**/debug.log',
                   '*.out',
                   '.config',
                   'data__',
                   'html',
                   'logs',
                   'pkgs/sources/sources*.gem',
                   'scripts/*.hieraki',
                   'util/gem_prelude.rb')

  extra_dev_deps << 'builder' << 'session' << 'hoe-seattlerb'
  extra_dev_deps << ['minitest', '~> 1.4']

  spec_extras['rdoc_options'] = proc do |rdoc_options|
    rdoc_options << "--title=RubyGems #{self.version} Documentation"
  end
end

desc "Run just the functional tests"
Rake::TestTask.new(:test_functional) do |t|
  t.test_files = FileList['test/functional*.rb']
  t.warning = true
end

# --------------------------------------------------------------------
# Creating a release

task :release => [:clobber, :sanity_check, :test_functional,
                  :test, :package, :tag]

pkg_dir_path = "pkg/rubygems-update-#{hoe.version}"
task pkg_dir_path do
  mv pkg_dir_path, "pkg/rubygems-#{hoe.version}"
end

task :package => [pkg_dir_path, :sanity_check] do
  Dir.chdir 'pkg' do
    sh "tar -czf rubygems-#{hoe.version}.tgz rubygems-#{hoe.version}"
    sh "zip -q -r rubygems-#{hoe.version}.zip rubygems-#{hoe.version}"
  end
end

task :sanity_check do
  abort "svn status dirty. commit or revert them" unless `svn st`.empty?
end

task :tag => [:sanity_check] do
  reltag = "REL_#{PKG_VERSION.gsub(/\./, '_')}"
  svn_url = "svn+ssh://rubyforge.org/var/svn/rubygems"
  sh %{svn copy #{svn_url}/trunk #{svn_url}/tags/#{reltag}}
end

# Misc Tasks ---------------------------------------------------------

desc "build util/gem_prelude.rb from the template and defaults.rb"
file 'util/gem_prelude.rb' =>
     %w[util/gem_prelude.rb.template lib/rubygems/defaults.rb] do
  gem_prelude = File.read 'util/gem_prelude.rb.template'
  defaults = File.read 'lib/rubygems/defaults.rb'

  raise 'template error' unless defaults.sub!(/^module Gem\n/, '')
  raise 'template error' unless defaults.sub!(/^end\n/, '')

  defaults.gsub!(/^/, '  ')

  raise 'template error' unless
    gem_prelude.sub!(/^# WARN\n/,
                     "# THIS FILE WAS AUTOGENERATED, DO NOT EDIT\n")
  raise 'template error' unless
    gem_prelude.sub!(/^    # INCLUDE rubygems\/defaults\n/, defaults)

  rm_f 'util/gem_prelude.rb'

  open 'util/gem_prelude.rb', 'w' do |io|
    io.write gem_prelude
  end
end

# These tasks expect to have the following directory structure:
#
#   git/git.rubini.us/code # Rubinius git HEAD checkout
#   svn/ruby/trunk         # ruby subversion HEAD checkout
#   svn/rubygems/trunk     # RubyGems subversion HEAD checkout
#
# If you don't have this directory structure, set RUBY_PATH and/or
# RUBINIUS_PATH.

def rsync_with dir
  rsync_options = "-avP --exclude '*svn*' --exclude '*swp' --exclude '*rbc'" +
    "--exclude '*.rej' --exclude '*.orig' --exclude 'lib/rubygems/defaults/*'"
  sh "rsync #{rsync_options} bin/gem             #{dir}/bin/gem"
  sh "rsync #{rsync_options} lib/                #{dir}/lib"
  sh "rsync #{rsync_options} test/               #{dir}/test/rubygems"
  sh "rsync #{rsync_options} util/gem_prelude.rb #{dir}/gem_prelude.rb"
end

def diff_with dir
  diff_options = "-urpN --exclude '*svn*' --exclude '*swp' --exclude '*rbc'"
  sh "diff #{diff_options} bin/gem             #{dir}/bin/gem;         true"
  sh "diff #{diff_options} lib/ubygems.rb      #{dir}/lib/ubygems.rb;  true"
  sh "diff #{diff_options} lib/rubygems.rb     #{dir}/lib/rubygems.rb; true"
  sh "diff #{diff_options} lib/rubygems        #{dir}/lib/rubygems;    true"
  sh "diff #{diff_options} lib/rbconfig        #{dir}/lib/rbconfig;    true"
  sh "diff #{diff_options} test                #{dir}/test/rubygems;   true"
  sh "diff #{diff_options} util/gem_prelude.rb #{dir}/gem_prelude.rb;  true"
end

rubinius_dir = ENV['RUBINIUS_PATH'] || '../../../git/git.rubini.us/code'
ruby_dir     = ENV['RUBY_PATH']     || '../../ruby/trunk'

desc "Updates Ruby HEAD with the currently checked-out copy of RubyGems."
task :update_ruby     => 'util/gem_prelude.rb' do
  rsync_with ruby_dir
end

desc "Updates Rubinius HEAD with the currently checked-out copy of RubyGems."
task :update_rubinius => 'util/gem_prelude.rb' do
  rsync_with rubinius_dir
end

desc "Diffs Ruby HEAD with the currently checked-out copy of RubyGems."
task :diff_ruby       => 'util/gem_prelude.rb' do
  diff_with ruby_dir
end

desc "Diffs Rubinius HEAD with the currently checked-out copy of RubyGems."
task :diff_rubinius   => 'util/gem_prelude.rb' do
  diff_with rubinius_dir
end