# UNION
[RPC](http://union.srgts.xyz:22257) | [STATESYNC](#title1) 

### <a id="title1">State Sync</a>
````
curl -L https://raw.githubusercontent.com/defrisk0/srg0z10/main/UNION/addrbook.json > $HOME/.union/config/addrbook.json
sudo systemctl stop uniond
cp $HOME/.union/data/priv_validator_state.json $HOME/.union/priv_validator_state.json.backup
uniond tendermint unsafe-reset-all --home $HOME/.union --keep-addr-book
SNAP_RPC="http://union.srgts.xyz:22257"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.union/config/config.toml

mv $HOME/.union/priv_validator_state.json.backup $HOME/.union/data/priv_validator_state.json
sudo systemctl restart uniond && sudo journalctl -u uniond -f -o cat
````
