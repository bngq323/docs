This docuement is for Namada shielded-expedition "S" class: 
"Operating IBC/ Interoperability infrastructure" - "Operate a shielded expedition-compatible Osmosis testnet relayer"  

My Osmosis node provides rpc service coordinating with My Namada SE node to support IBC relayer channel. 
We can transfer assets via Osmosis and SE testnets. 
The following is the process deploying on the standlone VPS ubuntu22.  

---
Channel:  
Namada <-> Osmosis  
"channel-150", "channel-5683" 

RPC:  
Namada:  "144.76.65.89:26657"  
Osmosis: "127.0.0.1:26657"  


# Install Hermes and Namada SE
Download hermes v1.7.4 beta7
```
wget https://github.com/heliaxdev/hermes/releases/download/v1.7.4-namada-beta7/hermes-v1.7.4-namada-beta7-x86_64-unknown-linux-gnu.tar.gz
tar -xvf hermes-v1.7.4-namada-beta7-x86_64-unknown-linux-gnu.tar.gz
sudo mv hermes /usr/local/bin && rm hermes-v1.7.4-namada-beta7-x86_64-unknown-linux-gnu.tar.gz
hermes --version
```

Deploy hermes service
```
sudo tee /etc/systemd/system/hermesd.service > /dev/null <<EOF
[Unit]
Description=hermes service
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.hermes
ExecStart=/usr/local/bin/hermes --config $HOME/.hermes/config.toml start
StandardOutput=syslog
StandardError=syslog
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo chmod 755 /etc/systemd/system/hermesd.service
sudo systemctl daemon-reload
sudo systemctl enable hermesd
```
Install Namada SE v0.31.4
```
wget https://github.com/anoma/namada/releases/download/v0.31.4/namada-v0.31.4-Linux-x86_64.tar.gz
tar -xvf namada-v0.31.4-Linux-x86_64.tar.gz && cd namada-v0.31.4-Linux-x86_64
sudo cp namada namadan namadaw namadac /usr/local/bin/ && cd .. && rm namada-v0.31.4-Linux-x86_64.tar.gz && rm -rf namada-v0.31.4-Linux-x86_64
namada --version
Namada v0.31.4

namadac utils join-network --chain-id shielded-expedition.88f17d1d14
```

# Prepare wallets for Namada and Osmosis
Import Namada SE wallet registered before
```
namadaw derive --alias se_wallet

namadaw find --alias se_wallet
Found transparent keys:
  Alias "se_wallet" (encrypted):
    Public key hash: 2AD6DC2F0119A52E9BA2E0C2D096D878475AE459
    Public key: tpknam1qr8plwnj863wsa8lcm92daynl3u8z68uyjtkj8m0l6c6e3rry0euxhyey48
Found transparent address:
  "se_wallet": Implicit: tnam1qq4ddhp0qyv62t5m5tsv95ykmpuywkhytyp874cv

namadac balance --owner se_wallet --node "144.76.65.89:26657"
naan: 1542.207602
```
Generate an account for Osmosis and get faucet
```
osmosisd keys add osmo_wallet
- address: osmo1gzpus7t4k8jtjvrqp3u3glmjawk3taqznfukx6
  name: osmo_wallet
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AmnzLbKxbP40aZmvi+0XI5jArb8cg0GciumnRoO6CKuT"}'
  type: local
**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.
concert negative vocal viable moment excite curtain bid humble humble return among online net measure sound drum mountain subject spirit lab ordinary engage client

osmosisd query bank balances osmo1gzpus7t4k8jtjvrqp3u3glmjawk3taqznfukx6
balances:
- amount: "100000000"
  denom: uosmo
```
# configure Hermes
mkdir $HOME/.hermes   
vi $HOME/.hermes/config.toml
```
...

[[chains]]
id = 'shielded-expedition.88f17d1d14' 
type = 'Namada'
rpc_addr = 'http://144.76.65.89:26657'  
grpc_addr = 'http://144.76.65.89:9090' 
event_source = { mode = 'push', url = 'ws://144.76.65.89:26657/websocket', batch_delay = '500ms' } 
account_prefix = ''
key_name = 'se_wallet' 
store_prefix = 'ibc'
gas_price = { price = 0.0001, denom = 'tnam1qxvg64psvhwumv3mwrrjfcz0h3t3274hwggyzcee' } 
rpc_timeout = '30s'

[[chains]]
id = 'osmo-test-5'
type = 'CosmosSdk'
rpc_addr = 'http://127.0.0.1:26657' 
grpc_addr = 'http://127.0.0.1:9090'
event_source = { mode = 'push', url = 'ws://127.0.0.1:26657/websocket', batch_delay = '500ms' } 
account_prefix = 'osmo'
key_name = 'relayer_osmo'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 400000
max_gas = 120000000
gas_price = { price = 0.0025, denom = 'uosmo' }
gas_multiplier = 1.2
max_msg_num = 30
max_tx_size = 1800000
clock_drift = '15s'
max_block_time = '30s'
trusting_period = '4days'
trust_threshold = { numerator = '1', denominator = '3' }
rpc_timeout = '30s'
```
# Add Keys in Hermes
Add Namada key:
```
hermes --config $HOME/.hermes/config.toml keys add --chain shielded-expedition.88f17d1d14 --key-file $HOME/.local/share/namada/shielded-expedition.88f17d1d14/wallet.toml
```
Add Osmosis key:  
Input mnemonic phrase of osmo_wallet in ./mnemonic
```
hermes --config $HOME/.hermes/config.toml keys add --chain osmo-test-5 --mnemonic-file ./mnemonic
```

