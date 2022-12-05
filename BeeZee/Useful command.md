#### Useful Commands
```Bash
# check the blocks
bzed status 2>&1 | jq ."SyncInfo"."latest_block_height"

# check the logs
journalctl -u bzed -f -o cat

# check status
curl localhost:26657/status

# check the balance
bzed q bank balances <address>

# check the validator's pubkey
bzed tendermint show-validator
```
####  Полезные команды для валидатора и делегатора
```Bash
# collect revards from all validators who were delegated (no commission)
bzed tx distribution withdraw-all-rewards --from <name_wallet> --fees 5000ubze -y

# collect the revards from a separate validator or revards + commission from your validator
bzed tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 5000ubze --commission -y

# to delegate more to the steak (this is how 1 coin is sent)
bzed tx staking delegate <valoper_address> 1000000ubze --from <name_wallet> --fees 5000ubze -y

# redeleting to another validator
bzed tx staking redelegate <src-validator-addr> <dst-validator-addr> 1000000ubze --from <name_wallet> --fees 5000ubze -y

# unbond 
bzed tx staking unbond <addr_valoper> 1000000ubze --from <name_wallet> --fees 5000ubze -y

# send coins to another address
bzed tx bank send <name_wallet> <address> 1000000ubze --fees 5000ubze -y
```
#### Proposal
```Bash
# list of proposals
bzed q gov proposals

# see the voting results
bzed q gov proposals --voter <ADDRESS>

# vote for the proposal 
bzed tx gov vote 1 yes --from <name_wallet> --fees 555ubze
```

#### Delete
```Bash
systemctl stop bzed && \
systemctl disable bzed && \
rm /etc/systemd/system/bzed.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .bze bzed && \
rm -rf $(which bzed)
```

#### Edit validator 
```Bash
BINARY tx staking edit-validator \
  --chain-id "CHAIN_NAME" \
  --moniker "MONIKER" \
  --identity "IDENTITY" \
  --details "DETAILS" \
  --from "WALLET_NAME"
```
