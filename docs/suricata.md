# installs
sudo apt-get install libpcre3 libpcre3-dbg libpcre3-dev build-essential libpcap-dev 
                  libnet1-dev libyaml-0-2 libyaml-dev pkg-config zlib1g zlib1g-dev
                 libcap-ng-dev libcap-ng0 make libmagic-dev
                         libgeoip-dev liblua5.1-dev libhiredis-dev libevent-dev
                 python-yaml rustc cargo
sudo apt install libjansson-dev
git clone https://github.com/OISF/libhtp
sudo apt-get install libnspr4-dev libnss3-dev liblz4-dev
cargo install --force cbindgen
sudo cp $HOME/.cargo/bin/cbindgen /usr/local/bin

# builds
./autogen.h
./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var
make
sudo make install
sudo make install-conf

# run
sudo suricata -c suricata.yaml -i eth0
