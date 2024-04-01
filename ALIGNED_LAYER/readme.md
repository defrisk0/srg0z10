# ALIGNED LAYER

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
cd $HOME
wget https://github.com/yetanotherco/aligned_layer_tendermint/releases/download/v0.1.0/alignedlayerd
chmod +x alignedlayerd
mv alignedlayerd /root/go/bin/
````
Let's check the version (current as of March 2024 - v0.1.0 commit: 98643167990f8a597b331ddd879e079bafb25b08):
````
alignedlayerd version --long
````
Set the correct chain (alignedlayer), chooses his moniker and initialize node:
````
cd $HOME
MNK=ALR_NODE
alignedlayerd init $MNK --chain-id alignedlayer
````
Download the current genesis file:
````
curl http://91.107.239.79:26657/genesis | jq '.result.genesis' > ~/.alignedlayer/config/genesis.json
````
Let's check sum genesis file (current as of March 2024 - 01e24e0c999d69d1502e078ff78a7731205621a85d911ab5fa20f5126e6aa674):
````
sha256sum $HOME/.alignedlayer/config/genesis.json
````
Edit minimum-gas-prices parameter:
````
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001stake"|g' $HOME/.alignedlayer/config/app.toml
````
Adding seeds and peers:
````
seeds=""
peers="0768cb25654e0c799bd56d8b6eac2ea79f3bb937@37.27.71.199:11656,65ad483444b29d413605404cf33c47b2a8908640@137.184.39.243:26656,c355b86c882d05a83f84afba379291d7b954b28f@65.108.236.43:21256,32fbefec592ac2ff9ecb3cad69bafaaad01e771a@148.251.235.130:20656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.alignedlayer/config/config.toml
````
Edit pruning parameter:
````
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.alignedlayer/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.alignedlayer/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.alignedlayer/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.alignedlayer/config/app.toml
````
Create a service file:
````
sudo tee /etc/systemd/system/alignedlayerd.service > /dev/null << EOF
[Unit]
Description=ALIGNED_LAYER NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which alignedlayerd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Reset this node's validator to genesis state:
````
alignedlayerd tendermint unsafe-reset-all --home $HOME/.alignedlayer --keep-addr-book
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable alignedlayerd
sudo systemctl restart alignedlayerd
````
Checking the logs
````
sudo journalctl -u alignedlayerd -f -o cat
````
