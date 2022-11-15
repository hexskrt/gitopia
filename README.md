<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/38981255/201691739-df61f26e-5c05-46b5-8ce6-0981a6615d60.PNG">
</p>

#  gitopia node setup for testnet — gitopia-janus-testnet-2

Dokumentasi resmi:
> - [Official Documentation](https://docs.gitopia.com/installation/index.html)

Explorer :
> - [Explorer Checker](https://gitopia.explorers.guru/)

##  Hardware Requirements
Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

###  Minimum Hardware Requirements
 - 4x CPU; the faster clock speed the better
 - RAM 8GB
 - Storage 100GB (SSD atau NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

###  Recommended Hardware Requirements
 - 8x CPU; the faster clock speed the better
 - RAM 64GB
 - Storage 1TB (SSD atau NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

##  Set up your gitopia fullnode
###  Automatic Installation
You can setup your gitopia fullnode in few minutes by using automated script below. It will prompt you to input your validator node name!
```
wget -O gitopia.sh https://raw.githubusercontent.com/hexskrt/gitopia/main/gitopia.sh && chmod +x gitopia.sh && ./gitopia.sh
```

##  Post installation

When installation is finished please load variables into system
```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
gitopiad status 2>&1 | jq .SyncInfo
```
### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
gitopiad keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
gitopiad keys add $WALLET --recover
```

To get current list of wallets
```
gitopiad keys list
```

### Save wallet info
Add wallet and valoper address into variables 
```
GITOPIA_WALLET_ADDRESS=$(gitopiad keys show $WALLET -a)
GITOPIA_VALOPER_ADDRESS=$(gitopiad keys show $WALLET --bech val -a)
echo 'export GITOPIA_WALLET_ADDRESS='${GITOPIA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export GITOPIA_VALOPER_ADDRESS='${GITOPIA_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Fund your wallet

- Import Your Information Wallet To Keplr Wallet
- Open Web : https://gitopia.com/
- Connect Wallet 
- Get 10 Tlore Done

### State Sync

You can state sync your node in minutes by running commands below

Stop Service & Reset

```
sudo systemctl stop gitopiad
cp $HOME/.gitopia/data/priv_validator_state.json $HOME/.gitopia/priv_validator_state.json.backup
rm -rf $HOME/.gitopia/data
```
Download New Snapshot (File Size 880 MB)
```
curl -L https://snapshots.kjnodes.com/gitopia-testnet/snapshot_latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.gitopia
mv $HOME/.gitopia/priv_validator_state.json.backup $HOME/.gitopia/data/priv_validator_state.json
```
Restart your service and Check Block
```
sudo systemctl start gitopiad && journalctl -u gitopiad -f --no-hostname -o cat
```

### Create validator
Before creating validator please make sure that you have at least 1 tlore (1 tlore is equal to 1000000 utlore) and your node is synchronized

To check your wallet balance:
```
gitopiad query bank balances $GITOPIA_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 
To create your validator run command below
```
gitopiad tx staking create-validator \
  --amount 1000000utlore \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(gitopiad tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $GITOPIA_CHAIN_ID
```
### Check your validator key
```
[[ $(gitopiad q staking validator $GITOPIA_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(gitopiad status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

### Get list of validators
```
gitopiad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${GITOPIA_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Usefull commands
### Service management
Check logs
```
journalctl -fu gitopiad -o cat
```

Start service
```
sudo systemctl start gitopiad
```

Stop service
```
sudo systemctl stop gitopiad
```

Restart service
```
sudo systemctl restart gitopiad
```

### Node info
Synchronization info
```
gitopiad status 2>&1 | jq .SyncInfo
```

Validator info
```
gitopiad status 2>&1 | jq .ValidatorInfo
```

Node info
```
gitopiad status 2>&1 | jq .NodeInfo
```

Show node id
```
gitopiad tendermint show-node-id
```

### Wallet operations
List of wallets
```
gitopiad keys list
```

Recover wallet
```
gitopiad keys add $WALLET --recover
```

Delete wallet
```
gitopiad keys delete $WALLET
```

Get wallet balance
```
gitopiad query bank balances $GITOPIA_WALLET_ADDRESS
```

Transfer funds
```
gitopiad tx bank send $GITOPIA_WALLET_ADDRESS <TO_GITOPIA_WALLET_ADDRESS> 10000000utlore
```

### Voting
```
gitopiad tx gov vote 1 yes --from $WALLET --chain-id=$GITOPIA_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
gitopiad tx staking delegate $GITOPIA_VALOPER_ADDRESS 10000000utlore --from=$WALLET --chain-id=$GITOPIA_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
gitopiad tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000utlore --from=$WALLET --chain-id=$GITOPIA_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
gitopiad tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$GITOPIA_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
gitopiad tx distribution withdraw-rewards $GITOPIA_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$GITOPIA_CHAIN_ID
```

### Validator management
Edit validator
```
gitopiad tx staking edit-validator \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$GITOPIA_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```
gitopiad tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$GITOPIA_CHAIN_ID \
  --gas=auto
```

### Delete Node
This commands will completely remove node from server. Use at your own risk!
```
sudo systemctl stop gitopiad
sudo systemctl disable gitopiad
sudo rm /etc/systemd/system/gitopia* -rf
sudo rm $(which gitopiad) -rf
sudo rm $HOME/.gitopia* -rf
sudo rm $HOME/gitopia -rf
sed -i '/GITOPIA_/d' ~/.bash_profile
```
