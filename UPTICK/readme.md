# UPTICK

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
git clone https://github.com/UptickNetwork/uptick.git
cd uptick
git checkout v0.2.11
make install
````
Let's check the version (current as of November 2023 - v0.2.11 commit: a67f8c973f0b4068493b5063a3f99fa8816558cf):
````
uptickd version --long
````
Set the correct chain (uptick_117-1), chooses his moniker and initialize node:
````
cd $HOME
MNK=main
uptickd config chain-id uptick_117-1
uptickd init $MNK --chain-id uptick_117-1
````
Download the current genesis file:
````
curl -s https://raw.githubusercontent.com/UptickNetwork/uptick-mainnet/master/uptick_117-1/genesis.json > $HOME/.uptickd/config/genesis.json
````
Let's check sum genesis file (current as of November 2023 - df80462fed795d877fb1e372175ec66d004056fa0ec98c6c3ed52a6715efc66f):
````
sha256sum $HOME/.uptickd/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001auptick"|g' $HOME/.uptickd/config/app.toml
````
Adding seeds and peers:
````
seeds=""
peers="801a902be000e23d437a50f9d86379c82b6342c5@18.119.187.129:26656,5bdf35176e5eda32ec718bd62b039786292a7f7c@65.109.159.69:28656,14ca9d73314dd519bc0b0be8511c88f85fe6873e@46.4.81.204:17656,ef9af846dcb2d25e7ccf5f7975a6d5d51fa01477@5.9.138.213:26656,f05733da50967e3955e11665b1901d36291dfaee@65.108.195.30:21656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.uptickd/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.uptickd/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.uptickd/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.uptickd/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.uptickd/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/uptickd.service > /dev/null << EOF
[Unit]
Description=UPTICK NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which uptickd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
uptickd tendermint unsafe-reset-all --home $HOME/.uptickd --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable uptickd
sudo systemctl restart uptickd
````
Checking the logs
````
sudo journalctl -u uptickd -f -o cat
````
