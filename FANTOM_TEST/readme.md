# FANTOM TESTNET

Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential jq wget git htop curl screen bc -y
````
Install Go:
````
ver="1.20.5"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
````
Install CLI:
````
cd $HOME
git clone https://github.com/Fantom-foundation/go-opera.git
cd ~/go-opera
git checkout release/1.1.3-rc.5
make
mv ~/go-opera/build/opera /usr/local/bin/
````
Let's check the version (current as of October 2023 - v1.1.3-rc.5 commit: ace95a8a1d9a957116874a904fa725ac389abc1a):
````
opera version
````
Download a genesis file:
````
mkdir ~/.opera && cd ~/.opera
wget https://download.fantom.network/testnet-6226-no-mpt.g
cd $HOME
````
Create a service file:
````
sudo tee /etc/systemd/system/opera.service > /dev/null << EOF
[Unit]
Description=FANTOM TESTNET
After=network-online.target
[Service]
User=$USER
ExecStart=/usr/local/bin/opera --genesis /root/.opera/testnet-6226-no-mpt.g --syncmode snap --datadir /root/.opera/datadir --http --http.addr 0.0.0.0 --http.vhosts=* --http.corsdomain=* --ws --ws.addr 0.0.0.0 --ws.origins=* --datadir.minfreedisk=8096 --http.api="ftm,eth,abft,dag,rpc,web3,net,debug" --ws.api="ftm,eth,abft,dag,rpc,web3,net,debug" --cache 32152 --db.preset ldb-1
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
sudo systemctl enable opera
sudo systemctl restart opera
````
Checking the logs
````
sudo journalctl -u opera -f -o cat
````
