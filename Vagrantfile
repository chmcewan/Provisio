#
# Creates a minimal base CentOS 7 Vagrant box with 
#
# 1) Kernel updates for VirtualBox guest additions
# 2) Provisio installed for provisioning the VM
# 3) Docker installed with corresponding base/centos7 image and provisio installed
#
# NB: This VM needs to be restarted during provisioning before being 
#     packaged for future Vagrantfiles. Follow on-screen instructions.
#

Vagrant.configure(2) do |config|

  config.vm.box = "centos/7"

  config.vm.provider "virtualbox" do |vb|
    vb.gui    = false
    vb.name   = "Centos7-box"
    vb.memory = 1024
    vb.cpus   = 1
  end

  # Explicitly disable directory syncing until Guest additions has been built and installed
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox", disabled: true
  config.vm.synced_folder ".", "/home/vagrant/sync", type: "virtualbox", disabled: true

  config.vm.provision "shell", inline: <<-SHELL

    if [ ! -f /tmp/firstrun ]; then

      # update kernel (for guest additions)
      yum -y upgrade

      touch /tmp/firstrun
      echo "\e[31mNeeds restart. Run vagrant reload --provision to continue...\e[39m"
    else

      yum -y update

      # prepare for install of virtualbox guest additions
      yum -y install epel-release
      yum -y install gcc
      yum -y install kernel-devel-$(uname -r)
      yum -y install bzip2

      # install Provisio so that projects based on this image always have it for provisioning
      curl -o /usr/bin/provisio https://raw.githubusercontent.com/chmcewan/Provisio/master/provisio
      chmod 755 /usr/bin/provisio

      # install docker
      curl https://get.docker.com/ | bash
      usermod -aG docker vagrant
      systemctl enable docker
      systemctl start docker

      # build base docker image
      # Need to embed because cant rely on synced filesystem
      cat <<EOF > /tmp/Dockerfile
FROM centos:7

ENV container docker

RUN yum -y update

# Install provisio
RUN curl -o /usr/bin/provisio https://raw.githubusercontent.com/chmcewan/Provisio/master/provisio
RUN chmod 755 /usr/bin/provisio
RUN yum -y install perl

RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ \$i == systemd-tmpfiles-setup.service ] || rm -f \$i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;

# docker run -ti -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 80:80 local/c7-systemd-httpd
VOLUME /sys/fs/cgroup
VOLUME /host

WORKDIR /host
CMD /usr/sbin/init

EOF
      docker build --rm=true -t base/centos7 /tmp
      
    fi

  SHELL
end
