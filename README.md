# Initia
# PREPARE
```
sudo apt update && \
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```
## 1.GO
```
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```
## 2.Set version
```
git clone https://github.com/initia-labs/initia.git
cd initia
git checkout v0.2.15
make install
```
## UPDATE
```
rm -rf initia
git clone https://github.com/initia-labs/initia
git checkout v0.2... (lastest)
Make install
sudo systemctl restart initiad && sudo journalctl -fu initiad -o cat
```
## 3.SYSTEM
```
sudo tee /etc/systemd/system/initiad.service > /dev/null << EOF
[Unit]
Description=initia node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which initiad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
## 4. ADDRBOOK
```
wget -O $HOME/.initia/config/addrbook.json https://rpc-initia-testnet.trusted-point.com/addrbook.json
```
## 5. PEER
```
PEERS="e3ac92ce5b790c76ce07c5fa3b257d83a517f2f6@178.18.251.146:30656,2692225700832eb9b46c7b3fc6e4dea2ec044a78@34.126.156.141:26656,2a574706e4a1eba0e5e46733c232849778faf93b@84.247.137.184:53456,40d3f977d97d3c02bd5835070cc139f289e774da@168.119.10.134:26313,1f6633bc18eb06b6c0cab97d72c585a6d7a207bc@65.109.59.22:25756,4a988797d8d8473888640b76d7d238b86ce84a2c@23.158.24.168:26656,e3679e68616b2cd66908c460d0371ac3ed7795aa@176.34.17.102:26656,d2a8a00cd5c4431deb899bc39a057b8d8695be9e@138.201.37.195:53456,329227cf8632240914511faa9b43050a34aa863e@43.131.13.84:26656,517c8e70f2a20b8a3179a30fe6eb3ad80c407c07@37.60.231.212:26656,07632ab562028c3394ee8e78823069bfc8de7b4c@37.27.52.25:19656,028999a1696b45863ff84df12ebf2aebc5d40c2d@37.27.48.77:26656,3c44f7dbb473fee6d6e5471f22fa8d8095bd3969@185.219.142.137:53456,8db320e665dbe123af20c4a5c667a17dc146f4d0@51.75.144.149:26656,c424044f3249e73c050a7b45eb6561b52d0db456@158.220.124.183:53456,767fdcfdb0998209834b929c59a2b57d474cc496@207.148.114.112:26656,edcc2c7098c42ee348e50ac2242ff897f51405e9@65.109.34.205:36656,140c332230ac19f118e5882deaf00906a1dba467@185.219.142.119:53456,4eb031b59bd0210481390eefc656c916d47e7872@37.60.248.151:53456,ff9dbc6bb53227ef94dc75ab1ddcaeb2404e1b0b@178.170.47.171:26656,ffb9874da3e0ead65ad62ac2b569122f085c0774@149.28.134.228:26656" && \
SEEDS="2eaa272622d1ba6796100ab39f58c75d458b9dbc@34.142.181.82:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@testnet-seeds.whispernode.com:25756" && \
sed -i \
    -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" \
    -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" \
    "$HOME/.initia/config/config.toml"
```
## 6. Snapshot
```
sudo systemctl stop initiad
initiad tendermint unsafe-reset-all --home $HOME/.initia --keep-addr-book
wget https://rpc-initia-testnet.trusted-point.com/latest_snapshot.tar.lz4
lz4 -d -c ./latest_snapshot.tar.lz4 | tar -xf - -C $HOME/.initia
sudo systemctl restart initiad && sudo journalctl -u initiad -f -o cat
```
## 7. SET PRICE
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.15uinit,0.01uusdc\"/" $HOME/.initia/config/app.toml
```
## 8. SET PRUNING
```
sed -i \
    -e "s/^pruning *=.*/pruning = \"custom\"/" \
    -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" \
    -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" \
    "$HOME/.initia/config/app.toml"
```
## 9. SET GENESIS
```
initiad init <your_moniker> --chain-id=initiation-1
nano /root/.initia/config/genesis.json
```
## 10. Check Sync
```
initiad status | jq
```
### Check Catch block
```
local_height=$(initiad status | jq -r .sync_info.latest_block_height)
network_height=$(curl -s https://rpc-initia-testnet.trusted-point.com/status | jq -r .result.sync_info.latest_block_height)
blocks_left=$((network_height - local_height))
echo "Your node height: $local_height"
echo "Network height: $network_height"
echo "Blocks left: $blocks_left"
```
### CHECK BALANCE
```
initiad q bank balances $(initiad keys show wallet -a)
```
## 11. CREAT VALIDATORS
```
initiad tx mstaking create-validator \
    --amount="1000000uinit" \
    --pubkey=$(initiad tendermint show-validator) \
    --moniker="YOUR_MONIKER" \
    --identity="YOUR_KEY_BASE" \
    --chain-id="initiation-1" \
    --from="YOUR_WALLET" \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01"
```
## 12. STAKE
```
initiad tx mstaking delegate valoper_here 1000000uinit --from wallet_name_here --gas=2000000 --fees=300000uinit -y
```
### SERVICE
```
sudo systemctl daemon-reload
sudo systemctl enable initiad
sudo systemctl start initiad
sudo journalctl -fu initiad -o cat
```
