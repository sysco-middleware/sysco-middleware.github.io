if !Vagrant.has_plugin?("vagrant-triggers")
  puts "'vagrant-triggers' plugin is required"
  puts "This can be installed by running:"
  puts
  puts " vagrant plugin install vagrant-triggers"
  puts
  exit
end

if !Vagrant.has_plugin?("vagrant-hostmanager")
  puts "'vagrant-hostmanager' plugin is required"
  puts "This can be installed by running:"
  puts
  puts " vagrant plugin install vagrant-hostmanager"
  puts
  exit
end

Vagrant.configure(2) do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  config.vm.define "blog" do |node|
    node.vm.box = "jeqo/ansible-ubuntu14"
    node.vm.hostname = "sysco-middleware-blog"

    node.vm.network :private_network, :ip => '10.1.0.10'

    node.hostmanager.aliases = %w(sysco-middleware-blog.localdomain)

    node.vm.provision "ansible" do |ansible|
      ansible.playbook = "main.yml"
      # ansible.galaxy_role_file = "roles.yml"
    end

    node.trigger.after [:up, :reload] do
      run_remote "cd /vagrant && jekyll serve --detach"
    end
  end
end
