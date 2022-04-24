# Install Golang (go)  
https://github.com/erknfe/go-lang  

# Install Starport  
`curl https://get.starport.network/starport | bash`  
`sudo mv starport /usr/local/bin/`  

# Install CoHo  
`git clone https://github.com/cosmic-horizon/coho.git`  
`cd ~/coho`  
`git checkout v0.1`  
`starport chain build`  

# Initiate CoHo Instance  
`cd ~`  
`cohod init <your_moniker> --chain-id darkenergy-1`  

# Create or Import Validator Key  
Create  
`cohod keys add <your_validator_key_name>`  

Import  
`cohod keys add <your_validator_key_name> --recover`  

# Download the genesis file  
`curl -s https://raw.githubusercontent.com/cosmic-horizon/testnets/main/darkenergy-1/genesis.json > $HOME/.coho/config/genesis.json`  

# Prepare config  
`sudo nano .coho/config/app.toml`  

```
minimum-gas-prices = "0ucoho"
```

`sudo nano .coho/config/config.toml`  

```
seeds = "a06e58e39d4a471d00d2e5d58233089c64fa5bb8@149.28.70.87:26657"
persistent_peers = ""
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
`cohod query bank balances <account_address> --chain-id darkenergy-1`  
!!! IF amount is not null create validator. Othervise find some token from other validators. !!!  

```
cohod tx staking create-validator \
  --amount=1000000ucoho \
  --pubkey=$(cohod tendermint show-validator) \
  --moniker="<node_moniker>" \
  --chain-id=darkenergy-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas-adjustment 1.5 \
  --gas="auto" \
  --from=<key_name>
```

# Withdraw delegator rewards  
`cohod tx distribution withdraw-rewards <validator_address> --commission --from <key_name> --gas auto --fees=0ucoho --chain-id darkenergy-1 -y`  

# Check balance  
`cohod query bank balances <account_address> --chain-id darkenergy-1 | awk '/amount:/ {print}' | tr -cd [:digit:]`

# Delegate to a validator  
`cohod tx staking delegate <validator_address> <amount>ucoho --chain-id darkenergy-1 --gas auto --fees=0ucoho --from <key_name> --yes`  

# Redelegate  
`cohod tx staking redelegate <from_validator_address> <to_validator_address> <amount>ucoho --from <key_name> --gas auto --fees=0ucoho --chain-id darkenergy-1`  

# Send coho to another account address  
`cohod tx bank send <key_name> <account_address> <amount>ucoho --fees=0ucoho --chain-id darkenergy-1`
