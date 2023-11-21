# JUNEGO

Let's update and install the necessary packages:
````
sudo apt update && sudo apt upgrade -y
sudo apt install jq git curl -y
````
Install CLI:
````
git clone https://github.com/Juneo-io/juneogo-binaries
chmod +x $HOME/juneogo-binaries/juneogo && cp $HOME/juneogo-binaries/juneogo /root/go/bin/
mkdir $HOME/.juneogo/
chmod +x $HOME/juneogo-binaries/plugins/jevm
chmod +x $HOME/juneogo-binaries/plugins/srEr2XGGtowDVNQ6YgXcdUb16FGknssLTGUFYg7iMqESJ4h8e
cp -r $HOME/juneogo-binaries/plugins $HOME/.juneogo/
cp $HOME/juneogo-binaries/config.json $HOME/.juneogo/
````
Let's check the version (juneo/0.1.0 [database=v1.4.5, rpcchainvm=25]):
````
juneogo --version
````
Create a service file:
````
sudo tee /etc/systemd/system/juneogo.service > /dev/null << EOF
[Unit]
Description=JUNEGO NODE
After=network-online.target
[Service]
User=$USER
ExecStart=$(which juneogo) --config-file="$HOME/.juneogo/config.json" --data-dir="$HOME/.juneogo/"
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
sudo systemctl enable juneogo
sudo systemctl restart juneogo
````
Checking the logs
````
sudo journalctl -u juneogo -f -o cat
````
