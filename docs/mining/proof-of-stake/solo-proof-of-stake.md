# **Solo Proof-of-Stake (PoS) Mining**


## **Overview**
If you want to do something akin to running your own pool, where "hot wallets" on virtual private servers (VPSs) will do the voting on your behalf, but your wallet which contains your funds (see [Setting up a secure cool wallet for Decred](secure-cool-wallet.md)) will control ticket buying and receive all rewards this guide is for you.

Basically the advantage of the setup is that the VPSs will be online and ready to vote 24/7 so your machine at home (cool wallet) does not have to be. You will also have redundancy because you will be running multiple hot wallets on servers in different geographic locations.

The advantage of this over using a pool is that there is no risk of a pool operator voting in a manner contrary to your wishes, it promotes decentralization, and there is no pool fee. The only additional costs will be those of maintaining the VPSs which may be significantly lower than what a pool might cost if you have a large number of tickets.

---

## **VPS Providers**
The first thing you will need is to choose at least one VPS provider. There is a short list of good providers below, you may mix and match any number of providers.

* [Google Cloud Platform (GCP)](https://cloud.google.com/)
Use at least a VM Instance with 1 shared vCPU and 1.7 GB of memory.

* [Amazon Web Services (AWS)](https://aws.amazon.com/)
Use at least a t2.micro EC2 Instance with 1GB of memory.

* [OVH](https://www.ovh.com/)
Use at least a VPS SSD instance with 2 GB of memory.

Note: Another consideration is storage space. As you will need to store the complete Decred blockchain which is constantly growing, it is important to keep in mind that while it is currently quite small, it will eventually grow to the point where you may need to upgrade the storage space on your VPS instances.

---

## **Hot Wallet Setup**

Once you have set up a VPS you will need to get Decred up and running on it, so that you may generate a voting address. This will be the address which you specify when you purchase tickets, so as to grant the hot wallets voting rights. Follow the steps below to set up your hot wallet. You can then repeat the steps for each additional instance you want to run, or just clone the VPS from the control panel of your provider and select a different geographic location where you would like to host it. It is highly recommended you run at least two hot wallets, one on the East coast of the US and one in Europe, adding an additional one on the West coast of the US is also a good idea.

Use SSH to connect to one of the VPS instances you set up. We will assume the OS you are running is Ubuntu 16.04 LTS as it is popular and available on all the providers listed above.
Once connected download and install the latest version of Decred using dcrinstall (https://github.com/decred/decred-release/releases).

Update the OS:

`sudo apt update && sudo apt upgrade`

Install `tmux`:

`sudo apt install tmux`

Download the latest Decred CLI release:

`wget https://github.com/decred/decred-release/releases/download/v1.1.0/dcrinstall-linux-amd64-v1.1.0`

Make it executable:

`chmod +x dcrinstall-linux-amd64-v1.1.0`

Run the installer:

`./dcrinstall-linux-amd64-v1.1.0`

During the install process you will create a new wallet and write down the 33 word seed, you will reuse this seed for each VPS you set up so they all have the same addresses.

Create a script to set everything up:

`echo "tmux new -d -s dcrd 'dcrd' & tmux new -d -s dcrwallet 'dcrwallet --enablevoting --promptpass' & tmux attach -t dcrwallet" > ~/decred.sh`

Make the script executable:
`chmod +x decred.sh`

Create symlinks to the binaries to make things easier:

`sudo ln -s ~/decred/dcrd /usr/local/bin/dcrd & sudo ln -s ~/decred/dcrwallet /usr/local/bin/dcrwallet & sudo ln -s ~/decred/dcrctl /usr/local/bin/dcrctl & sudo ln -s ~/decred/promptsecret /usr/local/bin/promptsecret & sudo ln -s ~/decred.sh /usr/local/bin/decred`

Start everything:

`decred`

Now we will generate an address which we will use to delegate voting rights using: 

`dcrctl --wallet getnewaddress`

Copy that address as you will need it later when purchasing tickets using your cool wallet.

To check everything is working you can do:

`tmux attach -t dcrd` to check on `dcrd` or `tmux attach -t wallet` for `dcrwallet`.

To detach from this session press `<CTRL>` + `<B>` and then `<D>` on your keyboard.

You are free to close your `SSH` session and everything that is in a `tmux` session will continue running. You can reconnect and attach to the `dcrd` and `dcrwallet` sessions to verify this.

---

## **Voting**
In order for your hot wallets to vote as you wish you will need to connect to each one and specify your voting choices. This is done by using the following commands:

To see what votes are available.

`dcrctl --wallet getvotechoices`

To set a vote preference.

`dcrctl --wallet setvotechoice <agendaid> <choice>`

So if for example you wished to vote `yes` for the proposed sdiff algo change which will be the first hard fork vote you would do:

`dcrctl --wallet setvotechoice sdiffalgorithm yes`

And remember, this process will need to be repeated on each VPS instance running a hot voting wallet.

---

## **Ticket Buying**
Now from your cool wallet you can purchase tickets using the following command once your wallet is unlocked:

`dcrctl --wallet purchaseticket default 60 1 DsHotWalletAddressFromVPS 10`

Replacing `DsHotWalletAddressFromVPS` with your hot wallet voting address which you generated earlier and using the command above will attempt to purchase `10` tickets with a max price of `60` DCR and delegate voting rights to your VPS instances.

If you wish to automate ticket purchases using ticketbuyer you will need to add the following info in your `dcrwallet.conf` where `DsHotWalletAddressFromVPS` is once again the address you generated on your VPS.

```
enableticketbuyer=1
ticketaddress=DsHotWalletAddressFromVPS
```
