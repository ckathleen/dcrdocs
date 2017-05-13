# **Solo Proof-of-Stake (PoS) Mining**


## **Overview**
If you want to do something akin to running your own pool, where "hot wallets" on virtual private servers (VPSs) will do the voting on your behalf, but your wallet which contains your funds (cool wallet) [see guide on running a secure cool wallet which I’m writing after I finish this one] will control ticket buying and receive all rewards this guide is for you.

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

Once you have set up a VPS you will need to get Decred up and running on it, so that you may generate a voting address. This will be the address which you specify when you purchase tickets, so as to grant the hot wallets voting rights. Follow the steps below to set up your hot wallet. You can then repeat the steps for each additional instance you want to run, or just clone the VPS is the control panel of your provider and select a different geographic location where you would like to host it. It is highly recommended you run at least two hot wallets, one on the East coast of the US and one in Europe, adding an additional one on the West coast of the US is also a good idea.

Use SSH to connect to one of the VPS instances you set up. We will assume the OS you are running is Ubuntu 16.04 LTS as it is popular and available on all the providers listed above.
Once connected download and install the latest version of Decred using dcrinstall (https://github.com/decred/decred-release/releases).

`wget https://github.com/decred/decred-release/releases/download/v1.0.1/dcrinstall-linux-amd64-v1.0.1`

`sudo chmod +x dcrinstall-linux-amd64-v1.0.1 && ./dcrinstall-linux-amd64-v1.0.1`

Create symlinks to the binaries to make things easier.

`sudo ln -s ~/decred/dcrd /usr/bin/dcrd & sudo ln -s ~/decred/dcrwallet /usr/bin/dcrwallet & sudo ln -s ~/decred/dcrctl /usr/bin/dcrctl & sudo ln -s ~/decred/promptsecret /usr/bin/promptsecret`

Install `tmux` and `htop`.

`sudo apt install tmux htop`

Start `tmux` sessions for `dcrd` and `dcrwallet`.

`tmux new -s dcrd`

Then in that session start `dcrd`.

`dcrd`

To detach from this session press `<CTRL>` + `<B>` and then `<D>` on your keyboard.

`tmux new -s dcrwallet`

Now create a new wallet and write down the 33 word seed, you will reuse this seed for each VPS you set up so they all have the same addresses.

`dcrwallet --create`

Once complete you can start dcrwallet with voting enabled.

`dcrwallet --enablevoting`

Now detach from this session using `<CTRL>` + `<B>` and then `<D>` again.

If you need to reattach to a `tmux` session you can do `tmux attach -t dcrd` for example.

Now we can unlock the wallet permanently using the following command:

`promptsecret | dcrctl --wallet walletpassphrase - 0`

Then we will generate an address which we will use to delegate voting rights using:

`dcrctl --wallet getnewaddress`

Copy that address as you will need it later when purchasing tickets using your cool wallet.

You are free to close your `SSH` session and everything that is in a `tmux` session will continue running. You can reconnect and attach to the `dcrd` and `dcrwallet` sessions to verify this.

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
