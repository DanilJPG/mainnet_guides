#### Useful Commands
```
# check the blocks
lambdavm status 2>&1 | jq ."SyncInfo"."latest_block_height"

# check the logs
journalctl -u lambdavm -f -o cat

# check status
curl localhost:26657/status

# check the balance
lambdavm q bank balances <address>

# check the validator's pubkey
empowerd tendermint show-validator
```

#### Transactions
```
# collect revards from all validators who were delegated (no commission)
lambdavm tx distribution withdraw-all-rewards --from <name_wallet> --fees 5000ulamb -y

# collect the revards from a separate validator or revards + commission from your validator
lambdavm tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 5000ulamb --commission -y

# to delegate more to the steak (this is how 1 coin is sent)
lambdavm tx staking delegate <valoper_address> 1000000ulamb --from <name_wallet> --fees 5000ulamb -y

# redeleting to another validator
lambdavm tx staking redelegate <src-validator-addr> <dst-validator-addr> 1000000ulamb --from <name_wallet> --fees 5000ulamb -y

# unbond 
lambdavm tx staking unbond <addr_valoper> 1000000ulamb --from <name_wallet> --fees 5000ulamb -y

# send coins to another address
lambdavm tx bank send <name_wallet> <address> 1000000ulamb --fees 5000umpwr -y
```

#### Edit validator
```
lambdavm tx staking edit-validator \
  
  --chain-id "CHAIN_NAME" \
  --moniker "MONIKER" \
  --identity "IDENTITY" \
  --details "DETAILS" \
  --from "WALLET_NAME"
  ```
#### Change ports
```
# config.toml
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:6061\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.lambdavm/config/config.toml

# app.toml
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:9190\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:9191\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:1327\"%" $HOME/.lambdavm/config/app.toml

# client.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.lambdavm/config/client.toml

external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:36656\"/" $HOME/.lambdavm/config/config.toml
```
