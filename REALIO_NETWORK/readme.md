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
git checkout v0.8.0-rc4
make install
````
Let's check the version (current as of March 2023 - v0.8.0-rc4 commit: 692d8ccbd4c229135445d82b51bd2dfd52224651):
````
realio-networkd version --long
````
Set the correct chain (realionetwork_3300-2), chooses his moniker and initialize node.
For a possible node rebuild, generate a BIP39 mnemonic 24 words (e.g. here - https://iancoleman.io/bip39/), enter the mnemonic and be sure to.
````
cd $HOME
MNK=test
realio-networkd config chain-id realionetwork_3300-2
realio-networkd init $MNK --chain-id realionetwork_3300-2
````
Download the current genesis file:
````
curl -s https://raw.githubusercontent.com/realiotech/testnets/main/realionetwork_3300-2/genesis.json > $HOME/.realio-network/config/genesis.json
````
Let's check sum genesis file (current as of March 2023 - 7e3fef8375567d9cbffbbc9e32907c3c87143fd79ef2b6a1f13d232d89057ddc):
````
sha256sum $HOME/.realio-network/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001ario"|g' $HOME/.realio-network/config/app.toml
````
Add addrbook:
````
curl -s https://raw.githubusercontent.com/defrisk0/srg0z10/main/REALIO_NETWORK/addrbook.json > $HOME/.realio-network/config/addrbook.json
````
Adding seeds and peers:
````
seeds=""
peers="4d31b1306f36a7ac657a6ef0ba9fc14c0ac2bd7d@188.143.170.30:26656,51db60ff4c8c49bd84cf7cb93033772433a51be9@5.188.118.105:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.realio-network/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.realio-network/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.realio-network/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.realio-network/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.realio-network/config/app.toml
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
sudo systemctl stop realio-networkd
cp $HOME/.realio-network/data/priv_validator_state.json $HOME/.realio-network/priv_validator_state.json.backup
realio-networkd tendermint unsafe-reset-all --home $HOME/.realio-network --keep-addr-book
SNAP_RPC="http://realio.srgts.xyz:37657"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.realio-network/config/config.toml

mv $HOME/.realio-network/priv_validator_state.json.backup $HOME/.realio-network/data/priv_validator_state.json
sudo systemctl restart realio-networkd && sudo journalctl -u realio-networkd -f -o cat
````
