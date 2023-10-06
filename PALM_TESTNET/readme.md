# PALM TESTNET

Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential jq wget git htop curl screen bc -y
````
Install Go:
````
ver="1.20.5"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
````
Download the current genesis file:
````
mkdir $HOME/.palm-node
curl -L https://raw.githubusercontent.com/gateway-fm/polygon-edge/palm-migration/genesis-testnet.json > $HOME/.palm-node/genesis.json
````
Install Polygon-Edge:
````
cd $HOME
git clone https://github.com/gateway-fm/polygon-edge
cd $HOME/polygon-edge
git checkout 1.1.33
make build
chmod +x polygon-edge
cp polygon-edge $HOME/go/bin/
````
Let's check the version 1.1.33 (commit hash = 38af278323110b01dc930136574857a7aa55e817)
````
polygon-edge version
````
Create keys for node:
````
polygon-edge polybft-secrets --insecure --data-dir=$HOME/.palm-node/
````
Create a service file:
````
sudo tee /etc/systemd/system/palmd.service > /dev/null << EOF
[Unit]
Description=PALM TESTNET NODE
After=network-online.target
[Service]
User=$USER
ExecStart=/root/go/bin/polygon-edge server --data-dir=/root/.palm-node/ --chain=/root/.palm-node/genesis.json --libp2p "0.0.0.0:30301" --devp2p "0.0.0.0:30302" --jsonrpc="0.0.0.0:8545"
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
sudo systemctl enable palmd
sudo systemctl restart palmd
````
Checking the logs
````
sudo journalctl -u palmd -f -o cat
````
