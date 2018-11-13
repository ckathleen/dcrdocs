# Setting up a secure cool wallet for Decred


## Overview
Decred’s Proof-of-Stake system requires a user to have a wallet connect to the network in order to purchase tickets so as to participate in its governance model and receive the corresponding rewards. This introduces some risks compared to other setups where a user might choose to store private keys on a paper wallet and not have to worry about a host of attack vectors introduced by being online.

To mitigate a great number of the risks associated with having a Decred wallet connecting to the Internet to purchase tickets a user may want to consider setting up a secure cool wallet (not quite a cold wallet). This guide will walk you through a low cost solution that can protect the user from several common pitfalls while explaining the rationale behind the recommendations.

---

## Hardware
The first concept we will address is that of _compartmentalization_. There is really no reason that a machine which may be storing large amounts of cryptocurrency should have multiple roles. It should not be used, for gaming, word processing, web browsing, downloading torrents, etc...

I highly recommend [Qubes OS](https://www.qubes-os.org/) paired with [compatible hardware](https://www.qubes-os.org/hcl/).
For reference it runs really well on a Thinkpad X260 and even an older Thinkpad X220.

If cost is a huge factor, single-board computers such as the Raspberry Pi 3 Model B can also be tasked for this purpose however using a machine that is more powerful and can handle Qubes is preferable.

---

## Configuration

Once your staking machine is set up you will need to install the Decred CLI tools.
1. Import the Decred Release Signing Key in GnuPG.

`gpg --keyserver pgp.mit.edu --recv-keys 0x518A031D`

2. Download the installer, manifest, and signature files.

`wget https://github.com/decred/decred-release/releases/download/v1.3.0/{dcrinstall-linux-amd64-v1.3.0,manifest-dcrinstall-v1.3.0.txt,manifest-dcrinstall-v1.3.0.txt.asc}`

3. Verify the manifest.

`gpg --verify manifest-dcrinstall-v1.3.0.txt.asc`

4. Verify the SHA-256 hash in the manifest matches that of the binary.

`sha256sum dcrinstall-linux-amd64-v1.3.0`

5. Make the binary executable.

`chmod +x dcrinstall-linux-amd64-v1.3.0`

6. Run it.

`./dcrinstall-linux-amd64-v1.3.0`

**Important**

When you create your wallet, write the 33 seed words down using a pen and paper on a hard surface, and write out two copies.

* **DO NOT** take a picture of the seed words with your phone’s camera.
* **DO NOT** save the seed words in a text file on a computer.
* **DO NOT** email the seed words to yourself for "safe keeping".

The only way a person should be able to get the seed words is by acquiring one of the copies you have written out. You should store one copy offsite in case of fire or theft.

---

Once the wallet is created we can do a few things to make life easier. 

1. Create a `bash` script called `decred.sh` which will start a `tmux` session for each application, start `dcrd`, start `dcrwallet`, prompt us for the password to unlock the wallet, and start `dcrctl`. Also make sure you have `tmux` installed.

`echo "tmux new -d -s dcrd 'dcrd & tmux new -d -s dcrwallet 'dcrwallet --promptpass' & tmux attach -t dcrwallet" > ~/decred.sh"`

2. Add the pathe to the Decred binaries to your `.profile`.
`"PATH=~/decred:$PATH" >> ~/.profile && source ~/.profile`

3. Now make it executable with `chmod +x ~/decred.sh`

Now you can start `dcrd`, `dcrwallet`, and unlock it using `promptsecret` just by running `./decred.sh`.
If necessary do `tmux attach -t dcrwallet` to enter the password and then `tmux attach -t dcrctl` to enter commands

The next step will be to start buying tickets manually or using `ticketbuyer` in conjunction with a series of hot wallets you have set up using the [Solo Proof-of-Stake (PoS) Voting guide](solo-proof-of-stake.md).
After you have your voting wallets set up see the section below.

## Ticket Buying
Now from your cool wallet you can purchase tickets using the following command once your wallet is unlocked:

`dcrctl --wallet purchaseticket default 150 1 DsHotWalletAddressFromVPS 10`

Replacing `DsHotWalletAddressFromVPS` with your hot wallet voting address which you generated when you set up your VPS for voting. and using the command above will attempt to purchase `10` tickets with a max price of `150` DCR each and delegate voting rights to your VPS instances.

If you wish to automate ticket purchases using ticketbuyer you will need to add the following info in your `~/.dcrwallet/dcrwallet.conf` where `DsHotWalletAddressFromVPS` is once again the address you generated on your voting VPS.

```
enableticketbuyer=1
ticketbuyer.votingaddress=DsHotWalletAddressFromVPS
ticketbuyer.balancetomaintainabsolute=0
```

## Voting

Setting vote choices for on-chain votes will happen on the VPSs you have set up and the process is documented in [Solo Proof-of-Stake (PoS) Voting guide](solo-proof-of-stake.md).

For Politeia votes you will need to cast those from your cool wallet. I will outline the process for doing so using the `politeiavoter` CLI tool.

1. Download the Politeia archive, manifest, and signature files.

`wget https://github.com/decred/decred-binaries/releases/download/v1.3.1/{politeiavoter-linux-amd64-v1.3.1.tar.gz,manifest-politeiavoter-v1.3.1.txt,manifest-politeiavoter-v1.3.1.txt.asc}`

2. Verify the manifest.

`gpg --verify manifest-politeiavoter-v1.3.1.txt.asc`

4. Verify the SHA-256 hash in the manifest matches that of the archive.

`sha256sum politeiavoter-linux-amd64-v1.3.1.tar.gz`

5. Extract the archive.

`tar -xf politeiavoter-linux-amd64-v1.3.1.tar.gz`

6. Enter the `politeiavoter-linux-amd64-v1.3.1` directory.


Now to view the various agendas and vote on them you will need to run the following commands. Also remember that for this to work `dcrd` and `dcrwallet` must also be running.

To get a list of active proposals:

`./politeiavoter inventory`

To vote on a proposal:
`./politeiavoter vote <prop-hash> <yes|no>`

