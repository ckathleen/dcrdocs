# **Setting up a secure cool wallet for Decred**


## **Overview**
Decred’s Proof-of-Stake system requires a user to have a wallet connect to the network in order to purchase tickets so as to participate in its governance model and receive the corresponding rewards. This introduces some risks compared to other setups where a user might choose to store private keys on a paper wallet and not have to worry about a host of attack vectors introduced by being online.

To mitigate a great number of the risks associated with having a Decred wallet connecting to the Internet to purchase tickets a user may want to consider setting up a secure cool wallet. This guide will walk you through a low cost solution that can protect the user from several common pitfalls while explaining the rationale behind the recommendations.

---

## **Hardware**
The first concept we will address is that of _compartmentalization_. There is really no reason that a machine which may be storing hundreds or thousands of dollars worth of cryptocurrency should have multiple roles. It should not be used, for gaming, word processing, web browsing, downloading torrents, etc…

Single-board computers such as the Raspberry Pi 3 Model B are so cheap (~$35 USD) that it is a good investment as well as good operational security to set one up specifically for the purpose of PoS mining. The following steps will explain how to set one up in a secure manner so it may be used with a stakepool or hot wallet setup as described in Solo Proof-of-Stake (PoS) Mining.

* [Acquire a Raspberry Pi 3 Model B.](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)

* [Flash Raspbian Jessie Lite to a microSD card.](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)

* [Boot it up and install Decred using dcrinstall for Linux ARM64.](https://docs.decred.org/getting-started/install-guide/#dcrinstall)


**Important**

When you create your wallet, write the 33 seed words down using a pen and paper on a hard surface, and write out two copies.

* **DO NOT** take a picture of the seed words with your phone’s camera.
* **DO NOT** save the seed words in a text file on another computer.
* **DO NOT** email the seed words to yourself for safe keeping.

The only way a person should be able to get the seed words is by acquiring one of the copies you have written out. You should store one copy offsite in case of fire or theft.

Once you have two copies of the seed words we will erase the wallet you just created and recreate it from the seed words to test that they were written down correctly since incorrectly transcribing the words could leave you unable to recover your funds in case of emergency. We will verify that both copies are correct by taking the first seed word from the first copy the second seed word from the second copy and alternate until all words have been input while checking both copies for consistency.

`rm ~/.dcrwallet/mainnet/wallet.db`

Then create a new wallet using `dcrwallet --create` and choose to restore the wallet from seed.

Once the wallet is created we can do a few things to make life easier. 

Create a  `bash` script called `decred.sh` which will start a `tmux` session for each application, start `dcrd`, start `dcrwallet`, prompt us for the password to unlock the wallet, and start `dcrctl`.

```
cat << EOF > ~/decred.sh
#!/bin/bash
tmux new -d -s dcrd 'dcrd' & tmux new -d -s dcrwallet 'dcrwallet --enablevoting' & promptsecret | dcrctl --wallet walletpassphrase - 0 && tmux new -s dcrctl
EOF
```

Now make it executable with `chmod +x ~/decred.sh`

Then we will make symlinks to the Decred binaries and bash script we created above.

`sudo ln -s ~/decred/dcrd /usr/bin/dcrd & sudo ln -s ~/decred/dcrwallet /usr/bin/dcrwallet & sudo ln -s ~/decred/dcrctl /usr/bin/dcrctl & sudo ln -s ~/decred/promptsecret /usr/bin/promptsecret & sudo ln -s ~/decred.sh /usr/bin/decred`

Now block all incoming connections using Uncomplicated Firewall (ufw).

`sudo apt install ufw && sudo ufw default deny incoming && sudo ufw enable`

Now you can start `dcrd`, `dcrwallet`, and unlock it using `promptsecret` just by running `decred`.

The next step will be to start buying tickets manually or using `ticketbuyer` in conjunction with a stakepool or a series of hot wallets you have set up using the [Solo Proof-of-Stake (PoS) Mining guide](solo-proof-of-stake.md).
