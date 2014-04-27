# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$cleanup_script = <<SCRIPT
:
: clean-up-src
(
    set -e
    set -x

    mkdir /tmp/.empty
    time rsync -a --delete /tmp/.empty/ /usr/local/src/
    rmdir /tmp/.empty
)

SCRIPT

$script = <<SCRIPT
:
: kernel-module-development
(
    rpm -q kernel-devel  >/dev/null || rpm -ivh http://mirror.centos.org/centos/6/os/x86_64/Packages/kernel-devel-2.6.32-431.el6.x86_64.rpm
    rpm -q ncurses-devel >/dev/null || yum install -y ncurses-devel
    which gcc  >/dev/null || yum groupinstall -y 'Development Tools'
    which git  >/dev/null || yum install -y git
    which wget >/dev/null || yum install -y wget
    which bc   >/dev/null || yum install -y bc
    which vim  >/dev/null || yum install -y vim-enhanced;
)

:
: build-kernel
(
     set -e
     set -x

     # $1 is ENV["KERNEL_VERSION"]
     KERNEL_VERSION=$1

     cd /usr/local/src

     if [ "$KERNEL_VERSION" = "" ]; then
         KERNEL_VERSION=$( curl -s https://www.kernel.org | grep latest_link -A1 | tail -1 | perl -nle '/>([^<]+)</; print $1' )
     fi

     if [ "$KERNEL_VERSION" = $( uname -r ) ]; then
         echo "*** current running kernel version is latest ($KERNEL_VERSION) ***"
         exit 0
     fi

     if [ ! -d linux-${KERNEL_VERSION} ]; then
         time rsync -v rsync://ftp.kernel.org/pub/linux/kernel/v3.x/linux-${KERNEL_VERSION}.tar.xz .
         time tar xf linux-${KERNEL_VERSION}.tar.xz
     fi

     cd linux-${KERNEL_VERSION}
     yes "" | make oldconfig

     time make -j4
     time make modules
     time make modules_install
     time make install

     perl -pi'' -e "s/default=\\d/default=0/" /boot/grub/grub.conf

     /sbin/reboot
)
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box     = "CentOS 6.5 x86_64"

  config.vm.box_url = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.1/centos65-x86_64-20131205.box"
  config.vm.provision "shell" do |sh|
    if ENV["CLEANUP"]
      sh.inline = $cleanup_script
    else
      sh.inline = $script
    end
    # If KERNEL_VERSION is empty string, latest version is specified.
    sh.args   = ENV["KERNEL_VERSION"]
  end

  config.vm.provider :virtualbox do |vb|
    vb.gui = true
    vb.customize ["modifyvm", :id, "--memory", 2000]
    vb.customize ["modifyvm", :id, "--cpus", 4]
    vb.customize ["modifyvm", :id, "--ioapic", "on"]
    vb.customize ["modifyvm", :id, "--hpet", "on"]
    vb.customize ["modifyvm", :id, "--acpi", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
  end

  config.persistent_storage.enabled      = true
  config.persistent_storage.location     = "disks/sourcehdd.vdi"
  config.persistent_storage.size         = 50000
  config.persistent_storage.filesystem   = 'ext4'
  config.persistent_storage.mountpoint   = '/usr/local/src'
end
