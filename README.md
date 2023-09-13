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
	- `cd bin`
	- `source .profile` - to make sure the new `/bin` folder is added to `$PATH`.
4.  Get the MANTRA Chain Binary from MANTRA GitHub repo
	- `wget https://github.com/MANTRA-Finance/public/raw/main/no-cosmwasm/mantrachaind.zip`
	- `sudo apt install unzip`
	- `unzip mantrachaind.zip`
5. Initialise `mantrachaind`
	- `mantrachaind init validator-mchain-test-01 --chain-id mantrachain-1`
		- where `validator-mantrachain-test-01` is the moniker and can be any valid string name,
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
	-  `wget https://github.com/MANTRA-Finance/public/raw/main/genesis.json` - get the MANTRA Chain Genesis file.
8. Configure connection to TESTNET
	- Edit `config.toml` file (using your favourite text editor)
	- Add the `persistent_peers`
	
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
		
	- Add the `unconditional_peer_ids` (optional)
	
		```
		private_peer_ids = "<node-id>"
		```
9. Start and connet to TESTNET

	- `mantrachaind start`

	