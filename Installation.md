   # install dependencies, if needed   

`sudo apt update && sudo apt upgrade -y`   
`sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y`   

   # install go, if needed   
`cd $HOME   
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin`   

Sett Wallet="xxx" to your own wallet name   
and set moniker="xxx" to your own moniker    
example:   
WALLET=Captain   
Moniker=MY1stNode   
   # set vars   
`echo "export WALLET="xxx"" >> $HOME/.bash_profile`   
`echo "export MONIKER="xxx"" >> $HOME/.bash_profile`   
`echo "export OG_CHAIN_ID="zgtendermint_9000-1"" >> $HOME/.bash_profile`   
`echo "export OG_PORT="47"" >> $HOME/.bash_profile`   
`source $HOME/.bash_profile`   

   # download binary   
`cd $HOME
rm -rf 0g-evmos
git clone https://github.com/0glabs/0g-evmos.git
cd 0g-evmos
git checkout v1.0.0-testnet
make build
mv $HOME/0g-evmos/build/evmosd $HOME/go/bin/`

   # download binary
`cd $HOME`   
`rm -rf 0g-evmos
git clone https://github.com/0glabs/0g-evmos.git
cd 0g-evmos
git checkout v1.0.0-testnet
make build
mv $HOME/0g-evmos/build/evmosd $HOME/go/bin/`

# config and init app   
`evmosd config node tcp://localhost:${OG_PORT}657
evmosd config keyring-backend os
evmosd config chain-id zgtendermint_9000-1
evmosd init "xxx" --chain-id zgtendermint_9000-1`   

   # Download genesis.json   
   `wget https://github.com/0glabs/0g-evmos/releases/download/v1.0.0-testnet/genesis.json -O $HOME/.evmosd/config/genesis.json`   
      # set seeds and peers   
      `PEERS="1248487ea585730cdf5d3c32e0c2a43ad0cda973@peer-zero-gravity-testnet.trusted-point.com:26326" && \
SEEDS="8c01665f88896bca44e8902a30e4278bed08033f@54.241.167.190:26656,b288e8b37f4b0dbd9a03e8ce926cd9c801aacf27@54.176.175.48:26656,8e20e8e88d504e67c7a3a58c2ea31d965aa2a890@54.193.250.204:26656,e50ac888b35175bfd4f999697bdeb5b7b52bfc06@54.215.187.94:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.evmosd/config/config.toml`   
   # set custom ports in app.toml   
`sed -i.bak -e "s%:1317%:${OG_PORT}317%g;
s%:8080%:${OG_PORT}080%g;
s%:9090%:${OG_PORT}090%g;
s%:9091%:${OG_PORT}091%g;
s%:8545%:${OG_PORT}545%g;
s%:8546%:${OG_PORT}546%g;
s%:6065%:${OG_PORT}065%g" $HOME/.evmosd/config/app.toml`   

   # set custom ports in config.toml file   
`sed -i.bak -e "s%:26658%:${OG_PORT}658%g;
s%:26657%:${OG_PORT}657%g;
s%:6060%:${OG_PORT}060%g;
s%:26656%:${OG_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${OG_PORT}656\"%;
s%:26660%:${OG_PORT}660%g" $HOME/.evmosd/config/config.toml`

   # config pruning   
`sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.evmosd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.evmosd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.evmosd/config/app.toml`   
   # set minimum gas price, enable prometheus and disable indexing   
`sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0agnet"|g' $HOME/.evmosd/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.evmosd/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.evmosd/config/config.toml`

   # create service file   
`sudo tee /etc/systemd/system/evmosd.service > /dev/null <<EOF
[Unit]
Description=Og node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.evmosd
ExecStart=$(which evmosd) start --home $HOME/.evmosd
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF`
   # reset and download snapshot   
`evmosd tendermint unsafe-reset-all --home $HOME/.evmosd`   
`wget -O $HOME/.evmosd/config/addrbook.json https://rpc-zero-gravity-testnet.trusted-point.com/addrbook.json`
   # enable and start service   
`sudo systemctl daemon-reload
sudo systemctl enable evmosd
sudo systemctl restart evmosd && sudo journalctl -u evmosd -f`   
   # to create a new wallet, use the following command. don’t forget to save the mnemonic   
`evmosd keys add $WALLET`   
   # to restore exexuting wallet, use the following command   
`evmosd keys add $WALLET --recover`
   # save wallet and validator address   
`WALLET_ADDRESS=$(evmosd keys show $WALLET -a)
VALOPER_ADDRESS=$(evmosd keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile`
   # check sync status   
`evmosd status 2>&1 | jq .SyncInfo`
   # before creating a validator, you need to fund your wallet and check balance   
`evmosd query bank balances $WALLET_ADDRESS`

# Create validator   
wait till sync status goes to "False" before u creating the validator   
 `evmosd tx staking create-validator \
--amount 1000000aevmos \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(evmosd tendermint show-validator) \
--moniker "xxx" \
--identity "" \
--details "" \
--chain-id zgtendermint_9000-1 \
--gas 500000 --gas-prices 99999aevmos \
-y`
