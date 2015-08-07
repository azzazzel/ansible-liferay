
Vagrant.configure('2') do |config|
  config.vm.define 'testLiferay' do |srv|
    srv.vm.box = "ubuntu/vivid64"
    srv.vm.network :private_network, ip: '192.168.121.121'
    srv.vm.provision 'ansible' do |ansible|
      ansible.playbook = 'test.yml'
      ansible.inventory_path = 'test.inventory'
      ansible.host_key_checking = false
    end
  end
end