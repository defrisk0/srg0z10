# ANDROMEDA
[RPC](http://andromeda.srgts.xyz:26657) | [API](http://andromeda.srgts.xyz:1317) | [gRPC](http://andromeda.srgts.xyz:9091)

Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential jq wget git htop curl screen bc -y
````
Install Go:
````
cd $HOME
VRS="1.19.5"
wget "https://golang.org/dl/go$VRS.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VRS.linux-amd64.tar.gz"
rm "go$VRS.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
````
Install CLI:
````
git clone https://github.com/andromedaprotocol/andromedad.git
cd andromedad
git checkout galileo-3-v1.1.0-beta1
make install
````
Let's check the version (current as of February 2023 - galileo-3-v1.1.0-beta1 commit: b3f8d880dfcdb3265d321e465b47b04071d9480f):
````
andromedad version --long
````
Set the correct chain (galileo-3), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
andromedad config chain-id galileo-3
andromedad init $MNK --chain-id galileo-3
````
Download the current genesis file:
````
curl -L https://raw.githubusercontent.com/andromedaprotocol/testnets/galileo-3/genesis.json > $HOME/.andromedad/config/genesis.json
````
Let's check sum genesis file (current as of February 2023 - 29ad5850ed21e01416b5df103653d4380570380e61183770c5eaee5681b7bde2):
````
sha256sum $HOME/.andromedad/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uandr"|g' $HOME/.andromedad/config/app.toml
````
Adding seeds and peers:
````
seeds=""
peers="5f7a675b1d496e73f71b3c69a909091dc49b8366@136.243.136.241:14656,1e03ee63334167d21af85ff1a3835c687dc1c1cb@116.202.227.117:14756,064497a6f023caa1e5f1482425576540c22476fb@65.21.133.114:56656,c45d01b216a7f24a06448a47b6cf19a42e74c29b@65.21.170.3:32656,247f3c2bed475978af238d97be68226c1f084180@88.99.164.158:4376"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.andromedad/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.andromedad/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.andromedad/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.andromedad/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.andromedad/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/andromedad.service > /dev/null << EOF
[Unit]
Description=ANDROMEDA NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which andromedad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
andromedad tendermint unsafe-reset-all --home $HOME/.andromedad --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable andromedad
sudo systemctl restart andromedad
````
Checking the logs
````
sudo journalctl -u andromedad -f -o cat
````
