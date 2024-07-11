### **Install Dependencies**

```bash
# Update system package and install build tools

sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential fail2ban ufw
sudo apt -qy upgrade
```

### **Configure Moniker**

```bash
# Replace <your-moniker-name> with your own validator name

MONIKER="<your-moniker-name>"
```

### **Install Go**

```bash
# install go version 1.20.7

sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.20.7.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

### **Build Binaries**

```bash
# Cloning project repository & Compile binaries

cd $HOME
rm -rf artela
git clone https://github.com/artela-network/artela artela
cd artela
git checkout v0.4.7-rc6
make build
```

```bash
# Prepare binaries for cosmovisor

mkdir -p $HOME/.artelad/cosmovisor/genesis/binmv build/artelad $HOME/.artelad/cosmovisor/genesis/bin/rm -rf build
```

```bash
# Create symlinks

sudo ln -s $HOME/.artelad/cosmovisor/genesis $HOME/.artelad/cosmovisor/current -fsudo ln -s $HOME/.artelad/cosmovisor/current/bin/artelad /usr/local/bin/artelad -f
```

### **Cosmovisor Setup**

```bash
# Install cosmovisor

go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

### **Create Service**

```bash
# Create a systemd service

sudo tee /etc/systemd/system/artela.service > /dev/null << EOF
[Unit]
Description=artela node service
After=network-online.target
 
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.artelad"
Environment="DAEMON_NAME=artelad"
Environment="UNSAFE_SKIP_BACKUP=true"
 
[Install]
WantedBy=multi-user.target
EOF
```

### **Enable Service**

```bash
# Enable artela systemd service

sudo systemctl daemon-reloadsudo systemctl enable artela
```

### **Initialize Node**

```bash
# Setting node configuration

artelad config chain-id artela_11822-1
artelad config keyring-backend test
artelad config node tcp://localhost:23457
```

```bash
# Initialize node

artelad init $MONIKER --chain-id artela_11822-1
```

### **Download Genesis & Addrbook**

```bash
# Download genesis & addrbook file

curl -Ls https://snap.ziyavarte.net/artela-testnet/genesis.json > $HOME/.artelad/config/genesis.json
curl -Ls https://snap.ziyavarte.net/artela-testnet/addrbook.json > $HOME/.artelad/config/addrbook.json
```

### **Configure Seeds**

```bash
# Setting up a seed peers

sed -i -e "s|^seeds *=.*|seeds = \"d1d43cc7c7aef715957289fd96a114ecaa7ba756@testnet-seeds.ziyavarte.net:23410\"|" $HOME/.artelad/config/config.toml
```

### **Configure Gas Prices**

```bash
# Setting up a gas prices

sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025uart\"|" $HOME/.artelad/config/app.toml
```

### **Pruning Setting**

```bash
# Configure pruning setting

sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.artelad/config/app.toml
```

### **Download Snapshots**

```bash
# Download latest chain snapshot

curl -L https://snap.ziyavarte.net/artela-testnet/artela-latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.artelad
[[ -f $HOME/.artelad/data/upgrade-info.json ]] && cp $HOME/.artelad/data/upgrade-info.json $HOME/.artelad/cosmovisor/genesis/upgrade-info.json
```

### **Start Service**

```bash
sudo systemctl start artela
```
