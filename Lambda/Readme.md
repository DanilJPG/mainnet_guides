## Lambda Mainnet

[Discord](https://discord.gg/lambdanetwork) | [Website](https://www.lambda.im/) | [Telegram](https://t.me/HelloLambda)
--- | --- | ---

Steps | Comments
--- | --- |
[Server Preparation](https://github.com/DanilJPG/mainnet_guides/blob/main/Lambda/Readme.md#:~:text=%D0%9F%D0%BE%D0%B4%D0%B3%D0%BE%D1%82%D0%BE%D0%B2%D0%BA%D0%B0%20%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%B0%20%2D%20Server%20Preparation) | Check the version, if necessary update to the appropriate height blocks,server setup, Go language is necessary to work with a binary file and unpack it
[Working with a binary file and setting up](https://github.com/DanilJPG/mainnet_guides/blob/main/Lambda/Readme.md#:~:text=.%D0%A0%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%20%D1%81%20%D0%B1%D0%B8%D0%BD%D0%B0%D1%80%D0%BD%D1%8B%D0%BC%20%D1%84%D0%B0%D0%B9%D0%BB%D0%BE%D0%BC%20%D0%B8%20%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0%20%2D%20Working%20with%20a%20binary%20file%20and%20setting%20up) | Cloning the GitHub repository of a project,To generate configuration files,The genesis stores the state of the chain,Making changes to config.toml,Creating a service file
[Creating a validator and generating a wallet](https://github.com/DanilJPG/mainnet_guides/blob/main/Lambda/Readme.md#:~:text=%D0%A1%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%20%D0%B2%D0%B0%D0%BB%D0%B8%D0%B4%D0%B0%D1%82%D0%BE%D1%80%D0%B0%20%D0%B8%20%D0%B3%D0%B5%D0%BD%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D1%8F%20%D0%BA%D0%BE%D1%88%D0%B5%D0%BB%D1%8C%D0%BA%D0%B0%20%2D%20Creating%20a%20validator%20and%20generating%20a%20wallet) | Creating and restoring a wallet,Creating your node operator
[Useful commands](https://github.com/DanilJPG/mainnet_guides/blob/main/Lambda/Other%20commands.md) | Here are commands for the validator, for node management and for the wallet
[State Sunc](https://github.com/DanilJPG/mainnet_guides/blob/main/Lambda/State%20Sync.md) | Chain synchronization

### 1.Подготовка сервера - Server Preparation 
#### Обновление и установка зависимостей - Upgrade and install dependencies
```Shell
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
#### Installing Go
```Bash
ver="1.19.1" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

### 2.Работа с бинарным файлом и настройка - Working with a binary file and setting up
#### Cloning a repository 
```Shell
git clone https://github.com/LambdaIM/lambdavm.git
cd lambdavm
make install
```

#### Initializing 
```Shell
lambdavm init <your_custom_moniker> --chain-id lambda_92000-1 \
lambdavm config chain-id lambda_92000-1
```

#### Download genesis
```Shell
wget -O /root/.lambdavm/config//genesis.json "https://raw.githubusercontent.com/LambdaIM/mainnet/main/lambda_92000-1/genesis.json"
#check

`sha256sum genesis.json`
# 1ff02001539bc1e9828fe170006f055c04df280c61c4ca9ecc9e7b6a272b7777  genesis.json
```

#### Correct the configuration file
```Shell
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" /root/.lambdavm/config/config.toml

sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025ulamb\"/;" /root/.lambdavm/config/app.toml

PEERS=`curl -sL https://raw.githubusercontent.com/LambdaIM/mainnet/main/lambda_92000-1/peers.txt | sort -R | head -n 10 | awk '{print $1}' | paste -s -d, -`

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.lambdavm/config/config.toml

SEEDS=`curl -sL https://raw.githubusercontent.com/LambdaIM/mainnet/main/lambda_92000-1/seeds.txt | awk '{print $1}' | paste -s -d, -`
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" ~/.lambdavm/config/config.toml
```
#### Prunning `app.toml'
```Shell
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
```Shell
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
```Shell
systemctl daemon-reload && \
systemctl enable lambdavm && \
systemctl restart lambdavm && journalctl -u lambdavm -f -o cat
```


### 3.Создание валидатора и генерация кошелька - Creating a validator and generating a wallet
#### Wallet 
```Shell
# create a wallet
lambdavm keys add $WALLET --keyring-backend os

# restore the wallet (after the command insert seed)
lambdavm keys add $WALLET --recover --keyring-backend os

# export to metamask(view private key)
lambdavm keys unsafe-export-eth-key garfield_wallet

# export wallet from metamask
lambdavm keys unsafe-import-eth-key [account name] [private key]
```

#### Creating a validator 
```Shell
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

### 4. Удаление - Delete
#### Deleting
```Shell
systemctl stop lambdavm && \
systemctl disable lambdavm && \
rm /etc/systemd/system/lambdavm.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .lambdavm lambdavm && \
rm -rf $(which lambdavm)
```
