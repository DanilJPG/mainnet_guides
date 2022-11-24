## BeeZee

[Discord](https://discord.gg/VfrfsAbA) | [Website](https://getbze.com/) | [Telegram](https://t.me/BZEdgeOfficial)
--- | --- | ---

Steps | Comments
--- | --- |
[Upgrade]() | Check the version, if necessary update to the appropriate height blocks
[installing utilities]() | server setup
[Installing GO]() | Go language is necessary to work with a binary file and unpack it
[Copying a repository]() | Cloning the GitHub repository of a project
[Initializing]() | To generate configuration files
[Download genesis]() | The genesis stores the state of the chain
[Fixing the configure]() | Making changes to config.toml
[Prunning app.toml]() | To save memory usage
[Service file]() | Creating a service file
[Launch]() | Start node 
[Wallet]() | Creating and restoring a wallet
[Validator]() | Creating your node operator
[Useful commands]() | Here are commands for the validator, for node management and for the wallet
[State Sunc]() | Chain synchronization

#### Upgrade and install dependencies
```
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
#### Installing Go
```
ver="1.19.1"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
```

#### Cloning a repository 
```
git clone https://github.com/bze-alphateam/bze
cd bze
git checkout v5.1.1
make install
```

#### Initializing 
```
bzed init <name_moniker> --chain-id beezee-1
bzed config chain-id beezee-1
```

#### Download genesis
```
wget -O /root/.lambdavm/config//genesis.json "https://raw.githubusercontent.com/LambdaIM/mainnet/main/lambda_92000-1/genesis.json"
#check

`sha256sum genesis.json`
# 1ff02001539bc1e9828fe170006f055c04df280c61c4ca9ecc9e7b6a272b7777  genesis.json
```

#### Correct the configuration file
```
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" /root/.lambdavm/config/config.toml

sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025ulamb\"/;" /root/.lambdavm/config/app.toml

PEERS=`curl -sL https://raw.githubusercontent.com/LambdaIM/mainnet/main/lambda_92000-1/peers.txt | sort -R | head -n 10 | awk '{print $1}' | paste -s -d, -`

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.lambdavm/config/config.toml

SEEDS=`curl -sL https://raw.githubusercontent.com/LambdaIM/mainnet/main/lambda_92000-1/seeds.txt | awk '{print $1}' | paste -s -d, -`
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" ~/.lambdavm/config/config.toml
```
#### Prinning
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.lambdavm/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.lambdavm/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.lambdavm/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.lambdavm/config/app.toml
```

#### Create a service file
```
sudo tee /etc/systemd/system/lambdavm.service > /dev/null <<EOF
[Unit]
Description=lambdavm
After=network-online.target
[Service]
User=$USER
ExecStart=$(which lambdavm) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

#### Launch
```
systemctl daemon-reload && \
systemctl enable lambdavm && \
systemctl restart lambdavm && journalctl -u lambdavm -f -o cat
```

#### Creating a validator 
```
lambdavm tx staking create-validator \
--amount 1000000000000000000ulamb \
--pubkey $(lambdavm tendermint show-validator) \
--moniker "garfield" \
--chain-id lambda_92000-1 \
--commission-rate "0.05" \
--min-self-delegation "1000000" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.01" \
--from garfield_wallet \
--gas=auto \
--fees 5000ulamb
```
#### Wallet 
```
# create a wallet
empowerd keys add $WALLET --keyring-backend os

# restore the wallet (after the command insert seed)
empowerd keys add $WALLET --recover --keyring-backend os

# export to metamask(view private key)
lambdavm keys unsafe-export-eth-key garfield_wallet

# export wallet from metamask
lambdavm keys unsafe-import-eth-key [account name] [private key]
```

#### Deleting
```
systemctl stop lambdavm && \
systemctl disable lambdavm && \
rm /etc/systemd/system/lambdavm.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .lambdavm lambdavm && \
rm -rf $(which lambdavm)
```