# Establish IBC channel
```bash
hermes --config $HOME/.hermes/config.toml \
  create channel \
  --a-chain shielded-expedition.88f17d1d14 \
  --b-chain osmo-test-5 \
  --a-port transfer \
  --b-port transfer \
  --new-client-connection --yes
```
Channels are generated
```
channel handshake already finished for Channel { ordering: ORDER_UNORDERED, a_side: ChannelSide { chain: BaseChainHandle { chain_id: shielded-expedition.88f17d1d14 }, client_id: 07-tendermint-439, connection_id: connection-198, port_id: transfer, channel_id: channel-120, version: None }, b_side: ChannelSide { chain: BaseChainHandle { chain_id: osmo-test-5 }, client_id: 07-tendermint-2041, connection_id: connection-1977, port_id: transfer, channel_id: channel-5624, version: None }, connection_delay: 0ns }
SUCCESS Channel {
    ordering: Unordered,
    a_side: ChannelSide {
        chain: BaseChainHandle {
            chain_id: ChainId {
                id: "shielded-expedition.88f17d1d14",
                version: 0,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-439",
        ),
        connection_id: ConnectionId(
            "connection-198",
        ),
        port_id: PortId(
            "transfer",
        ),
        channel_id: Some(
            ChannelId(
                "channel-120",
            ),
        ),
        version: None,
    },
    b_side: ChannelSide {
        chain: BaseChainHandle {
            chain_id: ChainId {
                id: "osmo-test-5",
                version: 5,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-2041",
        ),
        connection_id: ConnectionId(
            "connection-1977",
        ),
        port_id: PortId(
            "transfer",
        ),
        channel_id: Some(
            ChannelId(
                "channel-5624",
            ),
        ),
        version: None,
    },
    connection_delay: 0ns,
}
```

# Start service of hermes 
```
sudo systemctl start hermesd && sudo journalctl -u hermesd -f -o cat
```

