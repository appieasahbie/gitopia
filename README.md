# Installing node on gitopia


![VFVF](https://user-images.githubusercontent.com/108979536/201230381-9d344d89-93e4-496e-bafc-ee1852f97392.png)



### Code Collaboration

for Web3
Gitopia is the next-generation Code Collaboration Platform fuelled by a decentralized network and interactive token economy. It is designed to optimize the open-source software development process through collaboration, transparency, and incentivization.




# Hardware Requirements

 ### Minimum Hardware Requirements
   + 4x CPUs; the faster clock speed the better
   + 8GB RAM
   + 100GB of storage (SSD or NVME)
   
### Recommended Hardware Requirements
   + 8x CPUs; the faster clock speed the better
   + 64GB RAM
   + 1TB of storage (SSD or NVME)
   
   
# 1-ɪɴᴛᴀʟʟᴀᴛɪᴏɴ ᴏɴᴇ ᴛɪᴍᴇ (ꜱᴄʀɪᴘᴛ ᴀᴜᴛᴏᴍᴀᴛɪᴄ ɪɴꜱᴛᴀʟʟᴀᴛɪᴏɴ)

        wget -O gitopia.sh https://raw.githubusercontent.com/appieasahbie/gitopia/main/gitopia.sh && chmod +x gitopia.sh && ./gitopia.sh
        
             
# Post installation

          source $HOME/.bash_profile
          
 +v(Check the status of your node and ctrl + c)
 
         gitopiad status 2>&1 | jq .SyncInfo
         
         
 ### open ports and active the firewall
 
         sudo ufw default allow outgoing
         sudo ufw default deny incoming
         sudo ufw allow ssh/tcp
         sudo ufw limit ssh/tcp
         sudo ufw allow ${GITOPIA_PORT}656,${GITOPIA_PORT}660/tcp
         sudo ufw enable
         
         
### Create wallet

  + (Please save all keys on your notepad)

        gitopiad keys add $WALLET
        
  + To recover your old wallet use this command

        gitopiad keys add $WALLET --recover
        
        
  + show keys

        gitopiad keys list
        
        
 ### Add wallet and valoper address and load variables into the system
 
        GITOPIA_WALLET_ADDRESS=$(gitopiad keys show $WALLET -a)
        GITOPIA_VALOPER_ADDRESS=$(gitopiad keys show $WALLET --bech val -a)
        echo 'export GITOPIA_WALLET_ADDRESS='${GITOPIA_WALLET_ADDRESS} >> $HOME/.bash_profile
        echo 'export GITOPIA_VALOPER_ADDRESS='${GITOPIA_VALOPER_ADDRESS} >> $HOME/.bash_profile
        source $HOME/.bash_profile
    
    
Fund your wallet (to create validator) TB you can also use web faucet [link](https://gitopia.com/home)



 ### Create validator (after recieving of tokens and must important sync is false)

  + replace with your wallet name and with your validator name         
  
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
         
         
 ## check your validator key
 
          [[ $(gitopiad q staking validator $GITOPIA_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(gitopiad status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"

## Get list of validators

          gitopiad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl

### Get currently connected peer list with ids

         curl -sS http://localhost:${GITOPIA_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'


### Delegate stake

         gitopiad tx staking delegate $GITOPIA_VALOPER_ADDRESS 10000000utlore --from=$WALLET --chain-id=$GITOPIA_CHAIN_ID --gas=auto


### Check logs

        journalctl -fu gitopiad -o cat
### Start service

        sudo systemctl start gitopiad
        
### Stop service

        sudo systemctl stop gitopiad
### Restart service

       sudo systemctl restart gitopiad
       
### Synchronization info

       gitopiad status 2>&1 | jq .SyncInfo
       
### Validator info

       gitopiad status 2>&1 | jq .ValidatorInfo
       
### Node info

       gitopiad status 2>&1 | jq .NodeInfo
       
### Show node id

       gitopiad tendermint show-node-id      
       
         
### Edit validator

      gitopiad tx staking edit-validator \
        --moniker=$NODENAME \
        --identity=<your_keybase_id> \
        --website="<your_website>" \
        --details="<your_validator_description>" \
        --chain-id=$GITOPIA_CHAIN_ID \
        --from=$WALLET
  
  
### Unjail validator

      gitopiad tx slashing unjail \
        --broadcast-mode=block \
        --from=$WALLET \
        --chain-id=$GITOPIA_CHAIN_ID \
        --gas=auto
        
        
### Delete node

      sudo systemctl stop gitopiad
      sudo systemctl disable gitopiad
      sudo rm /etc/systemd/system/gitopia* -rf
      sudo rm $(which gitopiad) -rf
      sudo rm $HOME/.gitopia* -rf
      sudo rm $HOME/gitopia -rf
      sed -i '/GITOPIA_/d' ~/.bash_profile  
      
      
 [buy me cup of coffee](https://www.paypal.com/paypalme/AbdelAkridi?country.x=NL&locale.x=en_US)     
      
  
