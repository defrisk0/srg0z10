# CASCADIA

[RPC](http://cascadia.srgts.xyz:25557) | [API](http://cascadia.srgts.xyz:3517) | [gRPC](http://cascadia.srgts.xyz:9551)

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
git checkout v0.1.2
make install
````
Let's check the version (current as of April 2023 - v0.1.1 commit: 5f5bb104a13407a7312b9c5b780cbfd16d4bd6ef):
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
Let's check sum genesis file (current as of April 2023 - 74ea3c84182028300d0c101c5cf017a055782c595ed91e4be3638380f0169582):
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
peers="275dc82c6fbea56c9c0b5098d5156a939a8698a0@178.63.102.172:25556,1d61222b7b8e180aacebfd57fbd2d8ab95ebdc4c@65.109.93.152:35656,"
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
ExecStart=$(which cascadiad) start
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
SNAP_RPC="http://cascadia.srgts.xyz:25557"

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