# Transfer shielded naan to osmosis
Check spending key balance.
```
namadac balance --owner my-spending-key --node "144.76.65.89:26657"
Last committed epoch: 7
naan : 1.5
```
Transfer shielded naan from Namada to Osmosis
```
namadac --base-dir $HOME/.local/share/namada \
    ibc-transfer \
    --amount 1 \
    --source my-spending-key \
    --signing-keys se_wallet \
    --receiver osmo1gzpus7t4k8jtjvrqp3u3glmjawk3taqznfukx6 \
    --token naan \
    --channel-id "channel-120" \
    --node "144.76.65.89:26657" \
    --memo tpknam1qr8plwnj863wsa8lcm92daynl3u8z68uyjtkj8m0l6c6e3rry0euxhyey48
Enter your decryption password: 
Enter your decryption password: 
Transaction added to mempool.
Wrapper transaction hash: 0A0AB8E71960546622C6B9FC38423FC9311BC441086C8C3A3FC981947516EC0E
Inner transaction hash: DAD377AD939978E15AE89EB10EE8A0A86236E63006F198204D710881144FFBEE
Wrapper transaction accepted at height 30238. Used 120 gas.
Waiting for inner transaction result...
Transaction was successfully applied at height 30239. Used 8602 gas.
```
Check balance of osmo_wallet. Received 1 naan successfully.
```
osmosisd query bank balances osmo1gzpus7t4k8jtjvrqp3u3glmjawk3taqznfukx6
balances:
- amount: "1"
  denom: ibc/E1121FF5D5A925B1E09E2C77E23E7BA3D12002ABAFB71F0DEDD490145F767829
- amount: "99986299"
  denom: uosmo
```

Transfer uosmo from Osmosis to Namada transparant address "se_wallet"
```
osmosisd tx ibc-transfer transfer \
  transfer \
  channel-5624 \
  tnam1qq4ddhp0qyv62t5m5tsv95ykmpuywkhytyp874cv \
  2000000uosmo \
  --from osmo_wallet \
  --gas auto \
  --gas-prices 0.035uosmo \
  --gas-adjustment 1.2 \
  --node "http://127.0.0.1:26657" \
  --home "$HOME/.osmosisd" \
  --chain-id osmo-test-5 \
  --yes
Enter keyring passphrase (attempt 1/3):
gas estimate: 131150
code: 0
codespace: ""
data: ""
events: []
gas_used: "0"
gas_wanted: "0"
height: "0"
info: ""
logs: []
raw_log: '[]'
timestamp: ""
tx: null
txhash: 47DDBD4FA11BB6C3F91EFEFDBBB3266E351E086B488993B0D100FD6945DE816D
```
Check balance of "se_wallet"
```
namadac balance --owner se_wallet --node "144.76.65.89:26657"
naan: 1500.207602
transfer/channel-120/uosmo: 2000000
```

# Reviving client
Due to Namada network upgrade and Hermes client expired. We create new channels.   
Channel:  
Namada <-> Osmosis  
"channel-150", "channel-5683" 
```
hermes create client --host-chain shielded-expedition.88f17d1d14 --reference-chain osmo-test-5

SUCCESS SendPacket(
    SendPacket {
        packet: Packet {
            sequence: Sequence(
                389,
            ),
            source_port: PortId(
                "transfer",
            ),
            source_channel: ChannelId(
                "channel-150",
            ),
            destination_port: PortId(
                "transfer",
            ),
            destination_channel: ChannelId(
                "channel-5683",
            ),
            data: [123, 34, 100, 101, 110, 111, 109, 34, 58, 34, 116, 114, 97, 110, 115, 102, 101, 114, 47, 99, 104, 97, 110, 110, 101, 108, 45, 49, 53, 48, 47, 116, 114, 97, 110, 115, 102, 101, 114, 47, 99, 104, 97, 110, 110, 101, 108, 45, 53, 54, 48, 53, 47, 116, 110, 97, 109, 49, 113, 120, 118, 103, 54, 52, 112, 115, 118, 104, 119, 117, 109, 118, 51, 109, 119, 114, 114, 106, 102, 99, 122, 48, 104, 51, 116, 51, 50, 55, 52, 104, 119, 103, 103, 121, 122, 99, 101, 101, 34, 44, 34, 97, 109, 111, 117, 110, 116, 34, 58, 34, 49, 34, 44, 34, 115, 101, 110, 100, 101, 114, 34, 58, 34, 116, 110, 97, 109, 49, 113, 113, 51, 115, 102, 101, 55, 107, 51, 113, 118, 118, 108, 109, 56, 57, 100, 57, 100, 50, 101, 118, 52, 97, 110, 101, 122, 99, 54, 102, 53, 57, 114, 113, 121, 115, 56, 116, 57, 106, 34, 44, 34, 114, 101, 99, 101, 105, 118, 101, 114, 34, 58, 34, 111, 115, 109, 111, 49, 121, 102, 53, 118, 106, 97, 51, 114, 100, 113, 113, 116, 99, 48, 104, 121, 97, 115, 51, 121, 100, 48, 102, 109, 122, 115, 48, 112, 55, 52, 102, 113, 107, 55, 53, 51, 112, 104, 34, 44, 34, 109, 101, 109, 111, 34, 58, 34, 34, 125],
            timeout_height: Never,
            timeout_timestamp: Timestamp {
                time: Some(
                    Time(
                        2024-02-20 14:06:15.812007219,
                    ),
                ),
            },
        },
    },
)
```

