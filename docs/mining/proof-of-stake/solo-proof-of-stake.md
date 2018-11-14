# Solo Proof-of-Stake (PoS) Voting


## Overview
If you want to do something akin to running your own pool, where "hot wallets" on virtual private servers (VPSs) will do the voting on your behalf, but your wallet which contains your funds (see [Setting up a secure cool wallet for Decred](secure-cool-wallet.md)) will control ticket buying and receive all rewards this guide is for you.

Basically the advantage of the setup is that the VPSs will be online and ready to vote 24/7 so your machine at home (cool wallet) does not have to be. You will also have redundancy and low latency as you will be running multiple hot wallets on servers in different geographic locations (3 is recommended).

The advantage of this over using a pool is that there is no risk of a pool operator voting in a manner contrary to your wishes, it promotes decentralization, and there is no pool fee. The only additional costs will be those of maintaining the VPSs which may be significantly lower than what a pool might cost if you have a fair number of tickets.

---

## VPS Providers
The first thing you will need is to choose at least one VPS provider. You will find a short list of decent providers below, you may mix and match any number of providers, ideally your VPS should have at least 2 GB of RAM.

* [Scaleway](https://www.scaleway.com/)
* [Google Cloud Platform (GCP)](https://cloud.google.com/)
* [Amazon Web Services (AWS)](https://aws.amazon.com/)
* [OVH](https://www.ovh.com/)

Note: Another consideration is storage space. As you will need to store the complete Decred blockchain which is constantly growing, it is important to keep in mind that while it is currently quite small (~3 GB), it will eventually grow to the point where you may need to upgrade the storage space on your VPS instances.

---

## Hot Wallet Setup

Once you have set up a VPS you will need to get Decred up and running on it, so that you may generate a voting address. This will be the address which you specify when you purchase tickets, so as to grant the hot wallets voting rights. Follow the steps below to set up your hot wallet. You can then repeat the steps for each additional instance you want to run, or just clone the VPS from the control panel of your provider and select a different geographic location where you would like to host it (each hot wallet needs to use the same seed). It is highly recommended you run at least three hot wallets, one on the East coast of the US, one in Europe, and one in Asia should be adequate, adding an additional one on the West coast of the US might be desireable as welif you're into overkill.

Use SSH to connect to one of the VPS instances you set up. We will assume the OS you are running is Ubuntu as it is popular and available on all the providers listed above.
Once connected download and install the latest version of Decred using dcrinstall (https://github.com/decred/decred-release/releases).

To avoid a super long-winded section on setting up the wallets I'll just include the script I've written to set everything up:

`sudo apt update && sudo apt upgrade && sudo apt install tmux curl && v=v1.3.0; a=amd64; b=dcrinstall-linux-${a}-${v}; wget https://github.com/decred/decred-release/releases/download/${v}/${b}; chmod +x ${b}; ./${b}; ip=$(curl icanhazip.com); echo "tmux new -d -s dcrd 'dcrd --externalip=${ip}' & tmux new -d -s dcrwallet 'dcrwallet --enablevoting --promptpass' & tmux attach -t dcrwallet" > ~/decred.sh && chmod +x decred.sh && echo "PATH=~/decred:$PATH" >> ~/.profile && source ~/.profile`

You may want to change the `v=v1.3.0` to the latest version if a newer one has been relased and `a=amd64` to whatever CPU architecture your VPS is using.

Start everything:

`./decred.sh`

The first time you do this the blockchain will be downloaded, I would wait until that process is complete before continuing. You can check what the latest block is on [dcrdata](https://explorer.dcrdata.org/).
Then you can moitor the download progress by doing `tmux attach -t dcrd`.

To detach from this session press `<CTRL>` + `<B>` and then `<D>` on your keyboard.

Once the initial sync is complete do `tmux attach -t dcrwallet` and enter your password.

Now we will generate an address which we will use to delegate voting rights using: 

`dcrctl --wallet getnewaddress`

Copy that address as you will need it later when purchasing tickets using your cool wallet.

You are free to close your `SSH` session and everything that is in a `tmux` session will continue running. You can reconnect and attach to the `dcrd` and `dcrwallet` sessions to verify this.

Now either clone the VPS instance to different regions or repeat the process from scratch with one change. When creating the wallet choose "restore" and reuse the seed you generated for the first voting wallet.

You will also want to open up TCP port `9108` for all your voting nodes so they have better connectivity to the Decred network. On AWS this is done in the "Security Groups" section under "Networking" in EC2. On Google Cloud you can run `gcloud compute firewall-rules create dcrd --allow tcp:9108 --source-ranges=0.0.0.0/0` in the console to achieve the same thing.


---

## Voting
In order for your hot wallets to vote as you wish you will need to connect to each one and specify your voting choices. This is done by using the following commands:

To see what votes are available.

`dcrctl --wallet getvotechoices`

To set a vote preference.

`dcrctl --wallet setvotechoice <agendaid> <yes|no>`

So if for example you wished to vote `yes` for the proposed sdiff algo change which was the first hard fork vote you would do:

`dcrctl --wallet setvotechoice sdiffalgorithm yes`

And remember, this process will need to be repeated on each VPS instance running a hot voting wallet.

---

