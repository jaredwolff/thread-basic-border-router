## Install

    vagrant plugin install vagrant-vbguest

## Commands that are run

  # Set the default interface
  sudo sed -i '/Config:NCP:SocketPath "\/dev/i Config:NCP:SocketPath "/dev/ttyACM0"' /etc/wpantund.conf

  # Setting up forwarding
  sudo tee /etc/sysctl.d/60-otbr-ip-forward.conf <<EOF
  net.ipv6.conf.all.forwarding = 1
  net.ipv4.ip_forward = 1
  EOF

  # Make sure ipfowarding is enabled
  # echo 1 | sudo tee /proc/sys/net/ipv6/conf/all/forwarding
  # echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

  # Kill tayga
  sudo killall tayga

  # Set up tayga
  sudo rm -r /var/spool/tayga
  sudo mkdir -p /var/spool/tayga/
  sudo touch /var/spool/tayga/dynamic.map

  #Remove just in case
  sudo ip link del dev nat64

  sudo tayga --mktun
  sudo ip link set nat64 up
  sudo ip addr add 2001:db8:1::1 dev nat64          # replace with your router's address
  sudo ip addr add 192.168.255.1 dev nat64          # replace with your router's address
  sudo ip route add 2001:db8:1:ffff::/96 dev nat64  # from tayga.conf
  sudo ip route add 192.168.255.0/24 dev nat64      # from tayga.conf

  sudo sed -i 's/^RUN="no"/RUN="yes"/' '/etc/default/tayga'

  sudo tayga

  sudo echo 1 > /proc/sys/net/ipv4/conf/all/forwarding
  sudo echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
  sudo echo 0 > /proc/sys/net/ipv6/conf/eth0/forwarding

  sudo iptables -F
  sudo iptables -t nat -F
  sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

  sudo iptables -A FORWARD -i eth0 -o nat64 -m state --state RELATED,ESTABLISHED -j ACCEPT
  sudo iptables -A FORWARD -i nat64 -o eth0 -j ACCEPT

  sudo /etc/init.d/bind9 reload

  # Add this to named.conf.options (http://ipvsix.me/?p=106)
  #     very simple BIND9 named.conf.options can look like this:
  #
  # options {
  #         directory "/var/cache/bind";
  #         auth-nxdomain no;
  #         listen-on-v6 { any; };
  #         allow-query { any; };
  #         allow-query-on { 2001:db8:1:ffff::1/128 };
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

  ping6 2001:db8:1:ffff:0:0:0808:0808

  # cd borderrouter
  # # Install dependencies
  # ./script/bootstrap
  # # Build and install border router and wpantund
  # ./script/setup
