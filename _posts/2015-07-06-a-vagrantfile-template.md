---
layout: post
title: A Vagrantfile template
categories: [ linux ]
comments: true
---
This is a Vagrantfile template that I use for most projects.  It's easy to add new nodes or mount points as the cluster grows and each node can be customized.  Each server can be easily provisioned with Puppet, Chef, SaltStack, or a script.  The [vagrant-cachier](https://github.com/fgrehm/vagrant-cachier) plugin is used to speed up provisioning time.

<!--more-->

{% highlight ruby %}
domain = 'dev.example.com'
nodes = {
  :'web-0' => {
    ip:   '10.10.10.2',
    mem:  512,
    cpus: 1,
    box:  'chef/centos-6.6'
  },
  :'db-0' => {
    ip:   '10.10.10.3',
    mem:  512,
    cpus: 1,
    box:  'jkyle/centos-7.0-x86_64'
  }
}
mounts = {
  'localdir'  => '/root/remotedir',
  'localdir2' => '/root/remotedir2'
}
Vagrant.configure("2") do |config|
  nodes.each do |hostname, settings|
    fqdn = "#{hostname}.#{domain}"

    config.vm.define hostname do |node|
      if Vagrant.has_plugin?("vagrant-cachier")
        config.cache.scope = :box
        config.cache.synced_folder_opts = {
          type: :nfs,
          mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
        }
      end

      node.vm.box = settings[:box]
      node.vm.hostname = fqdn
      node.vm.network :private_network, ip: settings[:ip]
      node.vm.provider "vmware_fusion" do |v|
        v.vmx["memsize"]  = settings[:mem]
        v.vmx["numvcpus"] = settings[:cpus]
      end

      mounts.each do |folder, mount|
        node.vm.synced_folder folder, mount
      end
    end
  end
end
{% endhighlight %}

Please note the above VM settings are built for VMWare Fusion.  The following settings can be used for the default provider, Virtualbox:

{% highlight ruby %}
node.vm.provider "virtualbox" do |v|
  v.memory  = settings[:mem]
  v.cpus    = settings[:cpus]
end
{% endhighlight %}