# Check if the channel is operational
Tx Osmosis -> Namada
```
osmosisd tx ibc-transfer transfer \
  transfer \
  channel-5683 \
  tnam1qq4ddhp0qyv62t5m5tsv95ykmpuywkhytyp874cv \
  3000000uosmo \
  --from osmo_wallet \
  --gas auto \
  --gas-prices 0.035uosmo \
  --gas-adjustment 1.2 \
  --node "http://127.0.0.1:26657" \
  --home "$HOME/.osmosisd" \
  --chain-id osmo-test-5 \
  --yes
Enter keyring passphrase (attempt 1/3):
gas estimate: 116648
code: 0
codespace: ""
data: ""
events: []
gas_used: "0"
gas_wanted: "0"
height: "0"
info: ""
logs: []
raw_log: '[]'
timestamp: ""
tx: null
txhash: DD89F3E927B260CD653F15F4794FA18FC4A7568F48C6B006D26AC74A7BCF85D6
```

Tx Namada -> Osmosis
```
namadac --base-dir $HOME/.local/share/namada \
    ibc-transfer \
    --amount 1 \
    --source se_wallet \
    --signing-keys se_wallet \
    --receiver osmo1gzpus7t4k8jtjvrqp3u3glmjawk3taqznfukx6 \
    --token naan \
    --channel-id "channel-150" \
    --node "144.76.65.89:26657" \
    --memo tpknam1qr8plwnj863wsa8lcm92daynl3u8z68uyjtkj8m0l6c6e3rry0euxhyey48
Enter your decryption password: 
Transaction added to mempool.
Wrapper transaction hash: 84FB8F172274FB0B9EF1B829A86DCF0DC63889D9FE3693645BE7F2826915D774
Inner transaction hash: 81405F7C5CE208FA3A2FC8463E25C2DE5180DEA7BA52755100C9062643A06718
Wrapper transaction accepted at height 40281. Used 26 gas.
Waiting for inner transaction result...
Transaction was successfully applied at height 40282. Used 6193 gas.
namadanet@vmi1662412:~$ osmosisd query bank balances osmo1gzpus7t4k8jtjvrqp3u3glmjawk3taqznfukx6
```

# Check balance again
```
namadac balance --owner se_wallet --node "144.76.65.89:26657"
naan: 1452.807602
transfer/channel-120/uosmo: 2000000
transfer/channel-150/uosmo: 3000000
```

```
query bank balances osmo1gzpus7t4k8jtjvrqp3u3glmjawk3taqznfukx6
balances:
- amount: "1"
  denom: ibc/AA2E8FA94E10000CD564A0458334A55D002EE59BE95D693C060AFE39559F32D6
- amount: "1"
  denom: ibc/E1121FF5D5A925B1E09E2C77E23E7BA3D12002ABAFB71F0DEDD490145F767829
- amount: "297973482"
  denom: uosmo
```
