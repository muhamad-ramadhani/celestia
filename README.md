<p style="font-size:14px" align="right">
<a href="https://t.me/PemulungAirdropID" target="_blank">Join our telegram <img src="https://user-images.githubusercontent.com/72949170/194228482-0f875615-e155-4b12-8716-8111addd6cba.jpg" width="30"/></a>
</p>

<p align="center">
  <img width="60%" height="auto" src="https://user-images.githubusercontent.com/72949170/203788902-f9ec1f48-a425-49cf-a981-d3678df34104.png">
</p>

# CELESTIA TESTNET | MAMAKI

Official documentation:
> https://docs.celestia.org/nodes/overview

Explorer:
> https://celestia.explorers.guru

Manual guides:
- [Run Validator and Bridge Node on same machine](https://github.com/kj89/testnet_manuals/blob/main/celestia/manual_install.md)
- [Run Bridge Node seperately](https://github.com/kj89/testnet_manuals/blob/main/celestia/manual_bridge.md)
- [Run Light Node seperately](https://github.com/kj89/testnet_manuals/blob/main/celestia/manual_light.md)
- [Run Full Node seperately](https://github.com/kj89/testnet_manuals/blob/main/celestia/manual_full.md)


|  Komponen |  Persyaratan Rekomendasi |
| ------------ | ------------ |
| CPU  | Quad-Core  |
| RAM | 8 GB  |
| Penyimpanan  | 250 GB SSD Storage |
| Koneksi | 100 Mbit/s |

## 1. Setup Otomotasi
```
wget -O celestia.sh https://raw.githubusercontent.com/muhamad-ramadhani/celestia/main/celestia.sh && chmod +x celestia.sh && ./celestia.sh
```

## 2. Post installation
load variables ke system
```
source $HOME/.bash_profile
```

Selanjutnya kalian harus memastikan validator kalian menyinkronkan blok. kalian dapat menggunakan perintah di bawah ini untuk memeriksa status sinkronisasi
```
celestia-appd status 2>&1 | jq .SyncInfo
```

### (OPTIONAL) Disable and cleanup indexing
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.celestia-app/config/config.toml
sudo systemctl restart celestia-appd
sleep 3
sudo rm -rf $HOME/.celestia-app/data/tx_index.db
```

### (OPTIONAL) Use Quick Sync by restoring data from snapshot
```
systemctl stop celestia-appd
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app
cd $HOME
rm -rf ~/.celestia-app/data
mkdir -p ~/.celestia-app/data
SNAP_NAME=$(curl -s https://snaps.qubelabs.io/celestia/ | egrep -o ">mamaki.*tar" | tr -d ">")
wget -O - https://snaps.qubelabs.io/celestia/${SNAP_NAME} | tar xf - -C ~/.celestia-app/data/
systemctl restart celestia-appd && journalctl -fu celestia-appd -o cat
```

## 3. Create wallet

```
celestia-appd keys add $WALLET
```
Jangan lupa simpan data2nya

(OPTIONAL) Untuk memulihkan dompet kalian menggunakan seed phrase
```
celestia-appd keys add $WALLET --recover
```

Untuk mendapatkan daftar dompet saat ini
```
celestia-appd keys list
```

## 4. Save wallet info
Masukan wallet dan valoper address ke variables 
```
CELESTIA_WALLET_ADDRESS=$(celestia-appd keys show $WALLET -a)
CELESTIA_VALOPER_ADDRESS=$(celestia-appd keys show $WALLET --bech val -a)
echo 'export CELESTIA_WALLET_ADDRESS='${CELESTIA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export CELESTIA_VALOPER_ADDRESS='${CELESTIA_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## 5. Req Faucet
Untuk membuat validator terlebih dahulu, kalian perlu mendanai dompet kalian dengan token testnet.
Untuk mengisi ulang dompet kalian, join [celestia discord server](https://discord.gg/neBFH8Se) dan masuk ke:
- **#faucet** untuk request test tokens

request faucet via command:
```
$request <YOUR_WALLET_ADDRESS>
```

check wallet balance:
```
$balance <YOUR_WALLET_ADDRESS>
```

## 6. Create validator
Sebelum membuat validator, pastikan kalian memiliki setidaknya 1 tia (1 tia sama dengan 1.000.000 utia) dan node kalian telah disinkronkan

untuk check wallet balance kalian:
```
celestia-appd query bank balances $CELESTIA_WALLET_ADDRESS
```
> Jika dompet kalian tidak menunjukkan saldo apa pun, kemungkinan node kalian masih disinkronkan. Harap tunggu hingga selesai untuk menyinkronkan, lalu lanjutkan

Untuk membuat validator kalian, jalankan perintah di bawah ini
```
celestia-appd tx staking create-validator \
  --amount 1000000utia \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(celestia-appd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $CELESTIA_CHAIN_ID
```

## DONE, UNTUK UPDATE SELANJUTNYA KALIAN BISA PANTAU <a href="https://t.me/PemulungAirdropID" target="_blank">CHANNEL KITA </a>

## Keamanan

Mulailah dengan memeriksa status ufw.
```
sudo ufw status
```

Menyetel default untuk mengizinkan koneksi keluar, menolak semua masuk kecuali ssh dan 26656. Batasi upaya login SSH
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow ${CELESTIA_PORT}656,${CELESTIA_PORT}660/tcp
sudo ufw enable
```


## Perintah Berguna
### Service management
Check logs
```
journalctl -fu celestia-appd -o cat
```

Start service
```
sudo systemctl start celestia-appd
```

Stop service
```
sudo systemctl stop celestia-appd
```

Restart service
```
sudo systemctl restart celestia-appd
```

### Node info
Synchronization info
```
celestia-appd status 2>&1 | jq .SyncInfo
```

Validator info
```
celestia-appd status 2>&1 | jq .ValidatorInfo
```

Node info
```
celestia-appd status 2>&1 | jq .NodeInfo
```

Show node id
```
celestia-appd tendermint show-node-id
```

### Wallet operations
List of wallets
```
celestia-appd keys list
```

Recover wallet
```
celestia-appd keys add $WALLET --recover
```

Delete wallet
```
celestia-appd keys delete $WALLET
```

Get wallet balance
```
celestia-appd query bank balances $CELESTIA_WALLET_ADDRESS
```

Transfer funds
```
celestia-appd tx bank send $CELESTIA_WALLET_ADDRESS <TO_CELESTIA_WALLET_ADDRESS> 10000000utia
```

### Voting
```
celestia-appd tx gov vote 1 yes --from $WALLET --chain-id=$CELESTIA_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
celestia-appd tx staking delegate $CELESTIA_VALOPER_ADDRESS 10000000utia --from=$WALLET --chain-id=$CELESTIA_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
celestia-appd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000utia --from=$WALLET --chain-id=$CELESTIA_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
celestia-appd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$CELESTIA_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
celestia-appd tx distribution withdraw-rewards $CELESTIA_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$CELESTIA_CHAIN_ID
```

### Validator management
Edit validator
```
celestia-appd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$CELESTIA_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```
celestia-appd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CELESTIA_CHAIN_ID \
  --gas=auto
```

### Delete node
This commands will completely remove node from server. Use at your own risk!
```
sudo systemctl stop celestia-appd
sudo systemctl disable celestia-appd
sudo rm /etc/systemd/system/celestia* -rf
sudo rm $(which celestia-appd) -rf
sudo rm $HOME/.celestia-app* -rf
sudo rm $HOME/celestia -rf
sed -i '/CELESTIA_/d' ~/.bash_profile
```
