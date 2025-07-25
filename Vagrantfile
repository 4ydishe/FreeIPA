Vagrant.configure("2") do |config|
  config.vm.box = "centos/stream8"

  config.vm.provider :virtualbox do |v|
    v.memory = 2048
    v.cpus = 1
  end

  boxes = [
    { :name => "ipa.otus.lan", :ip => "192.168.57.10" },
    { :name => "client1.otus.lan", :ip => "192.168.57.11" },
    { :name => "client2.otus.lan", :ip => "192.168.57.12" }
  ]

  boxes.each do |opts|
    config.vm.define opts[:name] do |machine|
      machine.vm.hostname = opts[:name]
      machine.vm.network "private_network", ip: opts[:ip]

      if opts[:name] == "ipa.otus.lan"
        machine.vm.provision "shell", inline: <<-SHELL
          set -eux

          # Timezone and chrony
          timedatectl set-timezone Europe/Moscow
          yum install -y chrony
          systemctl enable chronyd --now

          # Firewall and SELinux
          systemctl stop firewalld || true
          systemctl disable firewalld || true
          setenforce 0
          sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config

          # Hosts
          echo "192.168.57.10 ipa.otus.lan ipa" >> /etc/hosts

          # Fix CentOS 8 repo with vault
          sudo sed -i 's/^mirrorlist=/#mirrorlist=/g' /etc/yum.repos.d/CentOS-*.repo
          sudo sed -i 's|^#baseurl=http://mirror.centos.org|baseurl=https://vault.centos.org/8-stream|g' /etc/yum.repos.d/CentOS-*.repo
          sudo yum clean all
          sudo yum makecache

          # Enable FreeIPA module and install
          yum module enable idm:DL1 -y
          yum install -y @idm:DL1 ipa-server

          # Run IPA installer
          ipa-server-install -U \
            --hostname=ipa.otus.lan \
            --domain=otus.lan \
            --realm=OTUS.LAN \
            --ds-password=otus2022 \
            --admin-password=otus2022 \
            --no-ntp
          # Install EPEL and Ansible
          sudo curl -O https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
          sudo rpm -ivh epel-release-latest-8.noarch.rpm
          sudo yum clean all
          sudo yum makecache
          sudo yum install -y ansible
        SHELL
      end
    end
  end
end