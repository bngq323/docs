This docuement is for Namada shielded-expedition "S" class: 
"Operating IBC/ Interoperability infrastructure" - "Operate a Shielded Expedition-compatible Cosmos testnet relayer"  

My Osmosis node provides rpc service coordinating with Namada SE node to support IBC relayer channel. We can transfer assets via Osmosis and SE testnets. 
The following is the process deploying on the standlone VPS ubuntu22. 

# Install Hermes and Namada SE
```
wget https://github.com/heliaxdev/hermes/releases/download/v1.7.4-namada-beta7/hermes-v1.7.4-namada-beta7-x86_64-unknown-linux-gnu.tar.gz
tar -xvf hermes-v1.7.4-namada-beta7-x86_64-unknown-linux-gnu.tar.gz
sudo mv hermes /usr/local/bin && rm hermes-v1.7.4-namada-beta7-x86_64-unknown-linux-gnu.tar.gz
hermes --version
```
