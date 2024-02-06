# UNION
[RPC](http://union.srgts.xyz:22257) | [STATESYNC](#title1) 

### <a id="title1">State Sync</a>
````
seeds="8a07752a234bb16471dbb577180de7805ba6b5d9@union.testnet.4.seed.poisonphang.com:26656"
peers="51d1865773793be6ccd2db37b5368363044e1876@union.srgts.xyz:21656,45f8c58e121b5414a5a931132d0c08b2a735af0b@union.testnet.4.val.poisonphang.com:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.union/config/config.toml

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
