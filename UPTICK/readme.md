# UPTICK
[RPC](http://uptick.srgts.xyz:26657) | [API](http://uptick.srgts.xyz:1317) | [gRPC](http://uptick.srgts.xyz:9090)

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
Set the correct chain (uptick_7000-2), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
uptickd config chain-id uptick_7000-2
uptickd init $MNK --chain-id uptick_7000-2
````
Download the current genesis file:
````
curl https://raw.githubusercontent.com/UptickNetwork/uptick-testnet/main/uptick_7000-2/genesis.json > $HOME/.uptickd/config/genesis.json
````
Let's check sum genesis file (current as of November 2022 - f96764c7ae1bc713b2acc87b5320f2d10ee26716b3daa6cc455cb3a3906f05c2):
````
sha256sum $HOME/.uptickd/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001auptick"|g' $HOME/.uptickd/config/app.toml
````
Adding seeds and peers:
````
seeds=""
peers="eecdfb17919e59f36e5ae6cec2c98eeeac05c0f2@peer0.testnet.uptick.network:26656"
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
State Sync:
````
sudo systemctl stop uptickd
uptickd tendermint unsafe-reset-all --home $HOME/.uptickd --keep-addr-book
SNAP_RPC="http://uptick.srgts.xyz:26657"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.uptickd/config/config.toml

sudo systemctl uptickd neutrond && sudo journalctl -u uptickd -f -o cat
````
