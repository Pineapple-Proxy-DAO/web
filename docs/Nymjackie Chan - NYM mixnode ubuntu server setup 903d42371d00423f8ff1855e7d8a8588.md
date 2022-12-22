# Nymjackie Chan - NYM mixnode ubuntu server setup

[RU](Nymjackie%20Chan%20-%20NYM%20mixnode%20ubuntu%20server%20setup%20903d42371d00423f8ff1855e7d8a8588/Nymjackie%20Chan%20-%20%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20%D0%BC%D0%B8%D0%BA%D1%81%D0%BD%D0%BE%D0%B4%D1%8B%20NYM%20%D0%BD%D0%B0%20ubuntu%20%20fa37ce50e4c34b5aabf31de18f25f6e2.md)

### Check IPv6 address is available

Order an additional IPv6 address from your provider before the server setup. To ensure everything is okay view all addresses of your host, should output both IPv4 and IPv6 addresses

```bash
hostname -I
```

```
194.135.119.150 2a12:e340:2000:7::2
```

Ping external IPv6 address

```bash
ping6 -c 5 ipv6.google.com
```

```
PING ipv6.google.com(par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e)) 56 data bytes
64 bytes from par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e): icmp_seq=1 ttl=115 time=72.2 ms
64 bytes from par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e): icmp_seq=2 ttl=115 time=72.1 ms
64 bytes from par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e): icmp_seq=3 ttl=115 time=73.0 ms
64 bytes from par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e): icmp_seq=4 ttl=115 time=72.2 ms
64 bytes from par21s20-in-x0e.1e100.net (2a00:1450:4007:818::200e): icmp_seq=5 ttl=115 time=73.3 ms
```

### Add a mixnode user

```bash
adduser mixnode
```

Give ‘sudo’ privileges to a new user

```bash
usermod -aG sudo mixnode
```

Login as a ‘mixnode’ user

```bash
su - mixnode
```

### Install necessary software

Update system software

```bash
sudo apt-get update
sudo apt-get upgrade
```

Install necessary packages

```bash
sudo apt-get install pkg-config build-essential libssl-dev curl jq git
```

Install rust compiler. Get the actual url [here](https://www.rust-lang.org/tools/install) 

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Log out and log in again to get cargo path available in the current shell

### Enable firewall

Install ufw firewall if needed

```bash
sudo apt-get install ufw
```

Enable it and deny everything by default

```bash
sudo ufw enable
```

Allow nym-mixnode ports (1789,1790,8000) and ssh (22)

```bash
sudo ufw allow 1789,1790,8000,22/tcp
```

Check firewall status

```bash
sudo ufw status
```

### Build nym-mixnode from sources

Set the actual release version in checkout command

```bash
git clone https://github.com/nymtech/nym.git
cd nym
git checkout release/v1.1.3
cargo build --release
```

### Initialize a mixnode

Go to nym-mixnode binary directory

```bash
cd target/release
```

Upload a file named ‘wallet.txt’ with a wallet address inside you want to bond this node, you will use it in the command below. Run a mixnode initialization and save output into ‘bonding.txt’, you will need it to bond your node in the next step.

```bash
./nym-mixnode init --id pineapple --host $(curl ifconfig.me) --wallet-address $(cat wallet.txt) > bonding.txt
```

The directory ‘~/.nym’ with configs and keys will be created.  

### Bond your mixnode

Recommended way to bond is via desktop wallet. Open the wallet and press the Bonding menu.

Step 1. Fill all the needed information using bonding.txt in the previous step

![Screenshot from 2022-12-18 15-59-02.png](Nymjackie%20Chan%20-%20NYM%20mixnode%20ubuntu%20server%20setup%20903d42371d00423f8ff1855e7d8a8588/Screenshot_from_2022-12-18_15-59-02.png)

Step 2. Set the ‘Amount’ to bond starting from the 100 NYM and a ‘Profit margin’ (usually 1-10%).

![Screenshot from 2022-12-18 16-09-38.png](Nymjackie%20Chan%20-%20NYM%20mixnode%20ubuntu%20server%20setup%20903d42371d00423f8ff1855e7d8a8588/Screenshot_from_2022-12-18_16-09-38.png)

### Run mixnode as a service

A service allows you to run a mixnode in a background and start it automatically after server reboots. Upload a file ‘/etc/systemd/system/nym-mixnode.service’ with a content below

```
[Unit]
Description=Nym Mixnode (1.1.3)
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=mixnode
LimitNOFILE=65536
ExecStart=/home/mixnode/nym/target/release/nym-mixnode run --id pineapple
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

Start a service

```bash
sudo systemctl enable nym-mixnode.service
sudo service nym-mixnode start
```

### Describe a mixnode

Edit a file ‘~/.nym/mixnodes/pineapple/config/description.toml’ to your liking

```toml
name = "Pineapple on the pizza"
description = "We, the supporters of pineapple pizza, declare our resistance to all forms of taste oppression. We refuse to be dictated to by arbitrary prejudices and conventional tastes. We believe that everyone has the right to choose what they want to eat, without fear of reprisal or discrimination."
link = "https://pastenym.ch/#/5F5D49XB&key=c6872aafb31dbdfa8e411d884248a14d"
location = "Pizza, Atop"
```

Restart a service to get node description updated

```bash
sudo service nym-mixnode restart
```

Check your node in a explorer, replace %NODE_ID% with your ‘Identity Key’

```bash
https://mixnet.explorers.guru/mixnode/%NODE_ID%
```

### Verify open files limit

Processes in linux usually have an open files limitation. Check it with a command below 

```bash
grep -i "open files" /proc/$(ps -A -o pid,cmd|grep nym-mixnode | grep -v grep |head -n 1 | awk '{print $1}')/limits
```

If 65535 in the output, that everything is okay, otherwise go to the next step. 

```bash
sudo echo DefaultLimitNOFILE=65535 >> /etc/systemd/user.conf
sudo echo DefaultLimitNOFILE=65535 >> /etc/systemd/system.conf
```

Reboot the server

### Check mixnode status

Check if the node is started successfully

```bash
sudo journalctl -u nym-mixnode -o cat | grep "Since startup mixed"
```

Mixed packets should not be zero in the output

```
2022-12-18T05:23:59.829Z INFO  nym_mixnode::node::node_statistics > Since startup mixed 4271 packets! (0 in last 30 seconds)
2022-12-18T05:24:59.831Z INFO  nym_mixnode::node::node_statistics > Since startup mixed 4271 packets! (0 in last 30 seconds)
2022-12-18T05:25:59.832Z INFO  nym_mixnode::node::node_statistics > Since startup mixed 4283 packets! (12 in last 30 seconds)
2022-12-18T05:26:59.834Z INFO  nym_mixnode::node::node_statistics > Since startup mixed 4283 packets! (0 in last 30 seconds)
```

[Nymj**ackie Chan** - установка миксноды NYM на ubuntu server](Nymjackie%20Chan%20-%20NYM%20mixnode%20ubuntu%20server%20setup%20903d42371d00423f8ff1855e7d8a8588/Nymjackie%20Chan%20-%20%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20%D0%BC%D0%B8%D0%BA%D1%81%D0%BD%D0%BE%D0%B4%D1%8B%20NYM%20%D0%BD%D0%B0%20ubuntu%20%20fa37ce50e4c34b5aabf31de18f25f6e2.md)