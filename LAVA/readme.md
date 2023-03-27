# LAVA
[RPC](http://lava.srgts.xyz:23357) | [API](http://lava.srgts.xyz:3317) | [gRPC](http://lava.srgts.xyz:9391)

Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential jq wget git htop curl screen bc -y
````
Install Go:
````
cd $HOME
VRS="1.19.2"
wget "https://golang.org/dl/go$VRS.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VRS.linux-amd64.tar.gz"
rm "go$VRS.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
````
Install CLI:
````
git clone https://github.com/lavanet/lava
cd lava
git checkout v0.8.1
make install
````
Let's check the version (current as of March 2023 - v0.8.1 commit: 910bfdf6fca1ba4030f4587c3f5af3382a8b5238):
````
lavad version --long
````
Set the correct chain (altruistic-1), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
lavad config chain-id lava-testnet-1
lavad init $MNK --chain-id lava-testnet-1
````
Download the current genesis file:
````
curl -s https://raw.githubusercontent.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T/main/testnet-1/genesis_json/genesis.json > $HOME/.lava/config/genesis.json
````
Let's check sum genesis file (current as of March 2023 - 72170a8a7314cb79bc57a60c1b920e26457769667ce5c2ff0595b342c0080d78):
````
sha256sum $HOME/.lava/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025ulava"|g' $HOME/.lava/config/app.toml
````
Adding seeds and peers:
````
seeds="3a445bfdbe2d0c8ee82461633aa3af31bc2b4dc0@prod-pnet-seed-node.lavanet.xyz:26656,e593c7a9ca61f5616119d6beb5bd8ef5dd28d62d@prod-pnet-seed-node2.lavanet.xyz:26656"
peers="f120685de6785d8ee0eadfca42407c6e10593e74@144.76.90.130:32656,e4ebf07ed08ff8ee26a9a903d63ad34d1f59393e@95.217.35.186:56656,0561fed6e88f2167979e379436529861527d859d@65.109.92.148:61256,e4c705abed2f76830652799d2eb386a9b876daff@168.119.52.60:26656,d3a466c4892943059b6b361e63eb0665ead5c574@147.135.222.170:5567"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.lava/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.lava/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.lava/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.lava/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.lava/config/app.toml
````
Edit block time parameter:
````
sed -i 's/create_empty_blocks = .*/create_empty_blocks = true/g' $HOME/.lava/config/app.toml
sed -i 's/create_empty_blocks_interval = ".*s"/create_empty_blocks_interval = "60s"/g' $HOME/.lava/config/app.toml
sed -i 's/timeout_propose = ".*s"/timeout_propose = "60s"/g' $HOME/.lava/config/app.toml
sed -i 's/timeout_commit = ".*s"/timeout_commit = "60s"/g' $HOME/.lava/config/app.toml
sed -i 's/timeout_broadcast_tx_commit = ".*s"/timeout_broadcast_tx_commit = "601s"/g' $HOME/.lava/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/lavad.service > /dev/null << EOF
[Unit]
Description=LAVA NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which lavad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
lavad tendermint unsafe-reset-all --home $HOME/.lava --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable lavad
sudo systemctl restart lavad
````
Checking the logs
````
sudo journalctl -u lavad -f -o cat
````
