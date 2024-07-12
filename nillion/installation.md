### Install Dependencies
```bash
# Update system package and install build tools
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential fail2ban ufw
sudo apt -qy upgrade
```

### Configure Moniker
```bash
# Replace <your-moniker-name> with your own validator name
MONIKER="<your-moniker-name>"
```

### Install Go
```bash
# install go version 1.21.3
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.21.3.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

### Build Binaries
```bash
# Cloning project repository & Compile binaries
cd $HOME
rm -rf nillion
git clone https://github.com/NillionNetwork/nilliond
cd nillion
git checkout v0.2.1
make build
```

```bash
# Prepare binaries for cosmovisor
mkdir -p $HOME/.nillionapp/cosmovisor/genesis/bin
mv build/nilchaind $HOME/.nillionapp/cosmovisor/genesis/bin/
rm -rf build
```

```bash
# Create symlinks
sudo ln -s $HOME/.nillionapp/cosmovisor/genesis $HOME/.nillionapp/cosmovisor/current -f
sudo ln -s $HOME/.nillionapp/cosmovisor/current/bin/nilchaind /usr/local/bin/nilchaind -f
```

### Cosmovisor Setup
```bash
# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

### Create Service
```bash
# Create a systemd service
sudo tee /etc/systemd/system/nillion.service > /dev/null << EOF
[Unit]
Description=nillion node service
After=network-online.target
 
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.nillionapp"
Environment="DAEMON_NAME=nilchaind"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.nillionapp/cosmovisor/current/bin"
 
[Install]
WantedBy=multi-user.target
EOF
```

### Enable Service
```bash
# Enable nillion systemd service
sudo systemctl daemon-reload
sudo systemctl enable nillion
```

### Initialize Node
```bash
# Setting node configuration
nilchaind config set client chain-id nillion-chain-testnet-1
nilchaind config set client keyring-backend test
nilchaind config set client node tcp://localhost:24757
```

```bash
# Initialize node
nilchaind init $MONIKER --chain-id nillion-chain-testnet-1
```

### Download Genesis & Addrbook
```bash
# Download genesis & addrbook file
curl -Ls https://snap.ziyavarte.net/nillion-testnet/genesis.json > $HOME/.nillionapp/config/genesis.json
curl -Ls https://snap.ziyavarte.net/nillion-testnet/addrbook.json > $HOME/.nillionapp/config/addrbook.json
```

### Configure Seeds
```bash
# Setting up a seed peers
sed -i -e "s|^seeds *=.*|seeds = \"d1d43cc7c7aef715957289fd96a114ecaa7ba756@testnet-seeds.ziyavarte.net:24710\"|" $HOME/.nillionapp/config/config.toml
```

### Configure Gas Prices
```bash
# Setting up a gas prices
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0unil\"|" $HOME/.nillionapp/config/app.toml
```

### Pruning Setting
```bash
# Configure pruning setting
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.nillionapp/config/app.toml
```

### Download Snapshots
```bash
Download latest chain snapshot
curl -L https://snap.ziyavarte.net/nillion-testnet/nillion-latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.nillionapp
[[ -f $HOME/.nillionapp/data/upgrade-info.json ]] && cp $HOME/.nillionapp/data/upgrade-info.json $HOME/.nillionapp/cosmovisor/genesis/upgrade-info.json
```

### Start Service
```bash
sudo systemctl start nillion
```
