# Setup MANTRA Chain Validator

## Create VM

1. Spin up a new Ubuntu 20.04 virtual machine.
	- 200 GB disk
	- 1-2 vCPU
	- 128 GB RAM
2. SSH into machine and update OS
	- `sudo apt-get update`
3. Create a `/bin` in $HOME directory
	- `mkdir bin`
 	- `echo "export PATH=\$PATH:/home/mantra/bin" >> ~/.bashrc` 
	- `source ~/.bashrc` - to make sure the new `/bin` folder is added to `$PATH`.
	- `cd bin`
4.  Get the MANTRA Chain Binary from MANTRA GitHub repo
	- `wget https://github.com/MANTRA-Finance/public/raw/main/mantrachain-testnet/mantrachaind-linux-amd64.zip`
	- `sudo apt install unzip`
	- `unzip mantrachaind-linux-amd64.zip`
5. Install CosmWasm Library
	- `sudo wget -P /usr/lib https://github.com/CosmWasm/wasmvm/releases/download/v1.3.0/libwasmvm.x86_64.so`
5. Initialise `mantrachaind`
	- `mantrachaind init <your-moniker> --chain-id mantrachain-1`
		- where `<your-moniker>`is the moniker and can be any valid string name (e.g. `validator-mantrachain-test-01`),
		- and `mantrachain-1` is the Chain ID and must be exactly as specified.
6. Generate keys and add them to Validator
	- `mantrachaind config keyring-backend file`
	- `mantrachaind keys add validator-mchain-test-node-keys`
	- (alternatively `mantrachaind keys add validator-mchain-test-node-keys --recover` to use existing mnemoic).
	- The above command will prompt for a password - chose something secure! e.g. `Password-01!`
	- The above command will produce output like this:

		```
		- address: mantra1q55nrzygas0nespfu8mwt2yntq8gxll3kyug82
		  name: validator-mchain-test-node-keys
		  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"A/YebpAX8AqUNcNXcqIy53fJo8BGFCSaQA5A0XQWMlCG"}'
		  type: local
		
		
		**Important** write this mnemonic phrase in a safe place.
		It is the only way to recover your account if you ever forget your password.
		
		expect kid unfair uniform calm debris meadow despair vintage arrive walnut vast upset cart step funny truth vault naive note capable spray shine human
		```
	- **KEEP THE MNEMONIC PHRASE IN A SAFE PLACE!**

7. Replace `genesis` file.
	- `cd ~/.mantrachain/config` - change directory to `config`
	- `rm -rf genesis.json` - delete the default `genesis` file
	-  `wget https://github.com/MANTRA-Finance/public/raw/main/mantrachain-testnet/genesis.json` - get the MANTRA Chain Genesis file.
8. Configure connection to TESTNET
	- Edit `config.toml` file (using your favourite text editor)
	- Add the `persistent_peers`.  
	
		```		
		persistent_peers = "<node-id>@<ip-address>:<port>"
		```
		For example:
		
		```		
		persistent_peers = "eea43b2084e78945c3a51babe659efe55fe58370@35.222.198.102:26656"
		```
		
	- Add the `unconditional_peer_ids`
	
		```
		unconditional_peer_ids = "<node-id>"
		```
		For example:
		
		```
		unconditional_peer_ids = "eea43b2084e78945c3a51babe659efe55fe58370"
		```
		
	- Add the `private_peer_ids` (optional)
	
		```
		private_peer_ids = "<node-id>"
		```

	- The following are the current options for TESTNET peers:

		```
		sentry-node-00 - 0e687ef17922361c1aa8927df542482c67fb7571@35.222.198.102:26656

		sentry-node-01 - 114988f9a053f594ab9592beb79b924430d355ba@34.123.40.240:26656

		sentry-node-02 - c533d7ee2037ee6d382f773be04c5bbf27da7a29@34.70.189.2:26656

		sentry-node-03 - a435339f38ce3f973739a08afc3c3c7feb862dc5@35.192.223.187:26656
		```
9. Configure the systemd service and connect to TESTNET

   	- Create the service
   	  ```
   	  sudo <<EOF >> /etc/systemd/system/mantra.service
   	  [Unit]
	  Description=Mantra Node
	  After=network.target
	  StartLimitIntervalSec=60
	  StartLimitBurst=3
	
	  [Service]
	  User=mantra
	  Type=simple
	  Restart=always
	  RestartSec=30
	  ExecStart=/home/mantra/bin/mantrachaind start

   	  [Install]
   	  WantedBy=default.target
   	  EOF
	  ```
   	- Enable the service then start it.
   	  ```
   	  sudo systemctl daemon-reload
   	  sudo systemctl enable mantra.service
	  sudo systemctl start mantra.service
 	  ```
   	
10. Allow Validator to sync with network
    
    	- Check the progress using
          ```journalctl -u mantra -f```  or   
          ```curl 127.0.0.1:26657/status | jq .result.sync_info.catching_up```
    
	-	Once sync'ed, send a minimum of 1 AUM to the address of the Validator (e.g. `mantra1vqlnhx8pmwmz2sswg7r4syuyegej9zunclaudd`)
11. Create Validator
	- Run the following command:

		```
		mantrachaind tx staking create-validator \
		--amount=1000000uatom \
		--pubkey=$(mantrachaind tendermint show-validator) \
		--moniker="<your-moniker>" \
		--chain-id="mantrachain-1" \
		--commission-rate="0.10" \
		--commission-max-rate="0.20" \
		--commission-max-change-rate="0.01" \
		--min-self-delegation="1000000" \
		--gas=2 \
		--gas-prices="0.0002uaum" \
		--from="<your-key-ring>"
		```
	
