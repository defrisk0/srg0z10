# HUMANS
[RPC](http://humans.srgts.xyz:16657) | [API](http://humans.srgts.xyz:1117) | [gRPC](http://humans.srgts.xyz:9190)

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
cd $HOME
wget https://github.com/humansdotai/humans/releases/download/latest/humans_latest_linux_amd64.tar.gz
tar -xvf humans_latest_linux_amd64.tar.gz
sudo mv humansd /usr/local/bin/humansd
rm humans_latest_linux_amd64.tar.gz
````
Let's check the version (current as of December 2022 - latest commit: f66f3bd9af29ce790034e764332e6abacd0d5885):
````
humansd version --long
````
Set the correct chain (altruistic-1), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
humansd config chain-id testnet-1
humansd init $MNK --chain-id testnet-1
````
Download the current genesis file:
````
curl -s https://rpc-testnet.humans.zone/genesis | jq -r .result.genesis > genesis.json
cp genesis.json $HOME/.humans/config/genesis.json
````
Let's check sum genesis file (current as of December 2022 - f5fef1b574a07965c005b3d7ad013b27db197f57146a12c018338d7e58a4b5cd):
````
sha256sum $HOME/.humans/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025uheart"/g' $HOME/.humans/config/app.toml
````
Adding seeds and peers:
````
seeds=""
peers="5c27e54b2b8a597cbbd1c43905d2c18a67637644@142.132.231.118:36656,3d1e89341f64df76599748b634cbabbb8ba3d1b2@65.21.170.3:43656,c7181941789884d6c468bfca31778b10f83a388e@95.217.12.217:26656,981e9829afd1679cd9fafc90edc4ff918057e6fe@217.13.223.167:60556,69822c67487d4907f162fdd6d42549e1df60c82d@65.21.224.248:26656,5e41a64298ca653af5297833c6a47eb1ad1bf367@154.38.161.212:36656,fa57a5bd809eb234f0135e2e62039b5ea09d3992@65.108.250.241:36656,aac683209559ca9ea48de4c47f3806483a5ec13f@185.244.180.97:26656,3fc2c2e3a4b11d540c736a4ae4c9c247fb05fbae@168.119.186.161:26656,c40acba57194521c2d16d59e9dcb2250bb8f2db2@162.55.245.219:36656,295be5393e99c60763c85987fa3f8045af20d828@95.214.53.178:36656,d55876bc04e363bbe68a7fb344dd65632e310f45@138.201.121.185:26668,3f13ad6e8795479b051d147a5049bf4bd0a63817@65.108.142.47:22656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.humans/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.humans/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.humans/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.humans/config/app.toml
````
Edit time parameters:
````
CONFIG_TOML="$HOME/.humans/config/config.toml"
sed -i 's/timeout_propose =.*/timeout_propose = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_propose_delta =.*/timeout_propose_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote =.*/timeout_prevote = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote_delta =.*/timeout_prevote_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit =.*/timeout_precommit = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit_delta =.*/timeout_precommit_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_commit =.*/timeout_commit = "1s"/g' $CONFIG_TOML
sed -i 's/skip_timeout_commit =.*/skip_timeout_commit = false/g' $CONFIG_TOML
````
Create a service file:
````
sudo tee /etc/systemd/system/humansd.service > /dev/null << EOF
[Unit]
Description=HUMANS NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which humansd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
humansd tendermint unsafe-reset-all --home $HOME/.humans --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable humansd
sudo systemctl restart humansd
````
Checking the logs
````
sudo journalctl -u humansd -f -o cat
````
State Sync:
````
sudo systemctl stop humansd
humansd tendermint unsafe-reset-all --home $HOME/.humans --keep-addr-book
SNAP_RPC="http://humans.srgts.xyz:16657"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.humans/config/config.toml

sudo systemctl restart humansd && sudo journalctl -u humansd -f -o cat
````
