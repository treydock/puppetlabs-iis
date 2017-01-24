require 'puppetlabs_spec_helper/rake_tasks'
require 'puppet-lint/tasks/puppet-lint'
require 'puppet_blacksmith/rake_tasks' if Bundler.rubygems.find_name('puppet-blacksmith').any?

PuppetLint.configuration.fail_on_warnings = true
PuppetLint.configuration.send('relative')
PuppetLint.configuration.send('disable_80chars')
PuppetLint.configuration.send('disable_class_inherits_from_params_class')
PuppetLint.configuration.send('disable_documentation')
PuppetLint.configuration.send('disable_single_quote_string_with_variables')
PuppetLint.configuration.ignore_paths = ["spec/**/*.pp", "pkg/**/*.pp", "vendor/**/*.pp"]

begin
  require('rototiller')
  default_acceptance_platform = 'windows2012r2-64'

  desc 'Generate Beaker Host config'
  rototiller_task :host_config, [:default_platform] do |t, args|
    t.add_env({:name => 'PLATFORM', :message => 'PLATFORM Must be set. For example "windows2012r2-64"', :default => args[:default_platform]})
    hosts_file = "spec/acceptance/nodesets/#{ENV['PLATFORM']}.yml"
    t.add_command do |cmd|
      cmd.name = 'bundle exec beaker-hostgenerator'
      cmd.add_argument({:name => "#{ENV['PLATFORM']} > #{hosts_file}"})
    end
  end

  desc 'Executes acceptance tests'
  rototiller_task :acceptance_tests do |t|
    Rake::Task[:host_config].invoke("#{default_acceptance_platform}")
    t.add_command(name: 'bundle exec rspec spec/acceptance')

    t.add_env do |env|
      env.name = 'PUPPET_INSTALL_TYPE'
      env.default = 'agent'
    end

    t.add_env do|env|
      env.name = 'BEAKER_setfile'
      env.default = 'spec/acceptance/nodesets/' + ENV['PLATFORM'] + '.yml'
    end

    t.add_env do |env|
      env.name = 'BEAKER_keyfile'
      env.default = "#{ENV['HOME']}/.ssh/id_rsa-acceptance"
    end
  end
rescue LoadError
  #Do nothing, rototiller only installed with system_tests group
end

desc 'Generate pooler nodesets'
task :gen_nodeset do
  require 'beaker-hostgenerator'
  require 'securerandom'
  require 'fileutils'

  agent_target = ENV['TEST_TARGET']
  if ! agent_target
    STDERR.puts 'TEST_TARGET environment variable is not set'
    STDERR.puts 'setting to default value of "redhat-64default."'
    agent_target = 'redhat-64default.'
  end

  master_target = ENV['MASTER_TEST_TARGET']
  if ! master_target
    STDERR.puts 'MASTER_TEST_TARGET environment variable is not set'
    STDERR.puts 'setting to default value of "redhat7-64mdcl"'
    master_target = 'redhat7-64mdcl'
  end

  targets = "#{master_target}-#{agent_target}"
  cli = BeakerHostGenerator::CLI.new([targets])
  nodeset_dir = "tmp/nodesets"
  nodeset = "#{nodeset_dir}/#{targets}-#{SecureRandom.uuid}.yaml"
  FileUtils.mkdir_p(nodeset_dir)
  File.open(nodeset, 'w') do |fh|
    fh.print(cli.execute)
  end
  puts nodeset
end
