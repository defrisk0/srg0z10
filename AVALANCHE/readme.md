# AVALANCHE

Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential jq wget git htop curl screen bc -y
````
Install Go:
````
cd $HOME
VRS="1.21.5"
wget "https://golang.org/dl/go$VRS.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VRS.linux-amd64.tar.gz"
rm "go$VRS.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
````
Install CLI:
````
git clone https://github.com/ava-labs/avalanchego.git
cd avalanchego
./scripts/build.sh
````
Let's check the version (current as of December 2023 - v1.10.17, commit: 7623ffd4be915a5185c9ed5e11fa9be15a6e1f00):
````
avalanchego --version
````
Create a service file:
````
sudo tee /etc/systemd/system/avalanchego.service > /dev/null << EOF
[Unit]
Description=AVALANCHE NODE
After=network-online.target
[Service]
User=root
ExecStart=/root/go/bin/avalanchego --http-host 0.0.0.0
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable avalanchego
sudo systemctl restart avalanchego
````
Checking the logs
````
sudo journalctl -u avalanchego -f -o cat
````
