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
```

