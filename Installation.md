   # install dependencies, if needed
`sudo apt update && sudo apt upgrade -y`   
`sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y`   

   #install go, if needed   
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
