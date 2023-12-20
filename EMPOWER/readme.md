# EMPOWER

Let's update and install the necessary packages:
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
git clone https://github.com/EmpowerPlastic/empowerchain
cd empowerchain/chain
git checkout v2.0.0
make install
````
Let's check the version (current as of December 2023 - v2.0.0 commit: 70ad47fc878d1854fe279ebf99e3a9260b78099c):
````
empowerd version --long
````
Set the correct chain (empowerchain-1), chooses his moniker and initialize node:
````
cd $HOME
MNK=test
empowerd config chain-id empowerchain-1
empowerd init $MNK --chain-id empowerchain-1
````
Download the current genesis file:
````
curl -L https://github.com/EmpowerPlastic/empowerchain/raw/main/mainnet/empowerchain-1/genesis.tar.gz | tar -xz -C $HOME/.empowerchain/config/
````
Let's check sum genesis file (current as of December 2023 - 819d33d14c35bbfbc5997db9bf545eb7a5504b5870a307ce90c3813add4b316b):
````
sha256sum $HOME/.empowerchain/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001umpwr"|g' $HOME/.empowerchain/config/app.toml
````
Adding seeds and peers:
````
seeds="f2ed98cf518b501b6d1c10c4a16d0dfbc4a9cc98@tenderseed.ccvalidators.com:27001,e16668ddd526f4e114ebb6c4714f0c18c0add8f8@empower-seed.zenscape.one:26656,6740fa259552a628266a85de8c2a3dee7702b8f9@empower-mainnet-seed.itrocket.net:14656"
peers=""
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.empowerchain/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.empowerchain/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.empowerchain/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.empowerchain/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.empowerchain/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/empowerd.service > /dev/null << EOF
[Unit]
Description=EMPOWER NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which empowerd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
empowerd tendermint unsafe-reset-all --home $HOME/.empowerchain --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable empowerd
sudo systemctl restart empowerd
````
Checking the logs
````
sudo journalctl -u empowerd -f -o cat
````
