#!/usr/bin/env rake
require 'rake/clean'

def stunnel
  Gem.win_platform? ? "\"C:\\Program Files\\stunnel\\stunnel.exe\"" : 'stunnel'
end

@config = {}
Rake.application.add_import 'config.rake'

namespace :server do

  @server_config = {
    :foreground => true,
    :cert_file  => "#{@config[:server_host]}.pem",
    :pidfile    => File.expand_path('stunnel_server.pid')
  }

  desc "write the server private key to #{@server_config[:cert_file]}"
  file @server_config[:cert_file] do |t|
    sh "openssl req -new -x509 -days 365 -nodes -config ssl.cnf -out #{t.name} -keyout #{t.name}"
    File.chmod 0700, t.name
  end
  CLOBBER.include(@server_config[:cert_file])
  CLOBBER.include('ssl.rnd')

  task :cert => @server_config[:cert_file]

  task :config => :cert do
    @server_config[:foreground] = true
  end

  task :conf_daemon => :cert do
    @server_config[:foreground] = false
  end

  desc "generate server.conf from the erb template"
  file 'server.conf' => 'server.conf.erb' do |t|
    require 'erb'
    open(t.name, 'w') do |f|
      f.write ERB.new(File.read("#{t.name}.erb")).result(binding)
    end
  end
  CLEAN.include('server.conf')

  desc "run the server in the foreground"
  task :run => [:config, :stop, 'server.conf'] do
    sh "#{stunnel} #{File.expand_path 'server.conf'}"
  end

  desc "run the server as a deamon (unix)"
  task :daemon => [:conf_daemon, :stop, 'server.conf'] do
    exec "#{stunnel}", "#{File.expand_path 'server.conf'}"
  end

  desc "install the server as a service (windows)"
  task :service => [:config, 'server.conf'] do
    sh "#{stunnel} -install #{File.expand_path 'server.conf'}"
    sh "sc start stunnel"
  end

  desc "stop a running server"
  task :stop do
    system "kill `cat #{@server_config[:pidfile]}`" if File.exist?(@server_config[:pidfile])
  end
  CLEAN.include('*.pid')
end

desc "alias for server:run"
task :server => :'server:run'

namespace :client do
  @client_config = {
    :foreground => true,
    :pidfile    => File.expand_path('stunnel_client.pid')
  }

  task :config do
    @client_config[:foreground] = true
  end

  task :conf_daemon do
    @client_config[:foreground] = false
  end

  desc "generate client.conf from the erb template"
  file 'client.conf' => 'client.conf.erb' do |t|
    require 'erb'
    open(t.name, 'w') do |f|
      f.write ERB.new(File.read("#{t.name}.erb")).result(binding)
    end
  end
  CLEAN.include('client.conf')

  desc "run the client in the foreground"
  task :run => [:config, :stop, 'client.conf'] do
    sh "#{stunnel} #{File.expand_path 'client.conf'}"
  end

  desc "run the client as a daemon (unix)"
  task :daemon => [:conf_daemon, :stop, 'client.conf'] do
    exec "#{stunnel}", "#{File.expand_path 'client.conf'}"
  end

  desc "install the client as a service (windows)"
  task :service => [:config, 'client.conf'] do
    sh "#{stunnel} -install #{File.expand_path 'client.conf'}"
    sh "sc start stunnel"
  end

  desc "stop a running client"
  task :stop do
    system "kill `cat #{@client_config[:pidfile]}`" if File.exist?(@client_config[:pidfile])
  end
  CLEAN.include('*.pid')
end

desc "change server host to be a lan host"
task :lan do
  @config[:server_host] = @config[:lan_host]
end

desc "alias for client:run"
task :client => :'client:run'

desc "alias for client"
task :default => :client