# UPTICK
Explorer:

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
curl -L -k https://github.com/UptickNetwork/uptick/releases/download/v0.2.4/uptick-linux-amd64-v0.2.4.tar.gz
tar -xvzf uptick-linux-amd64-v0.2.4.tar.gz
sudo mv -f uptick-linux-amd64-v0.2.4/uptickd /usr/local/bin/uptickd
chmod +x /usr/local/bin/uptickd
rm -rf uptick-linux-amd64-v0.2.4
rm -rf uptick-linux-amd64-v0.2.4.tar.gz
````
Let's check the version (current as of November 2022 - v0.2.4 commit: 6e6b1143af1a159249d112d190bf143536b1b1f2):
````
uptickd version --long
````
Set the correct chain (uptick_7000-1), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
uptickd config chain-id uptick_7000-1
uptickd init $MNK --chain-id uptick_7000-1
````
Download the current genesis file:
````
curl https://raw.githubusercontent.com/UptickNetwork/uptick-testnet/main/uptick_7000-1/genesis.json > $HOME/.uptickd/config/genesis.json
````
Let's check sum genesis file (current as of November 2022 - 9c2a5a9eb74103e3a9ae0599f66b9e665bdd7d67c178ab8308f853602b73be75):
````
sha256sum $HOME/.uptickd/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001auptick"|g' $HOME/.uptickd/config/app.toml
````
Adding seeds and peers:
````
seeds="61f9e5839cd2c56610af3edd8c3e769502a3a439@seed0.testnet.uptick.network:26656"
peers="61f9e5839cd2c56610af3edd8c3e769502a3a439@seed0.testnet.uptick.network:26656,5726ef5d4b2258bad3e9fd0e09708d92f791dbaa@116.202.236.115:26656,e24bde7fe207160442fe6b93ee376a739def5757@51.222.248.153:26656,3dbbfac16932869e66e44a9ef443102e6677cf82@154.12.236.153:11656,95231536864d0ce318d6f8b70c744d8179b2cb58@144.76.224.246:46656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.uptickd/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.uptickd/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.uptickd/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.uptickd/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/uptickd.service > /dev/null << EOF
[Unit]
Description=UPTICK NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which uptickd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
uptickd tendermint unsafe-reset-all --home $HOME/.uptickd --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable uptickd
sudo systemctl restart uptickd
````
Checking the logs
````
sudo journalctl -u uptickd -f -o cat
````
RPC:    http://uptick.srgts.xyz:26657

API:    http://uptick.srgts.xyz:1317

gRPC:   http://uptick.srgts.xyz:9090

STATE SYNC:
````
````
