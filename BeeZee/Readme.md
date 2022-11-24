## BeeZee

[Discord](https://discord.gg/VfrfsAbA) | [Website](https://getbze.com/) | [Telegram](https://t.me/BZEdgeOfficial)
--- | --- | ---

Steps | Comments
--- | --- |
[Upgrade](https://github.com/DanilJPG/mainnet_guides/blob/main/BeeZee/Readme.md#:~:text=Upgrade%20and%20install%20dependencies) | Check the version, if necessary update to the appropriate height blocks
[installing utilities](https://github.com/DanilJPG/mainnet_guides/blob/main/BeeZee/Readme.md#:~:text=Upgrade%20and%20install%20dependencies) | server setup
[Installing GO](https://github.com/DanilJPG/mainnet_guides/blob/main/BeeZee/Readme.md#:~:text=liblz4%2Dtool%20%2Dy-,Installing%20Go,-ver%3D%221.19.1%22%0Acd) | Go language is necessary to work with a binary file and unpack it
[Copying a repository](https://github.com/DanilJPG/mainnet_guides/blob/main/BeeZee/Readme.md#:~:text=Cloning%20a%20repository) | Cloning the GitHub repository of a project
[Initializing](https://github.com/DanilJPG/mainnet_guides/blob/main/BeeZee/Readme.md#:~:text=v5.1.1%0Amake%20install-,Initializing,-bzed%20init%20%3Cname_moniker) | To generate configuration files
[Download genesis](https://github.com/DanilJPG/mainnet_guides/blob/main/BeeZee/Readme.md#:~:text=id%20beezee%2D1-,Download%20genesis,-wget%20%2Do%20%24HOME) | The genesis stores the state of the chain
[Fixing the configure](https://github.com/DanilJPG/mainnet_guides/blob/main/BeeZee/Readme.md#:~:text=Correct%20the%20configuration%20file) | Making changes to config.toml
[Prunning app.toml](https://github.com/DanilJPG/mainnet_guides/tree/main/BeeZee#:~:text=config/config.toml-,Prinning,-pruning%3D%22custom%22%20%26%26%20%5C%0Apruning_keep_recent) | To save memory usage
[Service file](https://github.com/DanilJPG/mainnet_guides/blob/main/BeeZee/Readme.md#:~:text=Create%20a%20service%20file) | Creating a service file
[Wallet](https://github.com/DanilJPG/mainnet_guides/blob/main/BeeZee/Readme.md#:~:text=t%20%5C%0A%2D%2Dfees%205000ubze-,Wallet,-%23%20%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D1%82%D1%8C%20%D0%BA%D0%BE%D1%88%D0%B5%D0%BB%D0%B5%D0%BA%0Abzed) | Creating and restoring a wallet
[Validator](https://github.com/DanilJPG/mainnet_guides/blob/main/BeeZee/Readme.md#:~:text=Creating%20a%20validator) | Creating your node operator
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
wget -o $HOME/.bze/config/genesis.json https://raw.githubusercontent.com/bze-alphateam/bze/main/genesis.json
```

#### Correct the configuration file
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001ubze\"/;" ~/.bze/config/app.toml

external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.bze/config/config.toml

peers="6385d5fb198e3a793498019bb8917973325e5eb7@51.15.228.169:26656,ce25088267cef31f3be1ec03263524764c5c80bb@163.172.130.162:26656,2624d40b8861415e004d4532bb7d8d90dd0e6e66@51.15.115.192:26656,f238198a75e886a21cd0522b6b06aa019b9e182e@51.15.55.142:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bze/config/config.toml

seeds="6385d5fb198e3a793498019bb8917973325e5eb7@51.15.138.216:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.bze/config/config.toml
```
#### Prinning
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.bze/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.bze/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.bze/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.bze/config/app.toml
```

#### Create a service file
```
sudo tee /etc/systemd/system/bzed.service > /dev/null <<EOF
[Unit]
Description=bzed
After=network-online.target

[Service]
User=$USER
ExecStart=$(which bzed) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

#### Launch
```
systemctl daemon-reload
systemctl enable bzed
systemctl restart bzed && journalctl -u bzed -f -o cat
```

#### Creating a validator 
```
bzed tx staking create-validator \
--chain-id beezee-1 \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000ubze \
--pubkey $(bzed tendermint show-validator) \
--moniker "<name_moniker>" \
--identity "0393473F5F0C6667" \
--details "Delegate and eat lasagna" \
--from <name_wallet>t \
--fees 5000ubze
```
#### Wallet 
```
# создать кошелек
bzed keys add <name_wallet> --keyring-backend os

# восстановить кошелек (после команды вставить seed)
bzed keys add <name_wallet> --recover --keyring-backend os
```

#### Deleting
```
sudo systemctl stop bzed && \
sudo systemctl disable bzed && \
rm /etc/systemd/system/bzed.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .bze && \
rm -rf bze && \
rm -rf $(which bzed)
```
