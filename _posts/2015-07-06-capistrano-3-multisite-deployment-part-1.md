---
layout: post
title: Capistrano 3 multisite deployment (Part 1)
---
## Introduction
On a recent project, I had a need to deploy over 100 copies of software from a git repository, along with a second repository for add-ons and styles specific to that site.  Since Capistrano 3 wasn't built for this type of deployment, I had to get creative.  A CI/CD tool is used for testing and running capistrano.  The process needed to be automated, accessible to all team members, yet functional enough to handle a variety of situations.  This will be a three part series highlighting some of the techniques used on this project.  In the near future, I'll likely be doing a rewrite and offloading most functionality to ruby.

## Setting the stage
The first step was to build site specific parameters in YAML files, which could be easily updated by anyone on the team.  All passwords are encrypted for security and decrypted the configuration phase.  The YAML file consisted of a nested hash for each site:
{% highlight yaml %}
example.com:
  name: example
  pass:
  host: 127.0.0.1
  maintenance: false
{% endhighlight %}

{% highlight ruby %}
set :devel,         YAML.load_file("lib/deploy/devel.yaml")
set :staging,       YAML.load_file("lib/deploy/staging.yaml")
set :production,    YAML.load_file("lib/deploy/production.yaml")
set :sites,         fetch(:"#{fetch(:stage)}")
{% endhighlight %}

### Building the stages
Without writing some ruby, the only way I've found to effectively build individual parameters for each site is to create config files for each and loop through them locally.  It's possible that individual site parameters could be stored and accessed in an environment variable.  But overwriting global values in capistrano for each site while performing each task needed can run into problems with scope that are more easily solved with separate stages.

First we need to write a template class to build the stages from an ERB template.
{% highlight ruby %}
namespace :deploy do
  def template(from, to, as_root = false)
    template_path = File.expand_path("../templates/#{from}", __FILE__)
    template = ERB.new(File.new(template_path).read).result(binding)
    upload! StringIO.new(template), to

    execute :chmod, "644 #{to}"
    execute :chown, "root:root #{to}" if as_root == true
  end
end
{% endhighlight %}

The template file will include all of the variables we'll be accessing in the next steps.
{% highlight erb %}
set :application, "<%= fetch(:sname) %>"
set :deploy_to,   "/srv/<%= fetch(:sname) %>"
set :dbname,      "<%= fetch(:sname) %>"[0...16]
set :pass,        "<%= fetch(:pass) %>"
set :host,        "<%= fetch(:host) %>"
set :maintenance, <%= fetch(:maintenance) %>
set :version,     "<%= fetch(:version) %>"

server '127.0.0.1', user: 'deploy', roles: %w{web backup}
{% endhighlight %}

Next we use this template to build the staging configuration files, stored in config/deploy/.
{% highlight ruby %}
namespace :deploy do
  desc 'Build staging config files from template for each site'
  task :build_config do
    fetch(:sites).each do |site, attr|
      run_locally do
        set :pass, attr['pass']
        set :sname, attr['name']
        set :version, attr['version']
        set :maintenance, attr['maintenance']
        set :host, attr['host']
        template "#{fetch(:stage)}.rb.erb", "config/deploy/#{site}_#{fetch(:stage)}.rb"
      end
    end
  end
end
{% endhighlight %}

### Deploying the sites

Now all of these stages can be accessed individually, which makes testing much easier.  We just need a task that will deploy them all in a single command, which has to be run locally on either your workstation or the CI/CD tool of your choice.

{% highlight ruby %}
namespace :deploy do
  desc 'Loop through each environment locally and run the deployment'
  task :all_sites do
    fetch(:sites).each do |site, attr|
      run_locally do
        execute "bundle exec cap #{site}_#{fetch(:stage)} deploy"
      end
    end
  end
{% endhighlight %}

All that's left is to run
{% highlight bash %}
bundle exec cap staging deploy:all_sites
{% endhighlight %}

In the next post, we'll start writing tasks to use this information for our deployments.