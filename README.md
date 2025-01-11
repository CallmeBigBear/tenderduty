# TenderDuty v2

[![Go Reference](https://pkg.go.dev/badge/github.com/blockpane/tenderduty.svg)](https://pkg.go.dev/github.com/blockpane/tenderduty)
[![Gosec](https://github.com/blockpane/tenderduty/workflows/Gosec/badge.svg)](https://github.com/blockpane/tenderduty/actions?query=workflow%3AGosec)
[![CodeQL](https://github.com/blockpane/tenderduty/workflows/CodeQL/badge.svg)](https://github.com/blockpane/tenderduty/actions?query=workflow%3ACodeQL)

Tenderduty is a comprehensive monitoring tool for Tendermint chains. Its primary function is to alert a validator if they are missing blocks, and has many other features.

v2 is complete rewrite of the original tenderduty graciously sponsored by the [Osmosis Grants Program](https://grants.osmosis.zone/). This new version adds a web dashboard, prometheus exporter, telegram and discord notifications, multi-chain support, more granular alerting, and more types of alerts.

![dashboard screenshot](docs/dash.png)

## Documentation

The [documentation](docs/README.md) is a work-in-progress.

## Runtime options:

```
$ tenderduty -h
Usage of tenderduty:
  -example-config
    	print the an example config.yml and exit
  -f string
    	configuration file to use (default "config.yml")
  -state string
    	file for storing state between restarts (default ".tenderduty-state.json")
  -cc string
    	directory containing additional chain specific configurations (default "chains.d")
```

## Installing

Detailed installation info is in the [installation doc.](docs/install.md)

30 second quickstart if you already have Docker installed:

```
mkdir tenderduty && cd tenderduty
docker run --rm ghcr.io/blockpane/tenderduty:latest -example-config >config.yml
# edit config.yml and add chains, notification methods etc.
docker run -d --name tenderduty -p "8888:8888" -p "28686:28686" --restart unless-stopped -v $(pwd)/config.yml:/var/lib/tenderduty/config.yml ghcr.io/blockpane/tenderduty:latest
docker logs -f --tail 20 tenderduty
```


## Split Configuration

For validators with many chains, chain specific configuration may be split into additional files and placed into the directory "chains.d".

This directory can be changed with the -cc option

The user friendly chain label will be taken from the name of the file.  

For example:

```
chains.d/Juno.yml -> Juno
chains.d/Lum Network.yml -> Lum Network
```

Configuration inside chains.d/Network.yml will be the YAML contents without the chain label.

For example start directly with:

```
chain_id: demo-1
    valoper_address: demovaloper...
```

##################################################################################


INSTALL TENDERDUTY LIKA A SERVICE
https://teletype.in/@itrocket/bdJAHvC_q8h

#Install GO
```
sudo rm -rf /usr/local/go && \
curl -L https://go.dev/dl/go1.23.2.linux-arm64.tar.gz | sudo tar -xzf - -C /usr/local && \
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
#Install tenderduty
```
cd $HOME
rm -rf tenderduty
git clone https://github.com/blockpane/tenderduty
cd tenderduty
go install
cp example-config.yml config.yml
```
#Edit config.yml
For simple monitoring without notifications, just change these in the config:

Osmosys to <PROJECT_NAME>
chain_id: osmosis-1 to chain_id: <YOUR_CHAIN_ID> 
valoper_address: osmovaloper1xxxxxxx... to valoper_address: <YOUR_VALOPER_ADDRESS>
url: tcp://localhost:26657 TO url: tcp://localhost:<YOUR_NODE_RPC_PORT>

#Optional 
If you want to add another node monitoring, you can dublicate this section on conf.yml file
```
chains:

  # The user-friendly name that will be used for labels. Highly suggest wrapping in quotes.
  "Osmosis":

    ...

      - url: https://some-other-node:443
        alert_if_down: no
```
#Create service file and start tenderduty
```
sudo tee /etc/systemd/system/tenderdutyd.service << EOF
[Unit]
Description=Tenderduty
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=5
TimeoutSec=180

User=$USER
WorkingDirectory=$HOME/tenderduty
ExecStart=$(which tenderduty)

# there may be a large number of network connections if a lot of chains
LimitNOFILE=infinity

# extra process isolation
NoNewPrivileges=true
ProtectSystem=strict
RestrictSUIDSGID=true
LockPersonality=true
PrivateUsers=true
PrivateDevices=true
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable tenderdutyd
sudo systemctl start tenderdutyd
```
#Check logs
```
sudo journalctl -fu tenderdutyd
```
#Open port of tenderduty (default 8888)

#You can open dashboard on web browser by using tenderduty port and your server IP 
http://<YOUR_SERVER_IP>:<PORT>


## Configure Telegram alerting
Open telegram and find @BotFather 

Create telegram bot via , customize it and @BotFatherget bot API token (how_to).
Create the group:  . Customize them, add the bot in your chat and alarmget chats IDs (how_to).
Open config.yml file 
```
change enabled: no to enabled: yes
api_key: '5555555555:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA' to  api_key: '<YOUR_BOT_API_KEY>'
channel: "-666666666" to channel: "<YOUR_GROUP_CHAT_ID>"
```
#Restart tenderduty
```
sudo systemctl restart tenderdutyd && sudo journalctl -fu tenderdutyd
```
Done!
