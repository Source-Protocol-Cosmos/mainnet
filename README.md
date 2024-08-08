# mainnet

![c11](https://static.wixstatic.com/media/80368b_b2c7b9f0d8614798bd9df0111903155a~mv2.png/v1/fill/w_624,h_108,al_c,q_85,usm_0.66_1.00_0.01/source%20logo%20final%20hrzn.webp)

### Source Blockchain Set up


If you are reusing your testnet box, you must first remove the old data. Validators from the previous testnet may have usource available to start a new node, use your same wallet:

```bash
sudo systemctl stop sourced && \
cd $HOME && \
rm -rf .source && \
rm -rf source && \
rm -rf $(which sourced)
```


For full Source Chain Documentation click: [HERE](https://docs.sourceprotocol.io/source-chain-documentation/introduction)

### Minimum hardware requirements
8GB RAM
250GB of disk space
1.4 GHz amd64 CPU

#### Install Go

**Prerequisites:** 

**Prepare Server**
```bash
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y
```

**Go** Make sure to have [Golang >=1.19](https://golang.org/).
```bash
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

### Clone Source Chain Repo

```bash
git clone https://github.com/Source-Protocol-Cosmos/source.git
```

### Compile sourced Binary

```bash
cd ~/source
git fetch
git checkout v3.0.3
make build && make install
```


### Initialize the Source directories and create the local genesis file with the correct chain-id:

```bash
sourced init <moniker-name> --chain-id=source-1
```

### Create a local key pair (or add existing key):

```sh
sourced keys add <walletName>
    or
sourced keys add <walletName> --recover
```

### Download Genesis File

```bash
curl -s  https://raw.githubusercontent.com/Source-Protocol-Cosmos/mainnet/master/source-1/genesis.json > ~/.source/config/genesis.json
```

**Genesis sha256**

```bash
sha256sum ~/.source/config/genesis.json
# ba2261082818227073bd8b49717a9781bf5c440c8e34e21ec72fb15806f047cc
```

### Seed nodes to add to config.toml


```bash
nano ~/.source/config/config.toml
```

```
# Comma separated list of nodes to keep persistent connections to persistent_peers = 
"96d63849a529a15f037a28c276ea6e3ac2449695@34.222.1.252:26656,0107ac60e43f3b3d395fea706cb54877a3241d21@35.87.85.162:26656"
```

### Set Minimum Gas Price


```bash
nano ~/.source/config/app.toml
```

```
0.25usource
```

### Start the chain
```bash
sourced start
```

## Setup validator node

```bash
sourced tx staking create-validator \
--amount 1000000000usource \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.20" \
--commission-rate "0.1" \
--min-self-delegation "1" \
--details "validators write bios too" \
--pubkey=$(sourced tendermint show-validator) \
--moniker “<key-name>” \
--chain-id source-1 \
--fees=50000usource \
--from <key-name>
```


### Running in production

**Note: Consider using [Cosmovisor](https://github.com/cosmos/cosmos-sdk/tree/master/cosmovisor) to make your life easier.**

Create a systemd file for your Source service:
```bash
sudo nano /etc/systemd/system/sourced.service
```   
Copy and paste the following and update <YOUR-USERNAME>:
```bash
Description=Source daemon
After=network-online.target

[Service]
User=<YOUR_USERNAME>
ExecStart=/home/<YOUR-USERNAME>/go/bin/sourced start --home /home/<YOUR-USERNAME>/.source
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```
This assumes $HOME/.source to be your directory for config and data. Your actual directory locations may vary.

Enable and start the new service:
```bash
sudo systemctl enable sourced
```
```bash   
sudo systemctl start sourced
```   
Check status:
```bash
sourced status
```
Check logs:
```bash
journalctl -u sourced -f
```   
