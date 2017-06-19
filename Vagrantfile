#
#  Copyright (c) 2016, The OpenThread Authors.
#  All rights reserved.
#
#  Redistribution and use in source and binary forms, with or without
#  modification, are permitted provided that the following conditions are met:
#  1. Redistributions of source code must retain the above copyright
#     notice, this list of conditions and the following disclaimer.
#  2. Redistributions in binary form must reproduce the above copyright
#     notice, this list of conditions and the following disclaimer in the
#     documentation and/or other materials provided with the distribution.
#  3. Neither the name of the copyright holder nor the
#     names of its contributors may be used to endorse or promote products
#     derived from this software without specific prior written permission.
#
#  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
#  AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
#  IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
#  ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
#  LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
#  CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
#  SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
#  INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
#  CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
#  ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
#  POSSIBILITY OF SUCH DAMAGE.
#

# -*- mode: ruby -*-
# vi: set ft=ruby :

# cribbed from https://github.com/adafruit/esp8266-micropython-vagrant
Vagrant.configure("2") do |config|
  config.vm.box = "debian/jessie64"

  # Virtualbox VM configuration.
  config.vm.provider "virtualbox" do |v|
    # extra memory for compilation
    v.memory = 2048
    # attach usb
    v.customize ["modifyvm", :id, "--usb", "on"]
    v.customize ["modifyvm", :id, "--usbehci", "on"]

    # Edit this to match your device
    # Run `VBoxManage list usbhost` to get all the pertinent info
    v.customize ["usbfilter", "add", "0",
      "--target", :id,
      "--name", "J-Link",
      "--manufacturer", "SEGGER",
      "--product", "J-Link",
      "--serialnumber", "000683767824"]
  end

  # downloads and configuration dependencies
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    echo "Installing dependencies..."

    # quiets some stdin errors
    export DEBIAN_FRONTEND=noninteractive

    sudo apt-get update

    # wpandtund runtime & build requirements
    sudo apt-get install -y build-essential git make autoconf autoconf-archive \
                            automake dbus libtool gcc g++ gperf flex bison texinfo \
                            ncurses-dev libexpat-dev python sed python-pip gawk \
                            libreadline6-dev libreadline6 libdbus-1-dev libboost-dev\
                            tayga bind9

    echo "Installing Basic Border Router"
    mkdir -p ~/src

    # Install wpantund
    echo "Installing Wpantund"
    git clone --recursive https://github.com/openthread/wpantund.git
    cd wpantund
    # Install dependencies
    ./bootstrap.sh
    ./configure --sysconfdir=/etc
    make -j4
    sudo make install

    # Set the default interface
    sudo sed -i '/Config:NCP:SocketPath "\/dev/i Config:NCP:SocketPath "/dev/ttyACM0"' /etc/wpantund.conf
    sudo sed -i 's/^#Config:Daemon:PrivDropToUser/Config:Daemon:PrivDropToUser/' /etc/wpantund.conf

    # Setting up forwarding
    sudo tee /etc/sysctl.d/60-otbr-ip-forward.conf <<EOF
net.ipv6.conf.all.forwarding = 1
net.ipv4.ip_forward = 1
EOF

    # Make sure ipfowarding is enabled
    # echo 1 | sudo tee /proc/sys/net/ipv6/conf/all/forwarding
    # echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

    # Set up tayga
    mkdir -p /var/db/tayga

    #Remove just in case
    sudo ip link del dev nat64

    sudo tayga --mktun
    sudo ip link set nat64 up
    sudo ip addr add 2001:db8:1::1 dev nat64          # replace with your router's address
    sudo ip addr add 192.168.255.1 dev nat64          # replace with your router's address
    sudo ip route add 2001:db8:1:ffff::/96 dev nat64  # from tayga.conf
    sudo ip route add 192.168.255.0/24 dev nat64      # from tayga.conf

    # TAYGA_DEFAULT=/etc/default/tayga
    # TAYGA_CONF=/etc/tayga.conf
    # TAYGA_IPV6_ADDR=fdaa:bb:1::1
    # NAT44_SERVICE=/etc/init.d/otbr-nat44
    # WLAN_IFNAMES=eth0

    sudo sed -i 's/^RUN="no"/RUN="yes"/' '/etc/default/tayga'
    # sudo sed -i 's/^prefix /##prefix /' 'etc/tayga.conf'
    # sudo sed -i 's/^# prefix 64:ff9b::\/96/prefix 64:ff9b::\/96/' $TAYGA_CONF
    # sudo sed -i '/^#ipv6-addr/a ipv6-addr '$TAYGA_IPV6_ADDR $TAYGA_CONF

    sudo tayga

    sudo iptables -F
    sudo iptables -t nat -F
    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

    #sudo iptables -A FORWARD -i eth0 -o nat64 -m state --state RELATED,ESTABLISHED -j ACCEPT
    #sudo iptables -A FORWARD -i nat64 -o eth0 -j ACCEPT

    # Add this to named.conf.options (http://ipvsix.me/?p=106)
    #     very simple BIND9 named.conf.options can look like this:
    #
    # options {
    #         directory "/var/cache/bind";
    #         auth-nxdomain no;
    #         listen-on-v6 { any; };
    #         allow-query { any; };
    #         dns64 2001:db8:1:ffff::/96 {
    #                 clients { any; };
    #         };
    # };

    # To start the daemon set the master key if needed
    # If working with nordic examples all of that is set to a default value
    # Network:Name = "OpenThread"
    # Network:XPANID = 0xDEAD00BEEF00CAFE
    # Network:Key = [00112233445566778899AABBCCDDEEFF]
    # NCP:Channel = 11
    # To get up and running without a fuss do this
    # sudo wpanctl set Network:PANID 0xabcd
    # sudo wpanctl attach

    # Gotta set the gateway and the router
    #
    # wpanctl config-gateway  -d
    # sleep 2
    # wpanctl add-route 2001:db8:1:ffff::

    ping 2001:db8:1:ffff:0:0:0808:0808

    # cd borderrouter
    # # Install dependencies
    # ./script/bootstrap
    # # Build and install border router and wpantund
    # ./script/setup

    echo "Border router setup complete!"
  SHELL

end
