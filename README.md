# Cascadia-Node
![2023-05-29_12-06-57](https://github.com/Stefpoker/Cascadia-Node/assets/124903755/5bb16034-d4ab-45d1-bef9-860cad5156cb)


Cascadia is the world's first neocybernetic blockchain. L-1 blockchain, a cosmovork with an unusual approach to block generation and a consensus algorithm.

**Equipment requirements:**

To run Defund, it is recommended to meet the following requirements:

* 4 or more physical processor cores
* The storage capacity on a solid-state disk is at least 200 GB
* The amount of RAM is at least 8 GB
* Network bandwidth of at least 100 Mbit/s

**You need to install:**

```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

**We are going to use GO version 1.20.2, if you already have GO installed, you can skip this**

```
ver="1.20.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

**further**

```
cd $HOME
rm -rf cascadia
git clone https://github.com/CascadiaFoundation/cascadia.git
cd cascadia
git checkout v0.1.1
make build
mkdir -p $HOME/.cascadiad/cosmovisor/genesis/bin
mv cascadiad $HOME/.cascadiad/cosmovisor/genesis/bin/
sudo ln -s $HOME/.cascadiad/cosmovisor/genesis $HOME/.cascadiad/cosmovisor/current
sudo ln -s $HOME/.cascadiad/cosmovisor/current/bin/cascadiad /usr/local/bin/cascadiad
```

**Make sure you have successfully installed cascadia**

```
cascadiad version
```

**Install a cosmovisor**

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
```

**customization**

```
cascadiad config chain-id cascadia_6102-1
cascadiad config keyring-backend test
```

**Initialize the node
Please change <YOUR_MONIKER> your own nickname.**

```
cascadiad init <YOUR_MONIKER> --chain-id cascadia_6102-1
```

**Download the Genesis file and an additional book**

**Genesis**

```
curl -Ls https://snapshots.indonode.net/cascadia-t/genesis.json > $HOME/.cascadiad/config/genesis.json
```

**book**

```
curl -Ls https://snapshots.indonode.net/cascadia-t/addrbook.json > $HOME/.cascadiad/config/addrbook.json
```

**Configuring initial and Peer nodes**

```
PEERS="$(curl -sS https://rpc.cascadiad-t.indonode.net/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}' | sed -z 's|\n|,|g;s|.$||')"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.cascadiad/config/config.toml
```

**Set minimum gas prices**

```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"7aCC\"|" $HOME/.cascadiad/config/app.toml
```

**cropping the configuration**

```
PRUNING="custom"
PRUNING_KEEP_RECENT="100"
PRUNING_INTERVAL="19"
sed -i -e "s/^pruning *=.*/pruning = \"$PRUNING\"/" $HOME/.cascadiad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$PRUNING_KEEP_RECENT\"/" $HOME/.cascadiad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$PRUNING_INTERVAL\"/" $HOME/.cascadiad/config/app.toml
```

**create a service file and start the node**

Create a service file Create a cascadiad.service file in the /etc/systemd/system folder with the following code. You can change USER to your Linux username. To complete this step, you will need sudo privilege.

```
sudo tee /etc/systemd/system/cascadiad.service > /dev/null << EOF
[Unit]
Description=cascadia-tesnet node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.cascadiad"
Environment="DAEMON_NAME=cascadiad"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable cascadiad
```

**Download the latest snapshot version**

```
curl -L https://snapshots.indonode.net/cascadia-t/cascadia-snapshot.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.cascadiad
[[ -f $HOME/.cascadiad/data/upgrade-info.json ]] && cp $HOME/.cascadiad/data/upgrade-info.json $HOME/.cascadiad/cosmovisor/genesis/upgrade-info.json
```

**Turn on the service and start the node**

```
sudo systemctl restart cascadiad
sudo journalctl -fu cascadiad -o cat
```
