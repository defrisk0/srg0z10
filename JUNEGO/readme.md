# JUNEO

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
ExecStart=$(which juneogo) --config-file="$HOME/.juneogo/config.json" --data-dir="$HOME/.juneogo/" --http-host="0.0.0.0"
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
Verify that the node is operational ({"jsonrpc":"2.0","result":{"isBootstrapped":true},"id":1}):
````
curl -X POST --data '{"jsonrpc":"2.0", "id":1,"method" :"info.isBootstrapped","params": {"chain":"JUNE"}}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
````
Get the value of nodeID ({"jsonrpc":"2.0","result":{"nodeID":"NodeID-9xnL72yF86mb4obDvu99ENA4GfzcvEhMy","nodePOP":{"publicKey":"0xad...45"}},"id":1} where NodeID-9...EhMy is your nodeID):
````
curl -X POST --data '{"jsonrpc":"2.0","id":1,"method" :"info.getNodeID"}' -H 'content-type:application/json' 127.0.0.1:9650/ext/info
````
Get a description of peer-to-peer connections:
````
curl -X POST --data '{"jsonrpc":"2.0", "id":1,"method" :"info.peers","params": {"chain":"JUNE"}}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
````
Check the validator uptime:
````
curl -X POST --data '{"jsonrpc":"2.0", "id":1,"method" :"info.uptime","params": {"chain":"JUNE"}}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
````
# JUNEO Super Network

Install Node.JS and add-on packages:
````
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
nvm ls-remote
nvm install 20.10.0
npm install -g typescript
npm install -g ts-node npm i juneojs
````
Let's check out the versions (nvm - 0.39.1, nodejs - 20.10.0):
````
nvm -v
node --version
````
Upload the repository "juneojs-examples" and going to install the packages:
````
git clone https://github.com/Juneo-io/juneojs-examples
cd juneojs-examples
npm install
````
Let's edit the file and add a menonic phrase:
````
cp .env.example .env
nano .env
````
Let's execute commands to transfer tokens:
````
npx ts-node ./src/docs/crossJUNEtoJVM.ts
npx ts-node ./src/docs/crossJVMtoP.ts
````
Create a Supernet and find out our supernetID and record the result:
````
npx ts-node ./src/supernet/createSupernet.ts
````
Edit the file addSupernetValidator.ts and enter our data (line 26 - nodeId, line 27 - durationInDays, line 30 - supernetId):
````
nano -l $HOME/juneojs-examples/src/supernet/addSupernetValidator.ts
````
Let's execute the command to add a node as a validator and record the result:
````
npx ts-node ./src/supernet/addSupernetValidator.ts
````
Let's cut the file and add your supernetID to it:
````
nano $HOME/.juneogo/config.json
{
 "track-supernets":"Zx...hP"
}
````
Restart the node:
````
sudo systemctl restart juneogo
````
