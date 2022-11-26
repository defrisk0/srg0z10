# NEUTRON
[RPC](http://neutron.srgts.xyz:43657) | [API](http://neutron.srgts.xyz:4517) | [gRPC](http://neutron.srgts.xyz:8390)

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
git clone https://github.com/neutron-org/neutron.git
cd neutron
git checkout v0.1.0
make install
````
Let's check the version (current as of November 2022 - v0.1.0 commit: a9e8ba5ebb9230bec97a4f2826d75a4e0e6130d9):
````
neutrond version --long
````
Set the correct chain (quark-1), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
neutrond config chain-id quark-1
neutrond init $MNK --chain-id quark-1
````
Download the current genesis file:
````
curl -s https://raw.githubusercontent.com/neutron-org/testnets/main/quark/genesis.json > $HOME/.neutrond/config/genesis.json
````
Let's check sum genesis file (current as of November 2022 - 357c4d33fad26c001d086c0705793768ef32c884a6ba4aa73237ab03dd0cc2b4):
````
sha256sum $HOME/.neutrond/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001untrn"|g' $HOME/.neutrond/config/app.toml
````
Adding seeds and peers:
````
seeds="e2c07e8e6e808fb36cca0fc580e31216772841df@seed-1.quark.ntrn.info:26656,c89b8316f006075ad6ae37349220dd56796b92fa@tenderseed.ccvalidators.com:29001"
peers="fcde59cbba742b86de260730d54daa60467c91a5@23.109.158.180:26656,5bdc67a5d5219aeda3c743e04fdcd72dcb150ba3@65.109.31.114:2480,3e9656706c94ae8b11596e53656c80cf092abe5d@65.21.250.197:46656,9cb73281f6774e42176905e548c134fc45bbe579@162.55.134.54:26656,27b07238cf2ea76acabd5d84d396d447d72aa01b@65.109.54.15:51656,f10c2cb08f82225a7ef2367709e8ac427d61d1b5@57.128.144.247:26656,20b4f9207cdc9d0310399f848f057621f7251846@222.106.187.13:40006,5019864f233cee00f3a6974d9ccaac65caa83807@162.19.31.150:55256,2144ce0e9e08b2a30c132fbde52101b753df788d@194.163.168.99:26656,b37326e3acd60d4e0ea2e3223d00633605fb4f79@nebula.p2p.org:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.neutrond/config/config.toml
````
Edit timeout_commit parameter:
````
sed -i 's|^timeout_commit *=.*|timeout_commit = "2s"|' $HOME/.neutrond/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.neutrond/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.neutrond/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.neutrond/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/neutrond.service > /dev/null << EOF
[Unit]
Description=NEUTRON NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which neutrond) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
neutrond tendermint unsafe-reset-all --home $HOME/.neutrond --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable neutrond
sudo systemctl restart neutrond
````
Checking the logs
````
sudo journalctl -u neutrond -f -o cat
````
State Sync:
````
sudo systemctl stop neutrond
neutrond tendermint unsafe-reset-all --home $HOME/.neutrond --keep-addr-book
SNAP_RPC="http://neutron.srgts.xyz:43657"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.neutrond/config/config.toml

sudo systemctl restart neutrond && sudo journalctl -u neutrond -f -o cat
````
