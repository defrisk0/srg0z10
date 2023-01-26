# SUI TESTNET
Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
apt-get install -y --no-install-recommends tzdata git ca-certificates curl build-essential libssl-dev pkg-config libclang-dev cmake jq
````
Install RUST:
````
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
source $HOME/.cargo/env
````
Install CLI:
````
cd $HOME
git clone https://github.com/MystenLabs/sui.git
cd sui
git remote add upstream https://github.com/MystenLabs/sui
git fetch upstream
git checkout -B testnet --track upstream/testnet
cargo build -p sui-node -p sui --release
mv ~/sui/target/release/sui-node /usr/local/bin/
mv ~/sui/target/release/sui /usr/local/bin/
````
Let's check the version (current as of January 2023: testnet a363182b0):
````
git branch -v
````
Download the current genesis.blob file and configure the node:
````
cp ~/sui/crates/sui-config/data/fullnode-template.yaml /var/sui/fullnode.yaml
wget -O /var/sui/genesis.blob https://github.com/MystenLabs/sui-genesis/raw/main/testnet/genesis.blob
sed -i.bak "s/db-path:.*/db-path: \"\/var\/sui\/suidb\"/ ; s/genesis-file-location:.*/genesis-file-location: \"\/var\/sui\/genesis.blob\"/" /var/sui/fullnode.yaml
````
Create a service file:
````
sudo tee /etc/systemd/system/suid.service > /dev/null << EOF
[Unit]
Description=SUI NODE
After=network-online.target
[Service]
User=$USER
ExecStart=/usr/local/bin/sui-node --config-path /var/sui/fullnode.yaml
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````
Starting the node:
````
sudo systemctl daemon-reload
sudo systemctl enable suid
sudo systemctl restart suid
````
Checking the logs
````
sudo journalctl -u suid -f -o cat
````
Let's check release (current as of January 2023 - v0.22.0):
````
curl -s -X POST http://127.0.0.1:9000 -H 'Content-Type: application/json' -d '{ "jsonrpc":"2.0", "method":"rpc.discover","id":1}' | jq .result.info
````
Let's сheck the height of the node:
````
curl -s -X POST http://127.0.0.1:9000 -H 'Content-Type: application/json' -d '{ "jsonrpc":"2.0", "method":"sui_getTotalTransactionNumber","id":1}' | jq -r .result
````
Let's сheck the height of https://fullnode.testnet.sui.io (by comparison):
````
curl -s -X POST https://fullnode.testnet.sui.io:443 -H 'Content-Type: application/json' -d '{ "jsonrpc":"2.0", "method":"sui_getTotalTransactionNumber","id":1}' | jq -r .result
````
