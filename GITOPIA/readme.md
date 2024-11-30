# GITOPIA

Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential jq wget git htop curl screen bc -y
````
Install Go:
````
cd $HOME
VRS="1.20.5"
wget "https://golang.org/dl/go$VRS.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VRS.linux-amd64.tar.gz"
rm "go$VRS.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
````
Install CLI:
````
cd $HOME
git clone https://github.com/gitopia/gitopia.git
cd gitopia
git checkout v5.1.0
make install
````
Let's check the version (current as of December 2024 - v5.1.0 | commit: 3b1ea32980e79ea74a1398f78a0388867e8d5fd4):
````
gitopiad version --long
````
Set the correct chain (gitopia), chooses his moniker and initialize node:
````
cd $HOME
MNK=GTP_NODE
gitopiad config chain-id gitopia
gitopiad init $MNK --chain-id gitopia
````
Download the current genesis file:
````
curl -sL https://github.com/gitopia/mainnet/raw/master/genesis.tar.gz > $HOME/genesis.tar.gz
tar -xzf $HOME/genesis.tar.gz
mv genesis.json $HOME/.gitopia/config/genesis.json
rm $HOME/genesis.tar.gz
````
Let's check sum genesis file (current as of December 2024 - 0cf5c55e6ea1fbcebccadba0f6dc0b83ac76d1b608487a06978956404ce33e66):
````
sha256sum $HOME/.gitopia/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001ulore"|g' $HOME/.gitopia/config/app.toml
````
Adding seeds and peers:
````
seeds="a4a69a62de7cb0feb96c239405aa247a5a258739@seeds.cros-nest.com:57656,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:11356"
peers="082e95b5d5351e68dcfb24dff802f9064cfd5a4c@65.109.92.241:51056,d1135f9f8e71c606a0f7a01c445550b836d0ec79@65.109.157.219:28656,a2d725392ea4cb4d596555bb6e56a073d140037b@194.163.171.231:26656,901c393d17c1e6094cbbc83c34f167a67bb5fab1@65.108.70.119:36656,112e976f58198f8da593fe4134bddd92cd0fbf55@65.21.192.90:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.gitopia/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.gitopia/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.gitopia/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.gitopia/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.gitopia/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/gitopiad.service > /dev/null << EOF
[Unit]
Description=GITOPIA NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which gitopiad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
gitopiad tendermint unsafe-reset-all --home $HOME/.gitopia --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable gitopiad
sudo systemctl restart gitopiad
````
Checking the logs
````
sudo journalctl -u gitopiad -f -o cat
````
