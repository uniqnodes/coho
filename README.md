# Install Golang (go)  
https://github.com/erknfe/go-lang  

# Install QWOYN  
`git clone https://github.com/cosmic-horizon/QWOYN`  
`cd QWOYN`  
`git checkout v3.1.0`  
`make install`  

# Initiate Qwoyn Instance  
`cd ~`  
`qwoynd init <moniker> --chain-id higgs-boson-3`  

# Create or Import Validator Key  
Create  
`qwoynd keys add <your_validator_key_name>`  

Import  
`qwoynd keys add <your_validator_key_name> --recover`  

# Download the genesis file  
`curl http://45.32.76.26:26657/genesis | jq .result.genesis > ~/.qwoynd/config/genesis.json`  

# Update Peers  
```
PERSISTENT_PEERS="a163f77e71214acaf2e0bcb762dcd5938e119f0d@45.32.76.26:26656"  
sed -i '/persistent_peers =/c\persistent_peers = "'"$PERSISTENT_PEERS"'"' ~/.qwoynd/config/config.toml
```
# Set minimum-gas-prices  
`sed -i '/minimum-gas-prices =/c\minimum-gas-prices = "0.025uqwoyn"' ~/.qwoynd/config/app.toml`

# Start the Node  
```
sudo tee /etc/systemd/system/qwoynd.service > /dev/null <<EOF
[Unit]
Description=Qwoyn Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which qwoynd) start
Restart=always
RestartSec=3
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
EOF
```
`sudo systemctl daemon-reload`  
`sudo systemctl enable qwoynd`  
`sudo systemctl restart qwoynd`  
`journalctl -u qwoynd -f`  

# Create Validator  
`curl -s localhost:26657/status | jq .result | jq .sync_info`  
!!! WAIT FOR - `catching_up: false` (means node is completely synced) !!!

# Get faucet
`curl -X POST -d '{"address": "ENTER-YOUR-QWOYN-ADDRESS"}' http://45.32.76.26:8000`

# Check balance:  
`qwoynd query bank balances <account_address> --chain-id higgs-boson-3`  

```
qwoynd tx staking create-validator \
  --amount=<qwoyn_amount> \
  --pubkey=$(qwoynd tendermint show-validator) \
  --moniker="<node_moniker>" \
  --chain-id="higgs-boson-3" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas="400000" \
  --from=<key_name> \
  --fees="1000uqwoyn"
```

# Withdraw delegator rewards  
`qwoynd tx distribution withdraw-rewards <validator_address> --commission --from <key_name> --gas auto --fees=1000uqwoyn --chain-id higgs-boson-3 -y`  

# Check balance  
`qwoynd query bank balances <account_address> --chain-id higgs-boson-3 | awk '/amount:/ {print}' | tr -cd [:digit:]`

# Delegate to a validator  
`qwoynd tx staking delegate <validator_address> <amount>uqwoyn --chain-id higgs-boson-3 --gas auto --fees=1000uqwoyn --from <key_name> -y`  

# Redelegate  
`qwoynd tx staking redelegate <from_validator_address> <to_validator_address> <amount>uqwoyn --from <key_name> --gas auto --fees=1000uqwoyn --chain-id higgs-boson-3`  

# Send qwoyn to another account address  
`qwoynd tx bank send <key_name> <account_address> <amount>uqwoyn --fees=1000uqwoyn --chain-id higgs-boson-3`
