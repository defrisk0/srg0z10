# ARTELA

Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential jq wget git htop curl screen bc -y
````
Install Go:
````
cd $HOME
VRS="1.21.5"
wget "https://golang.org/dl/go$VRS.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VRS.linux-amd64.tar.gz"
rm "go$VRS.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
````
Install CLI:
````
git clone https://github.com/artela-network/artela.git
cd artela
git checkout v0.4.7-rc4-5-gcd9dcb9
make install
````
Let's check the version (current as of January 2024 - v0.4.7-rc4-5-gcd9dcb9 commit: cd9dcb9ccd349198ba95dc0c3d8fb7c1d04549e3):
````
artelad version --long
````
Set the correct chain (artela_11822-1), chooses his moniker and initialize node:
````
cd $HOME
MNK=ART_KEY
artelad config chain-id artela_11822-1
artelad init $MNK --chain-id artela_11822-1
````
Download the current genesis file:
````
curl -L https://raw.githubusercontent.com/defrisk0/srg0z10/main/ARTELA/genesis.json > $HOME/.artelad/config/genesis.json
````
Let's check sum genesis file (current as of January 2024 - 9cc7d80f29b42a2b6658ee867608dd75cead81cac22e4ed9f830de3554b10438):
````
sha256sum $HOME/.artelad/config/genesis.json
````
Download the current addrbook:
````
curl -L https://raw.githubusercontent.com/defrisk0/srg0z10/main/ARTELA/addrbook.json > $HOME/.artelad/config/addrbook.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025art"|g' $HOME/.artelad/config/app.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.artelad/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.artelad/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.artelad/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.artelad/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/andromedad.service > /dev/null << EOF
[Unit]
Description=ARTELA NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which artelad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
artelad tendermint unsafe-reset-all --home $HOME/.artelad --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable artelad
sudo systemctl restart artelad
````
Checking the logs
````
sudo journalctl -u artelad -f -o cat
````
### <a id="title1">State Sync</a>
````
sudo systemctl stop artelad
cp $HOME/.artelad/data/priv_validator_state.json $HOME/.artelad/priv_validator_state.json.backup
artelad tendermint unsafe-reset-all --home $HOME/.artelad --keep-addr-book
SNAP_RPC=http://95.216.35.51:22357

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.artelad/config/config.toml

mv $HOME/.artelad/priv_validator_state.json.backup $HOME/.artelad/data/priv_validator_state.json
sudo systemctl restart artelad && sudo journalctl -u artelad -f -o cat
````
