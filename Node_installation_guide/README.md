<p style="font-size:14px" align="right">
<a href="https://t.me/L0vd_staking" target="_blank">Join our telegram <img src="https://raw.githubusercontent.com/L0vd/screenshots/main/Telegram_logo.png" width="30"/></a>
<a href="https://l0vd.com/" target="_blank">Visit our website <img src="https://raw.githubusercontent.com/L0vd/screenshots/main/L0vd.png" width="30"/></a>
</p>



# Table of contents <br />
[Node setup](#node_setup) <br />
[State Sync](#state_sync) <br />
[Starting a validator](#starting_validator) <br />
[Useful commands](#useful_commands)



<a name="node_setup"></a>
# Manual node setup
If you want to setup Andromeda fullnode manually follow the steps below

## Update and upgrade
```
sudo apt update && sudo apt upgrade -y
```

## Install GO
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.19"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Install node
```
cd $HOME
git clone https://github.com/andromedaprotocol/andromedad.git
cd andromedad
git checkout galileo-3-v1.1.0-beta1 
make install
andromedad version #galileo-3-v1.1.0-beta1
```


## Setting up vars
You should replace values in <> <br />
<YOUR_MONIKER> Here you should put name of your moniker (validator) that will be visible in explorer <br />
<YOUR_WALLET> Here you shoud put the name of your wallet

```
echo "export ANDROMEDA_WALLET="<YOUR_WALLET_NAME>"" >> $HOME/.bash_profile
echo "export ANDROMEDA_NODENAME="<YOUR_MONIKER>"" >> $HOME/.bash_profile
echo "export ANDROMEDA_CHAIN_ID="galileo-3"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```


## Configure your node
```
andromedad config chain-id $ANDROMEDA_CHAIN_ID
```

## Initialize your node
```
andromedad init $NODENAME --chain-id $ANDROMEDA_CHAIN_ID
```

## Download genesis
```
wget "$HOME/.andromedad/config/genesis.json" https://raw.githubusercontent.com/andromedaprotocol/testnets/galileo-3/genesis.json
```

## (OPTIONAL) Set custom ports

### If you want to use non-default ports
```
ANDROMEDA_PORT=<SET_CUSTOM_PORT> #Example: ANDROMEDA_PORT=56 (numbers from 1 to 64)
```
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${ANDROMEDA_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${ANDROMEDA_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${ANDROMEDA_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${ANDROMEDA_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${ANDROMEDA_PORT}660\"%" $HOME/.andromedad/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${ANDROMEDA_PORT}317\"%; s%^address = \":8080\"%address = \":${ANDROMEDA_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${ANDROMEDA_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${ANDROMEDA_PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${ANDROMEDA_PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${ANDROMEDA_PORT}546\"%" $HOME/.andromedad/config/app.toml
```


## Set seeds and peers
```
SEEDS=""
PEERS="77c04ed628aa56322c45b8da14b8567c1bf322e5@65.108.98.53:11056,d22bf273fc96fd6b184333271aa1e979e10876ff@188.40.140.51:21000"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.andromedad/config/config.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.andromedad/config/app.toml
```

## Set minimum gas price and null indexer
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uandr\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.andromedad/config/config.toml
```

## Create Service
```
sudo tee /etc/systemd/system/andromedad.service > /dev/null <<EOF
[Unit]
Description=Andromeda
After=network-online.target

[Service]
User=$USER
ExecStart=$(which andromedad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Reset blockchain info and restart your node
```
sudo systemctl daemon-reload
sudo systemctl enable andromedad
andromedad tendermint unsafe-reset-all --home $HOME/.andromedad --keep-addr-book
sudo systemctl restart andromedad && sudo journalctl -u andromedad -f -o cat
```

<a name="state_sync"></a>
## (OPTIONAL) Use State Sync

### [State Sync guide](https://github.com/L0vd/Andromeda/tree/main/StateSync)


<a name="starting_validator"></a>
## Starting a validator

### 1. Add a new key
```
andromedad keys add $ANDROMEDA_WALLET
```
#### (OR)

### 1. Recover your key
```
andromedad keys add $ANDROMEDA_WALLET --recover
```

### 2. Request tokens from [faucet](https://discord.com/channels/1007329761229545512/1025144166117814404)

### 3. Create validator
```
andromedad tx staking create-validator \
--amount 1000000uandr \
--commission-max-change-rate "0.01" \
--commission-max-rate "0.20" \
--commission-rate "0.1" \
--min-self-delegation "1" \
--details "" \
--pubkey=$(andromedad tendermint show-validator) \
--moniker $ANDROMEDA_NODENAME \
--chain-id $ANDROMEDA_CHAIN_ID \
--gas-prices 0.025uandr \
--from $ANDROMEDA_WALLET \
--yes
```
<a name="useful_commands"></a>
## Useful commands

### Check status
```
andromedad status | jq
```

### Check logs
```
sudo journalctl -u andromedad -f
```

### Check wallets
```
andromedad keys list
```

### Check balance
```
andromedad q bank balances $ANDROMEDA_WALLET
```

### Send tokens
```
andromedad tx bank send <FROM_WALLET_ADDRESS> <TO_WALLET_ADDRESS> <AMOUNT>uandr --fees 0uandr
```

### Delegate tokens to validator
```
andromedad tx staking delegate <MONIKER> <AMOUNT>uandr --from $ANDROMEDA_WALLET --chain-id $ANDROMEDA_CHAIN_ID --fees 0uandr
```

### Vote for proposal
#### Yes
```
andromedad tx gov vote <PROPOSAL_NUMBER> yes --from $ANDROMEDA_WALLET --chain-id $ANDROMEDA_CHAIN_ID --fees 0uandr
```
#### No
```
andromedad tx gov vote <PROPOSAL_NUMBER> no --from $ANDROMEDA_WALLET --chain-id $ANDROMEDA_CHAIN_ID --fees 0uandr
```
