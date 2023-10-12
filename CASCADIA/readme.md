# CASCADIA

[RPC](http://cascadia.srgts.xyz:21457) | [API](http://cascadia.srgts.xyz:1417) | [gRPC](http://cascadia.srgts.xyz:9491)

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
git clone https://github.com/cascadiafoundation/cascadia
cd ~/cascadia
git checkout v0.1.6
make install
````
Let's check the version (current as of October 2023 - v0.1.6 commit: 06ad7b0222b3d7795c25173231fb8b90ad79cdbf):
````
cascadiad version --long
````
Set the correct chain (cascadia_6102-1), chooses his moniker and initialize node:
````
cd $HOME
MNK=testnet
cascadiad config chain-id cascadia_6102-1
cascadiad init $MNK --chain-id cascadia_6102-1
````
Download the current genesis file:
````
curl -L https://anode.team/Cascadia/test/genesis.json > $HOME/.cascadiad/config/genesis.json
````
Let's check sum genesis file (current as of October 2023 - 74ea3c84182028300d0c101c5cf017a055782c595ed91e4be3638380f0169582):
````
sha256sum $HOME/.cascadiad/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01aCC"|g' $HOME/.cascadiad/config/app.toml
````
Adding seeds and peers:
````
seeds=""
peers="001933f36a6ec7c45b3c4cef073d0372daa5344d@194.163.155.84:49656,5530b2b41f983580a8e0ea83feb0e8eb732ef2d8@65.109.101.53:17856,21ca2712116138429aed3d72422379397c53fa86@65.109.65.248:34656,e25bf22448e62faca2359985303ec4557f662444@95.217.11.20:26656,b71287a85b70df75e1405c6831634738e6b957ab@65.108.72.253:15656,3afe6df94dc385efa85aef823e038c76147e4c99@95.217.35.111:26656,1d61222b7b8e180aacebfd57fbd2d8ab95ebdc4c@65.109.93.152:35656,c9256e4f42a23bbdc9ea79805f497a1923a4beac@65.108.230.113:17096,e85f72848ba9586c6704445d1118fb35e2ca5804@65.109.84.33:38656,df3cd1c84b2caa56f044ac19cf0267a44f2e87da@57.128.108.220:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.cascadiad/config/config.toml
````
Add addrbook:
````
curl -s https://raw.githubusercontent.com/defrisk0/srg0z10/main/CASCADIA/addrbook.json > $HOME/.cascadiad/config/addrbook.json
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.cascadiad/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.cascadiad/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.cascadiad/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.cascadiad/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/cascadiad.service > /dev/null << EOF
[Unit]
Description=CASCADIA NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cascadiad) start --chain-id cascadia_6102-1
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
cascadiad tendermint unsafe-reset-all --home $HOME/.cascadiad --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable cascadiad
sudo systemctl restart cascadiad
````
Checking the logs
````
sudo journalctl -u cascadiad -f -o cat
````
State Sync:
````
sudo systemctl stop cascadiad
cp $HOME/.cascadiad/data/priv_validator_state.json $HOME/.cascadiad/priv_validator_state.json.backup
cascadiad tendermint unsafe-reset-all --home $HOME/.cascadiad --keep-addr-book
SNAP_RPC="http://cascadia.srgts.xyz:21457"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.cascadiad/config/config.toml

mv $HOME/.cascadiad/priv_validator_state.json.backup $HOME/.cascadiad/data/priv_validator_state.json
sudo systemctl restart cascadiad && sudo journalctl -u cascadiad -f -o cat
````
