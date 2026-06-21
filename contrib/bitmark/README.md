# ElectrumX for Bitmark

This is a fork of [ElectrumX](https://github.com/kyuupichan/electrumx) configured for Bitmark (BTM).

## Prerequisites

1. A synced bitmarkd node with RPC enabled
2. Python 3.8+
3. Required system packages

### bitmarkd Configuration

Ensure your `~/.bitmark/bitmark.conf` has RPC enabled:

```
server=1
rpcuser=yourusername
rpcpassword=yourpassword
rpcallowip=127.0.0.1
rpcport=9266
txindex=1
```

## Installation

```bash
# Install system dependencies (Ubuntu/Debian)
sudo apt-get install python3-pip python3-venv libleveldb-dev

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install ElectrumX
pip install .

# Or for development
pip install -e .
```

## Configuration

Copy and edit the example configuration:

```bash
cp contrib/bitmark/electrumx_bitmark.conf ~/.electrumx-bitmark.conf

# Edit with your bitmarkd RPC credentials
nano ~/.electrumx-bitmark.conf
```

Key settings to update:
- `DAEMON_URL`: Your bitmarkd RPC URL with credentials
- `DB_DIRECTORY`: Where to store the ElectrumX database

## Running

```bash
# Source configuration
source ~/.electrumx-bitmark.conf

# Run ElectrumX
./electrumx_server
```

For initial sync, this will take some time as it indexes all blockchain data.

## Systemd Service

For production, create a systemd service:

```bash
sudo cp contrib/bitmark/electrumx-bitmark.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable electrumx-bitmark
sudo systemctl start electrumx-bitmark
```

## Connecting Electrum-Bitmark

Once ElectrumX is running and synced, configure Electrum-Bitmark to connect:

```bash
electrum-bitmark --server 127.0.0.1:50001:t
```

Or add to your wallet's server list.

## Ports

- **50001**: TCP (unencrypted Electrum protocol)
- **50002**: SSL (encrypted, requires certificate)

## Troubleshooting

### "Connection refused" from bitmarkd
- Ensure bitmarkd is running and fully synced
- Check RPC credentials match
- Verify `rpcallowip` includes ElectrumX's IP

### Database errors
- Ensure `DB_DIRECTORY` exists and is writable
- Check disk space

### Slow sync
- Increase `CACHE_MB` if you have RAM available
- Ensure bitmarkd has `txindex=1`
