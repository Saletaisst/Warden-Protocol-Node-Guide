# Warden-Protocol-Node-Guide
This simple guide will help you put a Warden Protocol project node the way I do. Just copy the commands and don't think about anything!


## Version History 📜

| Release | Upgrade Block Height | Upgrade Date      |
|---------|----------------------|-------------------|
| v0.3.0  | genesis              |                   |
| v0.4.0  | 1520000              | August 1, 2024    |

## Prerequisites 🛠️

For running public testnet nodes, we recommend:

- **CPU**: Minimum of 8 cores
- **RAM**: At least 32GB
- **Disk Space**: 300GB
- **Software**: Go installed on your machine

## Installation Guide 🚀

### 1. Getting Started

To join the Buenavista testnet, you need to install `wardend` (the Warden binary). You can do this in one of two ways:

#### Option 1: Prebuilt Binary

1. Download the binary for your platform from the release page and unzip it.
2. Inside the archive, you will find the `wardend` binary.
3. Initialize the chain home folder with:

    ```bash
    ./wardend init <custom_moniker>
    ```

#### Option 2: Build from Source

1. Clone the repository and switch to the appropriate branch:

    ```bash
    git clone --depth 1 --branch v0.3.0 https://github.com/warden-protocol/wardenprotocol/
    ```

2. Build the binary:

    ```bash
    just build
    ```

3. Initialize the chain home folder:

    ```bash
    build/wardend init <custom_moniker>
    ```

### 2. Configuration ⚙️

#### Prepare the Genesis File

1. Navigate to the configuration directory:

    ```bash
    cd $HOME/.warden/config
    ```

2. Remove the existing genesis file:

    ```bash
    rm genesis.json
    ```

3. Download the new genesis file:

    ```bash
    wget https://raw.githubusercontent.com/warden-protocol/networks/main/testnets/buenavista/genesis.json
    ```

#### Set Mandatory Configuration Options

1. Set the minimum gas price:

    ```bash
    sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.0025uward"/' app.toml
    ```

2. Set persistent peers:

    ```bash
    sed -i 's/persistent_peers = ""/persistent_peers = "92ba004ac4bcd5afbd46bc494ec906579d1f5c1d@52.30.124.80:26656,ed5781ea586d802b580fdc3515d75026262f4b9d@54.171.21.98:26656"/' config.toml
    ```

### 3. State Sync (Optional but Recommended) 🌐

To speed up the initial sync, you can use state sync. This allows you to download the state at a specific height from a trusted node and then sync only the blocks thereafter.

1. Set the RPC endpoint:

    ```bash
    export SNAP_RPC_SERVERS="https://rpc.buenavista.wardenprotocol.org:443,https://rpc.buenavista.wardenprotocol.org:443"
    ```

2. Fetch the latest block height and trust hash:

    ```bash
    export LATEST_HEIGHT=$(curl -s "https://rpc.buenavista.wardenprotocol.org/block" | jq -r .result.block.header.height)
    export BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
    export TRUST_HASH=$(curl -s "https://rpc.buenavista.wardenprotocol.org/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
    ```

3. Verify the variables:

    ```bash
    echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
    ```

4. Update the configuration for state sync:

    ```bash
    sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
    s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC_SERVERS\"| ; \
    s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
    s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.warden/config/config.toml
    ```

### 4. Starting Your Node 🚀

Finally, start your node with:

```bash
wardend start
