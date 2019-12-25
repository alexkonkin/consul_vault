VAGRANTFILE_API_VERSION = "2"

$configure_hosts = <<-SCRIPT
  sudo apt install mc -y
  sudo mv -v /etc/hosts /etc/hosts_$(date +"%Y_%m_%d_%H_%M_%S")
  echo -e "192.168.1.200       ansible1\n192.168.1.11       vault-node1\n192.168.1.12       vault-node2\n172.16.94.11       consul-node1\n172.16.94.12       consul-node2\n172.16.94.13       consul-node3" >> /etc/hosts
SCRIPT

$configure_consul_server = <<-SCRIPT
  node_number=$1

  echo "consul node_number is :"$node_number

  useradd consul
  mkdir -pv /var/consul/data/
  chown -Rv consul:consul /var/consul/data/
  mkdir -pv /usr/local/etc/consul/
  cp -pv /vagrant/downloads/consul /usr/local/bin/

  case $node_number in
    1)
      cp -pv /vagrant/consul1/consul.service /etc/systemd/system/
      cp -pv /vagrant/consul1/consul_node1.json /usr/local/etc/consul/
    ;;
    2)
      cp -pv /vagrant/consul2/consul.service /etc/systemd/system/
      cp -pv /vagrant/consul2/consul_node2.json /usr/local/etc/consul/
    ;;
    3)
      cp -pv /vagrant/consul3/consul.service /etc/systemd/system/
      cp -pv /vagrant/consul3/consul_node3.json /usr/local/etc/consul/
    ;;
  esac

  sudo systemctl start consul
  sudo systemctl enable consul
  sudo systemctl status consul

SCRIPT

$configure_consul_agent = <<-SCRIPT
  node_number=$1

  echo "consul agent node_number is :"$node_number

  useradd consul
  mkdir -pv /var/consul/data/
  chown -Rv consul:consul /var/consul/data/
  mkdir -pv /usr/local/etc/consul/
  cp -pv /vagrant/downloads/consul /usr/local/bin/

  case $node_number in
    1)
      cp -pv /vagrant/consul_a1/consul.service /etc/systemd/system/
      cp -pv /vagrant/consul_a1/consul_agent1.json /usr/local/etc/consul/
    ;;
    2)
      cp -pv /vagrant/consul_a2/consul.service /etc/systemd/system/
      cp -pv /vagrant/consul_a2/consul_agent2.json /usr/local/etc/consul/
    ;;
  esac

  sudo systemctl start consul
  sudo systemctl enable consul
  sudo systemctl status consul

SCRIPT

$configure_vault_server = <<-SCRIPT
  node_number=$1

  echo "vault server, node_number is :"$node_number

  useradd vault
  mkdir -pv /usr/local/etc/vault/
  cp -pv /vagrant/downloads/vault /usr/local/bin/

  case $node_number in
    1)
      cp -pv /vagrant/vault1/vault.service /etc/systemd/system/
      cp -pv /vagrant/vault1/vault_server.hcl /usr/local/etc/vault/
    ;;
    2)
      cp -pv /vagrant/vault2/vault.service /etc/systemd/system/
      cp -pv /vagrant/vault2/vault_server.hcl /usr/local/etc/vault/
    ;;
  esac
  sudo systemctl daemon-reload
  sudo systemctl start vault
  sudo systemctl enable vault
  sudo systemctl status vault

SCRIPT

