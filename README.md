# srsRAN-Project-E2E
Open5GS 5G Core + srsRAN gNB + srsUE

Reference Link: https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#srsran-gnb-with-srsue

## Hardware and Software Overview
For this application note, the following hardware and software are used:

- VM with Ubuntu 22.04.1 LTS
- srsRAN Project -> for srsGNB
- srsRAN 4G (23.11 or later) --> for srsUE
- Open5GS 5G Core --> Docker based setup
- ZeroMQ --> networking library to transfer radio samples between applications( i.e virtual radio)

srsRAN deployment

### Prerequisites


#### Docker
```
sudo apt install -y git net-tools putty

# https://docs.docker.com/engine/install/ubuntu/
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your username to the docker group, otherwise you will have to run in sudo mode.
sudo usermod -a -G docker $(whoami)
```

#### ZeroMQ Installation
sudo apt-get install libzmq3-dev

#### srsRAN 4G prerequiisites

```
sudo apt-get install build-essential cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev 
```

#### srsRAN Project prerequiaites

```
sudo apt-get install cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev
```

--------------------


# srsRAN 4G ( for srsUE)

```
cd ~
git clone https://github.com/srsRAN/srsRAN_4G.git
cd srsRAN_4G
mkdir build
cd build
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON
make -j `nproc`
make test # optional
```

# srsRAN project (for 5G RAN)

```
cd ~
git clone https://github.com/srsran/srsRAN_Project.git
cd srsRAN_Project
mkdir build
cd build
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON
make -j `nproc`
```

# open5gs 5gc on docker


cd ~/srsRAN_Project/docker
docker compose up --build 5gc -d


# run gnb 

If you have built srsRAN Project from source and have not installed it, then you can run the gNB from: /srsRAN_Project/build/apps/gnb. In this folder you will find the gNB application binary.


cd ~/srsRAN_Project/build/apps/gnb
sudo ./gnb -c  ~/gnb_zmq.yaml

wget https://docs.srsran.com/projects/project/en/latest/_downloads/a7c34dbfee2b765503a81edd2f02ec22/gnb_zmq.yaml
wget https://docs.srsran.com/projects/project/en/latest/_downloads/fbb79b4ff222d1829649143ca4cf1446/ue_zmq.conf


# run srsUe

sudo ip netns add ue1
sudo ip netns list


cd ~/srsRAN_4G/build/srsue/src
sudo ./srsue ~/ue_zmq.conf 


sudo ./srsue/src/srsue --rf.device_name=zmq --rf.device_args="tx_port=tcp://*:2001,rx_port=tcp://localhost:2000,id=ue,base_srate=23.04e6" --gw.netns=ue1

# ping test

reference: https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#testing-the-network


ip ro add 10.45.0.0/16 via 10.53.1.2
ip r
ip netns exec ue1 ip route add default via 10.45.1.1 dev tun_srsue
ip netns exec ue1 ip r

Uplink:
ping 10.45.1.1
Downlink:
ping 10.45.1.2


