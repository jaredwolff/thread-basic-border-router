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

    echo "Basic Border Bouter setup complete!"
  SHELL

end
