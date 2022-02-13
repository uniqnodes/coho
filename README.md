# Install Golang (go)  
https://github.com/erknfe/go-lang  

# Install Starport  
`curl https://get.starport.network/starport | bash`  
`sudo mv starport /usr/local/bin/`  

# Install CoHo  
`git clone https://github.com/cosmic-horizon/coho.git`  
`cd ~/coho`  
`starport chain build`  

# Initiate CoHo Instance  
`cd ~`  
`cohod init <your-moniker> --chain-id darkmatter-1`  

# Create or Import Validator Key  
Create  
`cohod keys add <your validator-key-name>`  

Import  
`cohod keys add <your validator-key-name> --recover`  

# Download the genesis file  
`curl -s https://raw.githubusercontent.com/cosmic-horizon/testnets/main/darkmatter-1/genesis.json > $HOME/.coho/config/genesis.json`  

# Prepare config  
`sudo nano .coho/config/app.toml`  

```
minimum-gas-prices = "0ucoho"
```

`sudo nano .coho/config/config.toml`  

```
seeds = "ddf218c8c7c5c63b976309443b9c3a2e860aa2c3@62.171.166.106:26656,4338abf9fdbe143e59119d25310d8187e776df8a@89.58.6.243:26656"
persistent_peers = "4338abf9fdbe143e59119d25310d8187e776df8a@89.58.6.243:26656,038e405c3bc3b7a72b2a8fe9759e4495ac9f7ab0@97.113.198.230:26656,20d436ab002bed85fbf0a1740cdf44d56594d62f@149.28.13.161:26656,ffc0a1443298df007f6caf165b4055f091067b41@173.212.249.116:26656,47dd5dc190bd28ccf91d17609682048dcb20ab67@65.108.11.6:46656,767595068673dfed33c0f95fce77f693fb27438c@173.212.230.119:26656,4177031549e3a53a697d0a0c2137925604c8651a@135.181.212.183:26656,fb14afb3ca33df42932ff9bd15e4662ae3d2e9fb@136.243.110.52:26656,6b2942a2266db223bef9104f59694d74d018f25b@142.132.170.122:26656,9291cebff2bb3781957451f85876a70cc7d386b4@95.179.186.131:26656,8ec8203e97e2d6f83d839b29519ea9298ac0b310@95.217.131.135:26656,0d67b8c164f20b82b055b8d88366b104fd3091f7@144.202.124.47:26656,6f6cac012b1ef57619294029e6843bcfe2eb4f5e@88.99.95.81:26656,a6e95e47d5b9ebc3249db1572cda4932cfca55d1@46.4.98.10:26656,c2526dba0581b4de079a09c25c5407103dad361c@5.9.22.226:26656,4730265f9eda448e3778ecec5a1e76133ff1f51e@65.108.41.38:26656,11105af96d70e92e0b733ee828b1f308df887fb7@65.108.12.222:26626,e9b241c26cb5b36e9fa8d091f68f5fa2594af3c4@144.202.20.79:26656,a5ab40eda3f41f7bec2d3b0a66556c9dc2b51930@144.91.77.189:26656,53fdbd70a669dba972abf4fc37bf940b1e44671d@116.202.209.108:26656,e604bdce5751e7ed5784b0cebfc3a1e0c0da2d23@62.171.166.106:26656,6024aeb1ea4489a029cbcf716343ff15c79c72b8@157.90.179.182:26656,e9870a62b6202ec82d5ef2386981c0f70879aef5@38.242.204.219:26656,1adef7b2370b38b455724c7824aa43e1a1a751eb@34.133.28.120:26656,eb8e911d52881dad82fa6a01b08469fa5a263724@135.181.111.125:26656,d303dce8c99ac40042267684487b8ada294bc7ad@95.217.165.63:26656,7ebfa8e7d8f6f46a967015d193ea81f29b6e42f8@185.185.80.4:26656,086b3c99f153bb6d606316fc0a99cd7fd2c7113f@38.242.200.166:26656,95aace7c559871eb9fd7611f79cfb20185a5023e@173.249.18.178:26656,404dc41da03ff6b833a90009a16857e104222de5@95.216.172.160:26656"
```  

# Use pruning data
https://github.com/erknfe/Cosmos-Scripts/blob/main/pruning-data.md

# Start the Node  
```
sudo tee /etc/systemd/system/cohod.service > /dev/null <<EOF
[Unit]
Description=Cohod Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cohod) start
Restart=always
RestartSec=3
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
EOF
```
`sudo systemctl daemon-reload`  
`sudo systemctl enable cohod`  
`sudo systemctl restart cohod`  
`journalctl -u cohod -f`  

# Create Validator  
`curl -s localhost:26657/status | jq .result | jq .sync_info`  
!!! WAIT FOR - `catching_up: false` (means node is completely synced) !!!

Check balance:  
`cohod query bank balances <account_address> --chain-id darkmatter-1`  
!!! IF amount is not null create validator. Othervise find some token from other validators. !!!  

```
cohod tx staking create-validator \
  --amount=1000000ucoho \
  --pubkey=$(cohod tendermint show-validator) \
  --moniker="<node_moniker>" \
  --chain-id=darkmatter-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas-adjustment 1.5 \
  --gas="auto" \
  --from=<key_name>
```

# Withdraw delegator rewards  
`cohod tx distribution withdraw-rewards <validator_address> --commission --from <key_name> --gas auto --fees=0ucoho --chain-id darkmatter-1 -y`  

# Check balance  
`cohod query bank balances <account_address> --chain-id darkmatter-1 | awk '/amount:/ {print}' | tr -cd [:digit:]`

# Delegate to a validator  
`cohod tx staking delegate <validator_address> ${compounding_coin}ucoho --chain-id darkmatter-1 --gas auto --fees=0ucoho --from <key_name> --yes`  

# Send coho to another account address  
`cohod tx bank send <key_name> <account_address> <amount>000000ucoho --fees=0ucoho --chain-id darkmatter-1`
