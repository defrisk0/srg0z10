# SGE 
[RPC](http://sge.srgts.xyz:16657) | [API](http://sge.srgts.xyz:1417) | [gRPC](http://sge.srgts.xyz:9490)

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
git clone https://github.com/sge-network/sge
cd sge
git fetch --tags
git checkout v0.0.3
make install
````
Let's check the version (current as of January 2023 - v0.0.3 commit: c2f074f15fa895b0d8e67a9d88bfd2b9d9833b2f):
````
sged version --long
````
Set the correct chain (jagrat), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
sged config chain-id sge-testnet-1
sged init $MNK --chain-id sge-testnet-1
````
Download the current genesis file:
````
curl -s https://raw.githubusercontent.com/sge-network/networks/master/sge-testnet-1/genesis.json > $HOME/.sge/config/genesis.json
````
Let's check sum genesis file (current as of January 2023 - 9a36a608e66fb194f404284c2d65aa5a7eb422b3efbd50869f270dfe65713e5b):
````
sha256sum $HOME/.sge/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01usge"|g' $HOME/.sge/config/app.toml
````
Adding seeds and peers:
````
seeds=""
peers="4a3f59e30cde63d00aed8c3d15bef46b34ec2c7f@50.19.180.153:26656,27f0b281ea7f4c3db01fdb9f4cf7cc910ad240a6@209.34.206.44:26656,8a7d722dba88326ee69fcc23b5b2ac93e36d7ff2@65.108.225.158:17756,12450c4223a2d6dcfbe5e9b9998cb67634cd2465@38.146.3.193:26656,413128504de36317e3bf000073aa3165351e0d52@44.197.179.40:26656,a05353fe9ae39dd0edbfa6341634dec781d84a5c@65.108.105.48:17756,5c240add1ea545da7082616d4fb7276371f0bf66@94.250.201.130:26656,43b05a6bab7ca735397e9fae2cb0ad99977cf482@34.82.157.5:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.sge/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.sge/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.sge/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.sge/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.sge/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/sged.service > /dev/null << EOF
[Unit]
Description=SGE NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which sged) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
sged tendermint unsafe-reset-all --home $HOME/.sge --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable sged
sudo systemctl restart sged
````
Checking the logs
````
sudo journalctl -u sged -f -o cat
````
State Sync:
````
sudo systemctl stop sged
cp $HOME/.sge/data/priv_validator_state.json $HOME/.sge/priv_validator_state.json.backup
sged tendermint unsafe-reset-all --home $HOME/.sge --keep-addr-book
SNAP_RPC="http://sge.srgts.xyz:16657"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.sge/config/config.toml

mv $HOME/.sge/priv_validator_state.json.backup $HOME/.sge/data/priv_validator_state.json
sudo systemctl restart sged && sudo journalctl -u sged -f -o cat
````
