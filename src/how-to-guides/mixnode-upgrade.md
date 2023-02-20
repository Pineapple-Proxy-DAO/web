# Nym Mixnode Upgrade Instructions

*(to get the final command and skip the detailed step-by-step, jump to the [end of this page](#optimized-command-for-an-efficient-update))*

[Official Nym Documentation on Mixnode Upgrade](https://nymtech.net/docs/stable/run-nym-nodes/nodes/mixnodes#upgrading-your-mix-node)

## Introduction

Currently Nym team releases the upgrades on a weekly basis on Tuesdays.

The latest releases can be found on their [Github](https://github.com/nymtech/nym/releases)


## Step-by-step upgrade process

1. Connect to your mixnode
2. Go to the location when your binaries are (e.g. /home/user)
3. Stop the running "nym-mixnode" process
  - `service nym-mixnode stop`
4. Download and replace the existing binaries with the newest ones using `wget`command
  - `wget -N https://github.com/nymtech/nym/releases/download/$(curl -s 'https://api.github.com/repos/nymtech/nym/releases' | sed -n '0,/.*"tag_name": "\(nym-binaries.*\)",/s//\1/p' )/nym-mixnode`
  - using `wget`command will fetch the binaries and replace the existing components, `-N` option will force the overwrite
5. *(Optional)* Run `init` command to update the configuration files
  - the `init` command will **NOT** overwrite your keys
  - `./nym-mixnode init --id <Mixnode ID> --host  <Your host IPV4 address> --wallet-address  <Your Mixnode Wallet address>`
6. Restart your "nym-mixnode" process
  - `service nym-mixnode start`
7. Check that your nym-mixnode process is running as intended
  - `systemctl status nym-mixnode.service`
  - `journalctl -u nym-mixnode -f`
8. Update the version of your binaries also in your configuration file and on the blockchain via the NYM-wallet


## Optimized command for an efficient Update

This command will first download the new binaries, store them into a separate file and change the permissions.
Then it will stop the "nym-mixnode "process, replace the binaries with the new ones, reinit and restart the "nym-mixnode" process

The fact that the `wget` download happens before stopping the "nym-mixnode" process will make the process more efficient.

### With init
```
wget -N https://github.com/nymtech/nym/releases/download/$( curl -s 'https://api.github.com/repos/nymtech/nym/releases' | sed -n '0,/.*"tag_name": "\(nym-binaries.*\)",/s//\1/p' )/nym-mixnode -O nym-mixnode.new && 
chmod u+x nym-mixnode.new && 
service nym-mixnode stop && 
mv nym-mixnode.new nym-mixnode && 
./nym-mixnode init --id <Mixnode ID> --host  <Your host IPV4 address> --wallet-address  <Your Mixnode Wallet address>  && 
service nym-mixnode start
```

### Without init
```
wget -N https://github.com/nymtech/nym/releases/download/$( curl -s 'https://api.github.com/repos/nymtech/nym/releases' | sed -n '0,/.*"tag_name": "\(nym-binaries.*\)",/s//\1/p' )/nym-mixnode -O nym-mixnode.new  && 
chmod u+x nym-mixnode.new  && 
service nym-mixnode stop  && 
mv nym-mixnode.new nym-mixnode  &&
service nym-mixnode start
```
