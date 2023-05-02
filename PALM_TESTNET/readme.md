# PALM TESTNET

Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential jq wget git htop curl screen -y
sudo apt install openjdk-17-jre-headless -y
````
Install Besu:
````
wget https://hyperledger.jfrog.io/hyperledger/besu-binaries/besu/23.1.3/besu-23.1.3.tar.gz
tar -xvf besu-23.1.3.tar.gz
rm -rf besu-23.1.3.tar.gz
````
Download the current genesis file:
````
mkdir palm-node
cd palm-node
curl -O https://genesis-files.palm.io/uat/genesis.json
cd $HOME
````
Create a configuration file:
````
sudo tee /root/palm-node/config.toml > /dev/null << EOF
# Palm Testnet genesis file
genesis-file="/root/palm-node/genesis.json"

# Network bootnodes
bootnodes=["enode://7c6e935eca89b230002294420c10d645844419ac50c5fc03fa53bf24fd82600508f5a4d5b89f7690c7e8f9c5dc833605d60bb1dd35997669ab7f1fc274683803@54.162.14.76:30303","enode://2f5d0489e2bbbc495e3d38ae3df9cc0a47faf42818057d193f0f4863d44505277c3d1b9a863f7ad961830ef15a8f8b72ec52791f3cca5ef84284a29f82f2dd73@18.235.20.166:30303"]

# Data directory
data-path="/root/palm-node"

# Enable the JSON-RPCs
rpc-http-enabled=true

# Specify required API methods
rpc-http-api=["ETH","NET","WEB3","ADMIN","IBFT","TXPOOL","DEBUG","TRACE"]
EOF
````
Create a service file:
````
sudo tee /etc/systemd/system/palmd.service > /dev/null << EOF
[Unit]
Description=PALM TESTNET NODE
After=network-online.target
[Service]
User=$USER
ExecStart=/root/besu-23.1.3/bin/besu --config-file=/root/palm-node/config.toml --sync-mode=FULL --random-peer-priority-enabled=true --rpc-http-enabled=true --rpc-http-api=ETH,NET,WEB3,ADMIN,IBFT,TXPOOL,DEBUG,TRACE --rpc-ws-api=ETH,NET,WEB3,ADMIN,IBFT,TXPOOL,DEBUG,TRACE --rpc-ws-enabled --rpc-http-host=0.0.0.0 --rpc-ws-host=0.0.0.0 --host-allowlist=* --metrics-enabled --metrics-host=0.0.0.0 --rpc-http-cors-origins=* --rpc-http-max-active-connections=10000 --rpc-ws-max-active-connections=10000 --max-peers=100
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
