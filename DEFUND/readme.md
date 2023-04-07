# DEFUND
[RPC](http://defund.srgts.xyz:23457) | [API](http://defund.srgts.xyz:3417) | [gRPC](http://defund.srgts.xyz:9340)

et's update and install the necessary packages:
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
git clone https://github.com/defund-labs/defund.git
cd defund
git checkout v0.2.6
make install
````
Let's check the version (current as of March 2023 - v0.2.6 commit: 8f2ebe3d30efe84e013ec5fcdf21a3b99e786c3d):
````
defundd version --long
````
Set the correct chain (orbit-alpha-1), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
defundd config chain-id orbit-alpha-1
defundd init $MNK --chain-id orbit-alpha-1
````
Download the current genesis file:
````
curl -L https://raw.githubusercontent.com/defund-labs/testnet/main/orbit-alpha-1/genesis.json > $HOME/.defund/config/genesis.json
````
Let's check sum genesis file (current as of March 2023 - 58916f9c7c4c4b381f55b6274bce9b8b8d482bfb15362099814ff7d0c1496658):
````
sha256sum $HOME/.defund/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001ufetf"|g' $HOME/.defund/config/app.toml
````
Adding seeds and peers:
````
seeds=""
peers="f114c02efc5aa7ee3ee6733d806a1fae2fbfb66b@5.9.147.22:25656,31b49e981e804cac50a092468e746e496740153e@65.109.84.254:26656,8b80bc13d578d4e80fd672c247491f917c26a71d@84.201.162.168:26656,2b76e96658f5e5a5130bc96d63f016073579b72d@rpc-1.defund.nodes.guru:45656,5b3dd55dade5bfa260d582a11af18ecb18e455b4@defund.srgts.xyz:23457"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.defund/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.defund/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.defund/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.defund/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.defund/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/defund.service > /dev/null << EOF
[Unit]
Description=DEFUND NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which defundd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
defundd tendermint unsafe-reset-all --home $HOME/.defund --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable defundd
sudo systemctl restart defundd
````
Checking the logs
````
sudo journalctl -u defundd -f -o cat
````
State Sync:
````
sudo systemctl stop defundd
cp $HOME/.defund/data/priv_validator_state.json $HOME/.defund/priv_validator_state.json.backup
defundd tendermint unsafe-reset-all --home $HOME/.defund --keep-addr-book
SNAP_RPC="http://defund.srgts.xyz:23457"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.defund/config/config.toml

mv $HOME/.defund/priv_validator_state.json.backup $HOME/.defund/data/priv_validator_state.json
sudo systemctl restart defundd && sudo journalctl -u defundd -f -o cat
````
