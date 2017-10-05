# linuxkit-arm64-initrd
various linuxkit initrd images from examples

```
### as root

cd /root

apt-get -y update && \
apt-get -y upgrade && \
apt-get -y install openssl libcurl4-openssl-dev libxml2 libssl-dev libxml2-dev pinentry-curses xclip cmake build-essential pkg-config git libltdl7 qemu qemu-kvm

wget https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/arm64/docker-ce_17.09.0~ce-0~ubuntu_arm64.deb && \
dpkg -i docker-ce_17.09.0~ce-0~ubuntu_arm64.deb

useradd -m -G docker,kvm -s /bin/bash build
mkdir -p /home/build/.ssh
cp /root/.ssh/authorized_keys /home/build/.ssh
touch /home/build/.cloud-warnings.skip
chown -R build:build /home/build
chmod -R u=rw,go=,u+X /home/build/.ssh
echo 'build ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

### as build (SSH back in)

cd /home/build

wget https://storage.googleapis.com/golang/go1.9.1.linux-arm64.tar.gz
tar -zxf go1.9.1.linux-arm64.tar.gz
mv go goroot
mkdir gopath

cat >> /home/build/.profile <<'EOF'
export GOPATH=/home/build/gopath
export GOROOT=/home/build/goroot
export PATH=$PATH:/home/build/goroot/bin
EOF
. /home/build/.profile

go get -u github.com/moby/tool/cmd/moby

git clone https://github.com/linuxkit/linuxkit
cd ./linuxkit
make
sudo make install

cd /home/build/linuxkit/examples
moby build -pull -format kernel+initrd getty.yml

### back on your local workstation

scp build@PACKET_HOST:/home/build/linuxkit/examples/getty-* .
```
