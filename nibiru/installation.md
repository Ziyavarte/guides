# **Installation**

### **Minimum hardware requirements**

- 4CPU
- 16GB RAM
- 1TB of disk space (SSD)

### **Update the system**

```bash
sudo apt update && sudo apt upgrade --yes
```

### **Install nibid**

Option 1: Use this version if you plan to sync from genesis block; you will need to swap it to the current one at the upgrade height (either manually or with Cosmovisor)

```bash
curl -s https://get.nibiru.fi/@v1.0.0! | bash
```

Option 2: Use this version if you plan to use state-sync or data snapshot

```bash
curl -s https://get.nibiru.fi/@v1.3.0-rc1! | bash
```

### **Verify nibid version**

```bash
nibid version
# Should output v1.0.0 or v1.3.0-rc1 depending on chosen approach
```

---

## **Init the Chain**

1. Init the chain
    
    ```bash
    nibid init <moniker-name> --chain-id=nibiru-testnet-1 --home $HOME/.nibid
    ```
    
2. Copy the genesis file to the `$HOME/.nibid/config` folder.
    
    You can get genesis from our networks endpoint with:
    
    ```bash
    NETWORK=nibiru-testnet-1
    curl -s https://networks.testnet.nibiru.fi/$NETWORK/genesis > $HOME/.nibid/config/genesis.json
    ```
    
    Or you can download it from the Tendermint RPC endpoint.
    
    ```bash
    curl -s https://rpc.testnet-1.nibiru.fi/genesis | jq -r .result.genesis > $HOME/.nibid/config/genesis.json
    ```
    
    **(Optional) Verify Genesis File Checksum**
    
    ```bash
    shasum -a 256 $HOME/.nibid/config/genesis.json
    # 23c95807de3c663e8d2018a7f10aa27cb773a84d99f331e4e07d367aceca19f5 $HOME/.nibid/config/genesis.json
    ```
    
3. Update persistent peers list in the configuration file `$HOME/.nibid/config/config.toml`.
    
    ```bash
    NETWORK=nibiru-testnet-1
    sed -i 's|\<persistent_peers\> =.*|persistent_peers = "'$(curl -s https://networks.testnet.nibiru.fi/$NETWORK/peers)'"|g' $HOME/.nibid/config/config.toml
    ```
    
4. Set minimum gas prices
    
    ```bash
    sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025unibi"/g' $HOME/.nibid/config/app.toml
    ```
    
5. (Optional) Configure one of the following options to catch up faster with the network
    
    Option 1: Setup state-sync
    
    ```bash
    NETWORK=nibiru-testnet-1
    config_file="$HOME/.nibid/config/config.toml"
    sed -i "s|enable =.*|enable = true|g" "$config_file"
    sed -i "s|rpc_servers =.*|rpc_servers = \"$(curl -s https://networks.testnet.nibiru.fi/$NETWORK/rpc_servers)\"|g" "$config_file"
    sed -i "s|trust_height =.*|trust_height = \"$(curl -s https://networks.testnet.nibiru.fi/$NETWORK/trust_height)\"|g" "$config_file"
    sed -i "s|trust_hash =.*|trust_hash = \"$(curl -s https://networks.testnet.nibiru.fi/$NETWORK/trust_hash)\"|g" "$config_file"
    ```
    
    Option 2: Download and extract data snapshot
    
    You can check [available snapshots list for nibiru-testnet-1](https://networks.testnet.nibiru.fi/nibiru-testnet-1/snapshots) to locate the snapshot with the date and type that you need
    
    ```bash
    curl -o nibiru-testnet-1-<timestamp>-<type>.tar.gz https://storage.googleapis.com/nibiru-testnet-1-snapshots/nibiru-testnet-1-<timestamp>-<type>.tar.gz
    tar -zxvf nibiru-testnet-1-<timestamp>-<type>.tar.gz -C $HOME/.nibid/
    ```
    
6. Start your node (choose one of the options)
    
    Option 1: **Systemd + Systemctl**
    
    After defining a [service file for use with `systemctl`](https://nibiru.fi/docs/run-nodes/full-nodes/systemctl), you can execute:
    
    ```bash
    sudo systemctl start nibiru
    ```
    
    Option 2: **Cosmovisor**
    
    After defining a [service file for use with `cosmovisor`](https://nibiru.fi/docs/run-nodes/full-nodes/cosmovisor), you can execute:
    
    ```bash
    sudo systemctl start cosmovisor-nibiru
    ```
    
    Option 3: Without a daemon
    
    ```bash
    nibid start
    ```
    
7. Request tokens from the [Web Faucet for nibiru-testnet-1 (opens new window)](https://app.nibiru.fi/faucet) if required.
    
    To create a local key pair, you may use the following command:
    
    ```bash
    nibid keys add <key-name>
    ```
