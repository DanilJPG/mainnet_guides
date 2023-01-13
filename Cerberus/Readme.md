## Cerberus


[Discord](https://discord.gg/2KyWFcZF) | [Website](https://docs.cerberus.zone/running-a-validator.html)
--- | --- |

Steps | Comments
--- | --- |
[Server Preparation](https://github.com/DanilJPG/mainnet_guides/blob/main/Cerberus/Readme.md#:~:text=for%20the%20wallet-,1.%D0%9F%D0%BE%D0%B4%D0%B3%D0%BE%D1%82%D0%BE%D0%B2%D0%BA%D0%B0%20%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%B0%20%2D%20Server%20Preparation,-%D0%9E%D0%B1%D0%BD%D0%BE%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5%20%D0%B8%20%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0) | Check the version, if necessary update to the appropriate height blocks,server setup, Go language is necessary to work with a binary file and unpack it
[Working with a binary file and setting up](https://github.com/DanilJPG/mainnet_guides/blob/main/Cerberus/Readme.md#:~:text=2.%D0%A0%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%20%D1%81%20%D0%B1%D0%B8%D0%BD%D0%B0%D1%80%D0%BD%D1%8B%D0%BC%20%D1%84%D0%B0%D0%B9%D0%BB%D0%BE%D0%BC%20%D0%B8%20%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0%20%2D%20Working%20with%20a%20binary%20file%20and%20setting%20up) | Cloning the GitHub repository of a project,To generate configuration files,The genesis stores the state of the chain,Making changes to config.toml,Creating a service file
[Creating a validator and generating a wallet](https://github.com/DanilJPG/mainnet_guides/blob/main/Cerberus/Readme.md#:~:text=f%20%2Do%20cat-,3.%D0%A1%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%20%D0%B2%D0%B0%D0%BB%D0%B8%D0%B4%D0%B0%D1%82%D0%BE%D1%80%D0%B0%20%D0%B8%20%D0%B3%D0%B5%D0%BD%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D1%8F%20%D0%BA%D0%BE%D1%88%D0%B5%D0%BB%D1%8C%D0%BA%D0%B0%20%2D%20Creating%20a%20validator%20and%20generating%20a%20wallet,-Wallet) | Creating and restoring a wallet,Creating your node operator
[Useful commands](https://github.com/DanilJPG/mainnet_guides/blob/main/Cerberus/Useful%20commands.md) | Here are commands for the validator, for node management and for the wallet

***

### 1.Подготовка сервера - Server Preparation 
#### Обновление и установка зависимостей - Upgrade and install dependencies
```Bash
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
#### Installing GO
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
```Bash
git clone https://github.com/cerberus-zone/cerberus.git

cd cerberus

git fetch -a

git checkout v1.0.1

make install && cd ~/go/bin && sudo cp cerberusd /usr/local/bin
```
#### Initializing
```Bash
export MONIKER_NAME="<name_moniker>"
cerberusd init "${MONIKER_NAME}" --chain-id cerberus-chain-1
```
#### Download genesis
```Bash
wget -O $HOME/.cerberus/config/genesis.json "https://raw.githubusercontent.com/cerberus-zone/cerberus_genesis/main/genesis.json"
```
#### Correct the configuration file
```Shell
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" /root/.cerberus/config/config.toml

sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025ucrbrus\"/;" ~/.cerberus/config/app.toml

PEERS=$(curl https://raw.githubusercontent.com/cerberus-zone/cerberus_genesis/main/peers.txt | \
head -n 32 | sed 's/$/,/' | tr -d '\n' | sed '$ s/.$//'); sed "s/persistent_peers = \"\"/persistent_peers = \"$PEERS\"/" \
$HOME/.cerberus/config/config.toml -i

seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.cerberus/config/config.toml
```
#### Prunning `app.toml'
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.cerberus/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.cerberus/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.cerberus/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.cerberus/config/app.toml
```
#### Create a service file
```Bash
sudo tee /etc/systemd/system/cerberusd.service > /dev/null <<EOF
[Unit]
Description=cerberusd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cerberusd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

```
#### Launch
```Bash
systemctl daemon-reload && \
systemctl enable cerberusd && \
systemctl restart cerberusd && journalctl -u cerberusd -f -o cat
```
### 3.Создание валидатора и генерация кошелька - Creating a validator and generating a wallet
#### Wallet 
```Bash
cerberusd keys add <wallet-name> --keyring-backend os

cerberusd keys add <wallet-name> --recover --keyring-backend os
```
Go to discord and use the tap 

#### Validator
```Bash
cerberusd tx staking create-validator \
--chain-id cerberus-chain-1 \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000ucrbrus \
--pubkey $(cerberusd tendermint show-validator) \
--moniker "<name_moniker>" \
--from <name_wallet> \
--fees 5000ucrbrus
```

### 4. Удаление - Delete
#### Deleting
```Shell
systemctl stop cerberusd && \
systemctl disable cerberusd && \
rm /etc/systemd/system/cerberusd.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .cerberus empowerchain && \
rm -rf $(which cerberusd)
```
