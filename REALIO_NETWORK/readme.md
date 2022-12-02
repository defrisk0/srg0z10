# REALIO NETWORK
[RPC](http://realio.srgts.xyz:37657) | [API](http://realio.srgts.xyz:3717) | [gRPC](http://realio.srgts.xyz:3790)

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
git clone https://github.com/realiotech/realio-network.git
cd realio-network
git checkout tags/v0.6.2
make install
````
Let's check the version (current as of December 2022 - v0.1.0 commit: 97eb6314360efb874ce1b5bf96b97db771188b11):
````
realio-networkd version --long
````
Set the correct chain (realionetwork_1110-2), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
realio-networkd init $MNK --chain-id realionetwork_1110-2
````
Download the current genesis file:
````
curl -s https://raw.githubusercontent.com/n0okOne/testnets-7/main/realionetwork_1110-2/genesis.json > $HOME/.realio-network/config/genesis.json
````
Let's check sum genesis file (current as of December 2022 - a2f8fae48eb019720ef78524d683a9ca22884dd4e9ba4f8d5b3ac10db1275183):
````
sha256sum $HOME/.realio-network/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001ario"|g' $HOME/.realio-network/config/app.toml
````
Adding seeds and peers:
````
seeds="aa194e9f9add331ee8ba15d2c3d8860c5a50713f@143.110.230.177:26656"
peers="aa194e9f9add331ee8ba15d2c3d8860c5a50713f@143.110.230.177:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.realio-network/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.realio-network/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.realio-network/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.realio-network/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/realio-networkd.service > /dev/null << EOF
[Unit]
Description=Realio Network Full Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which realio-networkd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
realio-networkd tendermint unsafe-reset-all --home $HOME/.realio-network --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable realio-networkd
sudo systemctl restart realio-networkd
````
Checking the logs
````
sudo journalctl -u realio-networkd -f -o cat
````
State Sync:
````
# SRG0Z10 peer: 672c28ea5435aeffe5ae057774f9175a740ab4f2@realio.srgts.xyz:37656
sudo systemctl stop realio-networkd
neutrond tendermint unsafe-reset-all --home $HOME/.neutrond --keep-addr-book
SNAP_RPC="http://realio.srgts.xyz:37657"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.realio-network/config/config.toml

sudo systemctl restart realio-networkd && sudo journalctl -u realio-networkd -f -o cat
````