$unseal_vault_server = <<-SCRIPT
  node_number=$1

  echo "vault server, node_number is :"$node_number

  sudo systemctl restart vault
  sleep 2
  case $node_number in
    1)
       export VAULT_ADDR=http://127.0.0.1:8200
       sudo -E vault operator init > /vagrant/vault.txt
       for i in $(cat /vagrant/vault.txt |grep 'Unseal Key'|awk -F: '{print $2}');do echo $i;vault operator unseal $i; done
    ;;
    2)
       export VAULT_ADDR=http://127.0.0.1:8200
       for i in $(cat /vagrant/vault.txt |grep 'Unseal Key'|awk -F: '{print $2}');do echo $i;vault operator unseal $i; done
    ;;
  esac
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "ansible1" do |ansible1|
    ansible1.vm.box = "bento/ubuntu-18.04"
    ansible1.vm.hostname = "ansible1"
    ansible1.vm.network :private_network, ip: "192.168.1.200"
    #config.vm.provision "shell" do |script|
    #   script.inline = $install_keepalived_haproxy
    #   script.args = ["MASTER","101","192.168.1.11","192.168.1.12"]
    #end
   config.vm.provision "configure_hosts", type: "shell" do |shell|
       shell.inline = $configure_hosts
    end
  end

  config.vm.define "vault1" do |vault1|
    vault1.vm.box = "bento/ubuntu-18.04"
    vault1.vm.hostname = "vault-node1"
    vault1.vm.network :private_network, ip: "192.168.1.11"
    #config.vm.provision "shell" do |script|
    #   script.inline = $install_keepalived_haproxy
    #   script.args = ["MASTER","101","192.168.1.11","192.168.1.12"]
    #end
   config.vm.provision "configure_hosts", type: "shell" do |shell|
       shell.inline = $configure_hosts
    end
    config.vm.provision "configure_consul_agent", type: "shell", run: "never" do |shell|
       shell.inline = $configure_consul_agent
       shell.args = ["1"]
    end
    config.vm.provision "configure_vault_server", type: "shell", run: "never" do |shell|
       shell.inline = $configure_vault_server
       shell.args = ["1"]
    end
    config.vm.provision "unseal_vault_server", type: "shell", run: "never" do |shell|
       shell.inline = $unseal_vault_server
       shell.args = ["1"]
    end
  end

  config.vm.define "vault2" do |vault2|
    vault2.vm.box = "bento/ubuntu-18.04"
    vault2.vm.hostname = "vault-node2"
    vault2.vm.network :private_network, ip: "192.168.1.12"
    #config.vm.provision "shell" do |script|
    #   script.inline = $install_keepalived_haproxy
    #   script.args = ["BACKUP","100","192.168.1.12","192.168.1.11"]
    #end
    config.vm.provision "configure_hosts", type: "shell" do |shell|
       shell.inline = $configure_hosts
    end
    config.vm.provision "configure_consul_agent", type: "shell", run: "never" do |shell|
       shell.inline = $configure_consul_agent
       shell.args = ["2"]
    end
    config.vm.provision "configure_vault_server", type: "shell", run: "never" do |shell|
       shell.inline = $configure_vault_server
       shell.args = ["2"]
    end
    config.vm.provision "unseal_vault_server", type: "shell", run: "never" do |shell|
       shell.inline = $unseal_vault_server
       shell.args = ["2"]
    end
  end

  config.vm.define "consul1" do |consul1|
    consul1.vm.box = "bento/ubuntu-18.04"
    consul1.vm.hostname = "consul-node1"
    consul1.vm.network :private_network, ip: "172.16.94.11"
    #config.vm.provision "shell" do |script|
       #script.inline = $install_consul_cluster
       #script.args = ["consul-node1","172.16.94.11","true"]
    #end
    #config.vm.provision "shell" do |script|
    #   script.inline = $install_hosts
    #end
    config.vm.provision "configure_hosts", type: "shell" do |shell|
       shell.inline = $configure_hosts
    end
    config.vm.provision "configure_consul_server", type: "shell", run: "never" do |shell|
       shell.inline = $configure_consul_server
       shell.args = ["1"]
    end
  end

  config.vm.define "consul2" do |consul2|
    consul2.vm.box = "bento/ubuntu-18.04"
    consul2.vm.hostname = "consul-node2"
    consul2.vm.network :private_network, ip: "172.16.94.12"
    #config.vm.provision "shell" do |script|
    #   script.inline = $install_consul_cluster
    #   script.args = ["consul-node2","172.16.94.12","true"]
    #end
    config.vm.provision "configure_hosts", type: "shell" do |shell|
       shell.inline = $configure_hosts
    end
    config.vm.provision "configure_consul_server", type: "shell", run: "never" do |shell|
       shell.inline = $configure_consul_server
       shell.args = ["2"]
    end
  end

  config.vm.define "consul3" do |consul3|
    consul3.vm.box = "bento/ubuntu-18.04"
    consul3.vm.hostname = "consul-node3"
    consul3.vm.network :private_network, ip: "172.16.94.13"
    #config.vm.provision "shell" do |script|
    #   script.inline = $install_consul_cluster
    #   script.args = ["consul-node3","172.16.94.13","true"]
    #end
    config.vm.provision "configure_hosts", type: "shell" do |shell|
       shell.inline = $configure_hosts
    end
    config.vm.provision "configure_consul_server", type: "shell", run: "never" do |shell|
       shell.inline = $configure_consul_server
       shell.args = ["3"]
    end
  end

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "512"]
    vb.customize ["modifyvm", :id, "--cpus", "1"]
  end
end
