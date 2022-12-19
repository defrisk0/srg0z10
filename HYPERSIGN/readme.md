# HYPERSIGN
[RPC](http://hypersign.srgts.xyz:41657) | [API](http://hypersign.srgts.xyz:4117) | [gRPC](http://hypersign.srgts.xyz:4190)

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
git clone https://github.com/hypersign-protocol/hid-node.git
cd hid-node
git checkout v0.1.5
make install
````
Let's check the version (current as of December 2022 - v0.1.5 commit: 7913651):
````
hid-noded version --long
````
Set the correct chain (jagrat), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
hid-noded init $MNK --chain-id jagrat
````
Download the current genesis file:
````
curl -s https://raw.githubusercontent.com/hypersign-protocol/networks/master/testnet/jagrat/final_genesis.json > $HOME/.hid-node/config/genesis.json
````
Let's check sum genesis file (current as of December 2022 - 7de2e77cff6d601387a46a760e9c0d7a573b2cfdbdaebb0f04512878543fc0a1):
````
sha256sum $HOME/.hid-node/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.02uhid"|g' $HOME/.hid-node/config/app.toml
````
Adding seeds and peers:
````
seeds=""
peers="c1a08dc3dc9ba7e84ab156854faf0209a0c6d705@65.108.7.143:26656,efcb16ec33d8e6233d1068fff679c6fd64bf5802@65.108.225.158:10956,3f658dc173540479ba70f58f27d60fa9de83378a@195.201.165.123:21056,672c72f28ed5b2a409c1edc2be760e38f76ee4f7@207.244.253.244:36656,47555eddb67b1f858cef8df5c2c917b3bf2c8df3@116.202.236.115:11056,bbea7242ddd3bafcfe1f5ac12b4a112d5bf04176@65.108.194.26:56656,cd13283cd646d71fae76aa2e54ac1c43ea478d58@5.161.41.237:26656,97387c1024c0b7c5478899d598937bdaf3871cb1@213.133.102.206:21066,e0c6b7e238af380c8b531ffd3e367fa1051f9c99@142.132.199.27:21056,de1f980cc59bdb2457202768d4b4d964d783789e@167.235.21.165:36656,1de2abae74a4c5fd7d96d9869ef02187f81498f0@134.209.238.66:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.hid-node/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.hid-node/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.hid-node/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.hid-node/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/hid-noded.service > /dev/null << EOF
[Unit]
Description=HYPERSIGN NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which hid-noded) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
hid-noded tendermint unsafe-reset-all --home $HOME/.hid-node --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable hid-noded
sudo systemctl restart hid-noded
````
Checking the logs
````
sudo journalctl -u hid-noded -f -o cat
````
State Sync:
````
sudo systemctl stop hid-noded
hid-noded tendermint unsafe-reset-all --home $HOME/.hid-node --keep-addr-book
SNAP_RPC="http://hypersign.srgts.xyz:41657"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.hid-node/config/config.toml

sudo systemctl restart hid-noded && sudo journalctl -u hid-noded -f -o cat
````
