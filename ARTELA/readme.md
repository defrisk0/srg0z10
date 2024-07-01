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
git checkout v0.4.7-rc6-12-g2dfe1c2
make install
````
Let's check the version (current as of July 2024 - v0.4.7-rc6 commit: 2dfe1c23808c8b91a97d07d7c6de0bdb32de938c):
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
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.02uart"|g' $HOME/.artelad/config/app.toml
````
Edit runtime cache size:
````
sed -E 's/^pool-size[[:space:]]*=[[:space:]]*[0-9]+$/apply-pool-size = 10\nquery-pool-size = 30/' $HOME/.artelad/config/app.toml > $HOME/.artelad/config/temp.app.toml && mv $HOME/.artelad/config/temp.app.toml $HOME/.artelad/config/app.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.artelad/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "362880"|g' $HOME/.artelad/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "100"|g' $HOME/.artelad/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 1000|g' $HOME/.artelad/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/artelad.service > /dev/null << EOF
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
