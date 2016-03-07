CEPH_RELEASE = "infernalis"


# Vagrant.require_plugin "vagrant-vbguest"

Vagrant.configure(2) do |config|
  config.vm.box = "debian/jessie64"

  config.vm.synced_folder ".", "/vagrant", type: ""

  config.vm.provision :shell, inline: <<-SHELL
    apt-get -qy update
    apt-get -qy install ntp openssh-server

    systemctl start ntp
    systemctl enable ntp
  SHELL

  config.vm.network :public_network

  config.vm.provider "virtualbox" do |vb|
    #vb.gui = false
    vb.memory = 2048
    vb.cpus = 1
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

  (1..3).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.host_name = "node#{i}"

      node.vm.provision :shell, privileged: false, inline: <<-SHELL
        if [ ! -f /vagrant/id_rsa.pub ]; then
          ssh-keygen -t rsa -N '' -f /vagrant/id_rsa -C "ceph-vagrant"
        fi
        cat /vagrant/id_rsa.pub >>~/.ssh/authorized_keys
      SHELL
    end
  end

  config.vm.define "admin-node", primary: true do |node|
    node.vm.host_name = "admin-node"

    node.vm.provision :shell, inline: <<-SHELL
      curl https://download.ceph.com/keys/release.asc | apt-key add -
      echo deb http://download.ceph.com/debian-#{CEPH_RELEASE}/ $(lsb_release -sc) main >/etc/apt/sources.list.d/ceph.list
      apt-get -qy update
      apt-get -qy install ceph-deploy
    SHELL

    node.vm.provision :shell, privileged: false, inline: <<-SHELL
      set -e  # stop on any error
      set -o verbose  # echo every command

      mv /vagrant/id_rsa ~/.ssh/
      cat /vagrant/id_rsa.pub >>~/.ssh/authorized_keys
      rm /vagrant/id_rsa.pub

      echo 'Host *node*'                 >~/.ssh/config
      echo '  User vagrant'             >>~/.ssh/config
      echo '  StrictHostKeyChecking no' >>~/.ssh/config

      cd /vagrant

      ceph-deploy new node1
      echo 'osd_pool_default_size = 2' >>ceph.conf

      ceph-deploy install --release #{CEPH_RELEASE} admin-node node{1..3}

      ceph-deploy mon create-initial

      ceph-deploy admin admin-node node{1..3}
      sudo chmod +r /etc/ceph/ceph.client.admin.keyring
      for node in node{1..3}; do
        ssh $node sudo chmod +r /etc/ceph/ceph.client.admin.keyring
      done

      for node in node{2..3}; do
        ssh $node sudo mkdir /var/local/osd
        ssh $node sudo chown ceph: /var/local/osd
        ceph-deploy osd prepare $node:/var/local/osd
        ceph-deploy osd activate $node:/var/local/osd
      done

      sleep 5
      ceph health

      ceph-deploy mds create node1
      ceph-deploy rgw create node1
    SHELL
  end

  config.vm.provision :shell, privileged: false, inline: <<-SHELL
    echo 'cd /vagrant' >>~/.bashrc
  SHELL
end


ENV["LC_ALL"] = "en_US.UTF-8"


# References:
# http://docs.ceph.com/docs/master/start/quick-start-preflight/
# http://docs.ceph.com/docs/master/start/quick-ceph-deploy/
# https://github.com/infn-bari-school/ceph/wiki/How-to-install-Ceph-cluster-%28version-Infernalis%29
# http://superuser.com/questions/125324/how-can-i-avoid-sshs-host-verification-for-known-hosts
# http://superuser.com/questions/1022926/how-to-script-install-of-virtualbox-guest-additions-to-a-vagrant-debian-box
