#### Useful Commands
```Shell
# check the blocks
cerberusd status 2>&1 | jq ."SyncInfo"."latest_block_height"

# check the logs
journalctl -u cerberusd -f -o cat

# check status
curl localhost:26657/status

# check the balance
cerberusd q bank balances <address>

# check the validator's pubkey
cerberusd tendermint show-validator
```
#### For validator
```Shell
# collect revards from all validators who were delegated (no commission)
cerberusd tx distribution withdraw-all-rewards --from $WALLET --fees 5000ucrbrus -y

# collect the revards from a separate validator or revards + commission from your validator
cerberusd tx distribution withdraw-rewards <valoper_address> --from $WALLET --fees 5000ucrbrus --commission -y

# to delegate more to the steak (this is how 1 coin is sent)
cerberusd tx staking delegate <valoper_address> 1000000ucrbrus --from $WALLET --fees 5000ucrbrus -y

# redeleting to another validator
cerberusd tx staking redelegate <src-validator-addr> <dst-validator-addr> 1000000ucrbrus --from $WALLET --fees 5000ucrbrus -y

# unbond 
cerberusd tx staking unbond <addr_valoper> 1000000ucrbrus --from $WALLET --fees 5000ucrbrus -y

# send coins to another address
cerberusd tx bank send $WALLET <address> 1000000ucrbrus --fees 5000ucrbrus -y
```
#### Proposal
```Shell
# list of proposals
cerberusd q gov proposals

# see the voting results
cerberusd q gov proposals --voter <ADDRESS>

# vote for the proposal 
cerberusd tx gov vote 1 yes --from <name_wallet> --fees 555umpwr
```

#### Edit validator 
```Shell
BINARY tx staking edit-validator \
  --chain-id "CHAIN_NAME" \
  --moniker "MONIKER" \
  --identity "IDENTITY" \
  --details "DETAILS" \
  --from "WALLET_NAME"
```
