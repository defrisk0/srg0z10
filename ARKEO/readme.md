# ARKEO
[RPC](http://arkeo.srgts.xyz:27757) | [API](http://arkeo.srgts.xyz:1177) | [STATESYNC](#title1) 

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
git checkout ab05b124336ace257baa2cac07f7d1bfeed9ac02
make install
````
Let's check the version (current as of November 2023 - arkeo commit: 68c59e9057e306dd99cdf55ebf4e6b1876835dc8):
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
curl -s http://seed.arkeo.network:26657/genesis | jq '.result.genesis' > ~/.arkeo/config/genesis.json
````
Let's check sum genesis file (current as of November 2023 - 214828d2dac5eaaa4d2e70dde63bd460dcc86ab9e5dd7868dbfa8c3186b6abf9):
````
sha256sum $HOME/.arkeo/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001uarkeo"|g' $HOME/.arkeo/config/app.toml
````
Adding seeds and peers:
````
seeds="20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:22856,df0561c0418f7ae31970a2cc5adaf0e81ea5923f@arkeo-testnet-seed.itrocket.net:18656"
peers="374facfe63ab4c786d484c2d7d614063190590b7@88.99.213.25:38656,8c2d799bcc4fbf44ef34bbd2631db5c3f4619e41@213.239.207.175:60656,939ab74d3f49428b9c5c6150929037680051e14e@65.109.30.110:22856,b3a8a2660ca5520e385df6d4f42fac8c6fab1fd0@144.126.142.78:34656,98911188da7520af2165b3562b1b28fdc55ed5e7@65.108.91.152:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.arkeo/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.arkeo/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.arkeo/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.arkeo/config/app.toml
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
