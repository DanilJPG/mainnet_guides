sudo systemctl stop bzed

cp $HOME/.bze/data/priv_validator_state.json $HOME/.bze/priv_validator_state.json.backup
bzed tendermint unsafe-reset-all --home $HOME/.bze --keep-addr-book

SNAP_RPC="https://rpc.getbze.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 3000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peers="bbe9ec8174f75cd971968c3de03dd543a6d75dfc@95.217.238.76:26656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.bze/config/config.toml

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.bze/config/config.toml

mv $HOME/.bze/priv_validator_state.json.backup $HOME/.bze/data/priv_validator_state.json

sudo systemctl restart bzed
sudo journalctl -u bzed -f --no-hostname -o cat
