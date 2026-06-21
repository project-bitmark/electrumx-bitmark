# ElectrumX-Bitmark - Electrum server for Bitmark

```
Licence: MIT
Language: Python (>= 3.10)
```

ElectrumX-Bitmark is an Electrum protocol server for
[Bitmark](https://github.com/project-bitmark/bitmark) (ticker **BTMK**). It indexes the
Bitmark blockchain from a `bitmarkd` full node and serves
[Electrum-Bitmark](https://github.com/project-bitmark/electrum-bitmark) wallet clients. It is
a fork of [spesmilo/electrumx](https://github.com/spesmilo/electrumx).

## What's different from upstream ElectrumX

Bitmark is a multi-algorithm, merge-mined chain, so the block format is non-standard:

- **Bitmark coin class** (`electrumx/lib/coins.py`) with a custom `DeserializerBitmark`
  (`electrumx/lib/tx.py`) that parses all 8 PoW algorithms. The algorithm is encoded in the
  block version (bit 8 = auxpow, bits 9-11 = algo).
- **Equihash** blocks use a ~1487-byte extended header; the block identity hash is double
  SHA-256 over the full header ("GetHashE"). All other algos use the standard 80-byte header.
- **AuxPoW (merged mining).** For CryptoNight/Equihash the parent coinbase and parent header
  are serialized as opaque length-prefixed vectors — handled per the Bitmark daemon's
  `primitives/{block,pureheader,transaction}.h`.
- **Multi-algo-aware header serving** that strips the auxpow blob and never truncates equihash
  headers to 80 bytes.
- Graceful handling of the **older Bitmark daemon** (v0.9.7.4), which lacks
  `estimatesmartfee`/`estimatefee`/`getmempoolinfo`.

Also fixes an upstream bug in `block_headers_array` that mis-split variable-length headers.

## Requirements

- A synced `bitmarkd` full node with RPC enabled (default RPC port 9266) and `txindex=1`.
- Python 3.10+, and a DB backend (`leveldb` via `plyvel`, or `rocksdb`).

## Setup

```bash
python3 -m venv venv && source venv/bin/activate
pip install -e . plyvel

# configure (see contrib/bitmark/ for examples)
export COIN=Bitmark
export NET=mainnet
export DB_ENGINE=leveldb
export DB_DIRECTORY=/var/lib/electrumx-bitmark
export DAEMON_URL=http://rpcuser:rpcpass@127.0.0.1:9266/
export SERVICES=tcp://:50001,rpc://:8001

./electrumx_server
```

Initial sync indexes the whole chain; the server starts accepting Electrum clients on
`tcp://…:50001` once it has caught up to the daemon tip. See `contrib/bitmark/` for a sample
config and systemd unit.

## Credits

Built on [ElectrumX](https://github.com/spesmilo/electrumx) (originally by Neil Booth,
maintained by the Electrum developers). See the upstream project for full server documentation.
