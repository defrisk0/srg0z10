# ARKEO

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
git clone https://github.com/arkeonetwork/arkeo
cd arkeo
git checkout master
TAG=testnet make install
````
Let's check the version (current as of September 2024 - arkeo commit: 5c547a39c5454f3487adf14fd4613fdc9a4e97a5):
````
arkeod version --long
````
Set the correct chain (arkeo), chooses his moniker and initialize node:
````
cd $HOME
MNK=ARK_NODE
arkeod config chain-id arkeo
arkeod init $MNK --chain-id arkeo
````
Download the current genesis file:
````
curl -s http://seed.innovationtheory.com:26657/genesis | jq '.result.genesis' > $HOME/.arkeo/config/genesis.json
````
Let's check sum genesis file (current as of September 2024 - d458511172d076d3e19f6b03212530efa6f7cd6a80412fe60365324d140fe0c3):
````
sha256sum $HOME/.arkeo/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001uarkeo"|g' $HOME/.arkeo/config/app.toml
````
Adding seeds and peers:
````
seeds="aab68f68841eb072d996cd1b45c2b9c9b612d95b@seed.innovationtheory.com:26656,85341b428cf5993fcc04a324d95d14590ae5172c@seed2.innovationtheory.com:26656"
peers="c27c96c5b54a9f2bea776858e2cff364e410d2a8@71.218.54.128:26656,f7da702c17e45e463adf21e57b1d0d936cbc97a3@peer2.innovationtheory.com:26656,46e6d4751bbc67d3e72e13dacdfb0770227fbfc3@65.108.79.241:46656,fc5464b2ce731c5787be0fd316b6c4b6611886ea@37.252.184.241:26656,57f693ba3fed4dd82d02d4cbcc73712c6da4bd34@65.109.113.228:60756,38ab548031dea2b46889c253b762e51306d29cbb@65.109.92.148:61656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.arkeo/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.arkeo/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.arkeo/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "50"|g' $HOME/.arkeo/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.arkeo/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/arkeod.service > /dev/null << EOF
[Unit]
Description=ARKEO NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which arkeod) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
arkeod tendermint unsafe-reset-all --home $HOME/.arkeo --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable arkeod
sudo systemctl restart arkeod
````
Checking the logs
````
sudo journalctl -u arkeod -f -o cat
````
### <a id="title1">State Sync</a>
````
sudo systemctl stop arkeod
cp $HOME/.arkeo/data/priv_validator_state.json $HOME/.arkeo/priv_validator_state.json.backup
arkeod tendermint unsafe-reset-all --home $HOME/.arkeo --keep-addr-book
SNAP_RPC="http://arkeod.srgts.xyz:27757"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.arkeo/config/config.toml

mv $HOME/.arkeo/priv_validator_state.json.backup $HOME/.arkeo/data/priv_validator_state.json
sudo systemctl restart arkeod && sudo journalctl -u arkeod -f -o cat
````
