# FIRMACHAIN
[RPC](http://firmachain.srgts.xyz:26657) | [API](http://firmachain.srgts.xyz:1317) | [STATESYNC](#title1) 

#### Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential jq wget git curl -y
````
#### Install Go:
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
#### Install CLI:
````
cd $HOME
git clone https://github.com/firmachain/firmachain
cd firmachain
git checkout v0.4.0
make install
````
#### Let's check the version (current as of December 2024 - v0.3.5-patch commit: e2ab40a4074a05838f94babab566fe231f4c39b3):
````
firmachaind version --long
````
#### Set the correct chain (gitopia), chooses his moniker and initialize node:
````
cd $HOME
MNK=FRCH_NODE
firmachaind config chain-id colosseum-1
firmachaind init $MNK --chain-id colosseum-1
````
#### Download the current genesis file:
````
wget https://raw.githubusercontent.com/FirmaChain/mainnet/main/colosseum-1/genesis.json -O ~/.firmachain/config/genesis.json
````
#### Let's check sum genesis file (current as of December 2024 - d03edc3362a677f4a0c2f605c7f848f9516fd4f55432fda63625398c52419954):
````
sha256sum $HOME/.firmachain/config/genesis.json
````
#### Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01ufct"|g' $HOME/.firmachain/config/app.toml
````
#### Adding seeds and peers:
````
seeds="f89dcc15241e30323ae6f491011779d53f9a5487@mainnet-seed1.firmachain.dev:26656,04cce0da4cf5ceb5ffc04d158faddfc5dc419154@mainnet-seed2.firmachain.dev:26656,940977bdc070422b3a62e4985f2fe79b7ee737f7@mainnet-seed3.firmachain.dev:26656"
peers="8458136b16445520ddbd6d12aa05f37fbe9302dc@195.201.110.169:26656,c5d73a7ebd292123068125356c615cc8db4c607a@65.109.154.185:28656,e5a99c4de697ad16ac74620ad3faf9ffabc728d8@136.243.55.237:43576,697e738961cfb28e67c7655bda49282dcf6b153a@140.82.53.152:26656,d5ac7fb49b76373cc2722ff802b1d245dcf85d75@178.250.154.15:26656,66b5ce316c5ba1dce402d48b22abb94331184795@65.109.231.247:26656,061bc813fe8aa0f6e90ac31e78e7b9e08ff585aa@65.108.70.119:35656,2a1f831aafb2225c797355163ba9a1d1090a2da2@164.92.231.224:26656,102ce7393900fe56d3bf06f7370939ca2b8b6fe3@207.244.245.6:56656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.firmachain/config/config.toml
````
#### Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.firmachain/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.firmachain/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.firmachain/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.firmachain/config/app.toml
````
#### Create a service file:
````
sudo tee /etc/systemd/system/firmachaind.service > /dev/null << EOF
[Unit]
Description=FIRMACHAIN NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which firmachaind) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
#### Reset this node's validator to genesis state:
````
firmachaind tendermint unsafe-reset-all --home $HOME/.firmachain --keep-addr-book
````
#### Download Snapshot file
````
wget https://firmachain-mainnet-snapshot.s3.ap-northeast-2.amazonaws.com/colosseum-1_2023-01-13_5150845.tar -O ~/.firmachain/data/firmachain.tar
tar -xvf ~/.firmachain/data/firmachain.tar
````
#### Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable firmachaind
sudo systemctl restart firmachaind
````
#### Checking the logs
````
sudo journalctl -u firmachaind -f -o cat
````
