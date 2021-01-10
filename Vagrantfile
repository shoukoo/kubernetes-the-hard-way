NUM_MASTER_NODE = 2
NUM_WORKER_NODE = 2

IP_NW = "192.168.5."
MASTER_IP_START = 10
NODE_IP_START = 20
LB_IP_START = 30

$setup_hosts = <<-SCRIPT
# The purpose of this script is to set the right host IP address for the machine
# and also the hostnames for other nodes.
INTERFACE="enp0s8"

# Get the IP address of enp0s8 interface
ADDRESS=$( ip -4 addr show $INTERFACE | grep "inet" | awk '{print $2}' | awk -F"/" '{print $1}')

# Associate IP to the $HOSTNAme
sed -e "s/^.*${HOSTNAME}.*/${ADDRESS} ${HOSTNAME} ${HOSTNAME}.local/" -i /etc/hosts

# remove ubuntu-bionic entry
sed -e '/^.*ubuntu-bionic.*/d' -i /etc/hosts

# Update /etc/hosts about other hosts
cat >> /etc/hosts <<EOF
192.168.5.11  master-1
192.168.5.12  master-2
192.168.5.21  worker-1
192.168.5.22  worker-2
192.168.5.30  lb
EOF

# Disable name server
sed -i -e 's/#DNS=/DNS=8.8.8.8/' /etc/systemd/resolved.conf
service systemd-resolved restart
SCRIPT

$setup_worker = <<-SCRIPT
#Install Docker
cd /tmp
curl -fsSL https://get.docker.com -o get-docker.sh
sh /tmp/get-docker.sh

sysctl net.bridge.bridge-nf-call-iptables=1
SCRIPT

Vagrant.configure("2") do | config |

  config.vm.box = "ubuntu/bionic64" #Upgrade to 20.04 later

  config.vm.box_check_update = false

  (1..NUM_MASTER_NODE).each do |i|
    config.vm.define "master-#{i}" do | node |
      # Name shown in the GUI
      node.vm.provider "virtualbox" do |vb|
        vb.name = "kubernetes-ha-master-#{i}"
        vb.memory = 2048
        vb.cpus = 2
      end

      node.vm.hostname = "master-#{i}"
      node.vm.network :private_network, ip: IP_NW + "#{MASTER_IP_START + i }"
      node.vm.network "forwarded_port", guest: 22, host: "#{2710 + i}"
      node.vm.provision "shell", inline: $setup_hosts

      #TODO Remove this later
      node.vm.provision "file", source: "./cert_verify.sh", destination: "$HOME/"
    end
  end

  config.vm.define "loadbalancer" do |node|
    node.vm.provider "virtualbox" do |vb|
      vb.name = "kubernetes-ha-lb"
      vb.memory = 512
      vb.cpus = 1
    end
    node.vm.hostname = "loadbalancer"
    node.vm.network :private_network, ip: IP_NW + "#{LB_IP_START}"
    node.vm.network :forwarded_port, guest: 22, host: 2730

    node.vm.provision "shell", inline: $setup_hosts
  end

  (1..NUM_WORKER_NODE).each do |i|
    config.vm.define "worker-#{i}" do |node|
      node.vm.provider "virtualbox" do |vb|
        vb.name = "kubernetes-ha-worker-#{i}"
        vb.memory = 512
        vb.cpus = 1
      end

      node.vm.hostname = "worker-#{i}"
      node.vm.network :private_network, ip: IP_NW + "#{NODE_IP_START + i }"
      node.vm.network :forwarded_port, guest: 22, host: "#{2720 + i }"
      node.vm.provision "shell", inline: $setup_hosts
      node.vm.provision "shell", inline: $setup_worker
      #TODO Remove this later
      node.vm.provision "file", source: "./cert_verify.sh", destination: "$HOME/"
    end
  end
end
