This docuement is for Namada shielded-expedition "S" class: 
"Operating IBC/ Interoperability infrastructure" - "Operate a Shielded Expedition-compatible Cosmos testnet relayer"  

My Osmosis node provides rpc service coordinating with Namada SE node to support IBC relayer channel. We can transfer assets via Osmosis and SE testnets. 
The following is the process deploying on the standlone VPS ubuntu22. 

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
event_source = { mode = 'push', url = 'ws://94.130.90.47:26657/websocket', batch_delay = '500ms' } 
account_prefix = ''
key_name = 'se_wallet' 
store_prefix = 'ibc'
gas_price = { price = 0.0001, denom = 'tnam1qxvg64psvhwumv3mwrrjfcz0h3t3274hwggyzcee' } 
rpc_timeout = '20s'

[[chains]]
id = 'osmo-test-5'
type = 'CosmosSdk'
rpc_addr = 'http://127.0.0.1:26657' 
grpc_addr = 'http://127.0.0.1:9090'
event_source = { mode = 'push', url = 'ws://127.0.0.1:26657/websocket', batch_delay = '500ms' } 
account_prefix = 'osmo'
key_name = 'osmo_wallet'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
gas_price = { price = 0.0025, denom = 'uosmo' }
gas_multiplier = 1.1
trusting_period = '5days'
trust_threshold = { numerator = '1', denominator = '3' }
rpc_timeout = '20s'
```
# Add Keys in Hermes
```
Add Namada key:
hermes --config $HOME/.osmosisd/config.toml keys add --chain shielded-expedition.88f17d1d14 --key-file $HOME/.local/share/namada/shielded-expedition.88f17d1d14/wallet.toml

Add Osmosis key:
Input mnemonic phrase of osmo_wallet in ./mnemonic
hermes --config $HOME/.osmosisd/config.toml keys add --chain osmo-test-5 --mnemonic-file ./mnemonic
```
