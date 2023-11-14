## Half-Life node monitoring Setup

This guide will help you set up and run a Half-Life for monitoring mantrachain validator 

## Prerequisites:

- Git: [https://git-scm.com/](https://git-scm.com/)
- Go: [https://golang.org/](https://golang.org/)
- jq: [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)

## Installation

```markdown

cd $HOME
git clone https://github.com/strangelove-ventures/half-life
cd half-life
wget -cO - https://raw.githubusercontent.com/Dedenwrg/half-life/main/config.yaml.example > config.yaml
MONIKER=$(mantrachaind status | jq -r .NodeInfo.moniker)
RPCADDRESS=$(mantrachaind status | jq -r .NodeInfo.other.rpc_address)
CONSENSUSADDRESS=$(mantrachaind tendermint show-address)
sed -i "s/DISCORD_WEBHOOK_ID/xxxx/" $HOME/half-life/config.yaml
sed -i "s/DISCORD_WEBHOOK_TOKEN/xxxx/" $HOME/half-life/config.yaml
sed -i "s/monikername/$MONIKER/" $HOME/half-life/config.yaml
sed -i "s#tcp://localhost:26657#$RPCADDRESS#" $HOME/half-life/config.yaml
sed -i "s/mantravalcons1/$CONSENSUSADDRESS/" $HOME/half-life/config.yaml
go install
```

## Discord ID Configuration

Every Discord account has a unique Discord ID. To get your Discord ID:

1. Go to settings (cog in bottom left) > Advanced.
2. Toggle on developer mode.
3. Right-click on your own user and click "Copy ID".

Replace `DISCORD_USER_ID` in `$HOME/half-life/config.yaml` with your own Discord ID (leave the brackets).

## Discord Webhook Configuration (Ask to Discord server owner) 

```markdown
sed -i "s/DISCORD_WEBHOOK_ID/xxxx/" $HOME/half-life/config.yaml
sed -i "s/DISCORD_WEBHOOK_TOKEN/xxxx/" $HOME/half-life/config.yaml
```

Replace "xxxx" in both lines with your actual Discord Webhook ID and Discord Webhook Token.

## Systemd Service Setup

```bash
sudo tee /etc/systemd/system/halflife.service << EOF
[Unit]
Description=Halflife
After=network.target
[Service]
Type=simple
Restart=always
RestartSec=5
TimeoutSec=180
User=$(whoami)
WorkingDirectory=$HOME/half-life
ExecStart=$(which halflife) monitor
LimitNOFILE=infinity
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

## Start and Monitor

```bash
systemctl daemon-reload
systemctl enable halflife
systemctl start halflife

journalctl -u halflife -f
```
