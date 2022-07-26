# Near Chunk Node AWS Installation
Near Chunk Node Installation Guide for AWS



### Useful links

Wallet: https://wallet.shardnet.near.org/

Explorer: https://explorer.shardnet.near.org/ 

MobaXterm:  https://www.mobatek.net/


#### Server Requirements
Please see the hardware requirement below:

| Hardware       | Chunk-Only Producer  Specifications                                   |
| -------------- | ---------------------------------------------------------------       |
| CPU            | 4-Core CPU with AVX support                                           |
| RAM            | 8GB DDR4                                                              |
| Storage        | 500GB SSD                                                             |



## AWS Chuck Node AWS Setup

Login to AWS and go to EC2 and select launch instances from the desired region:

![image](https://user-images.githubusercontent.com/74465527/181061834-923f78a7-9cce-4e86-a1ce-b52d1593a89a.png)

You will select the Ubuntu 22.04 HVM AMI with a instance type of c6axlarge:

![image](https://user-images.githubusercontent.com/74465527/181062124-247f1c79-e7dd-437b-b790-12f9b1c83527.png)

As you can see the cost is $0.153 USD per hour which roughly translates to $240 per month cost.

Next either create a new keypair or use an existing keypair

![image](https://user-images.githubusercontent.com/74465527/181062602-b470e91f-af09-4962-9f24-5e8892e60fd5.png)

Next we will configure a new security group for the near node and open port 24567 to the world and allow your public IP access to SSH:

![image](https://user-images.githubusercontent.com/74465527/181062641-a34caa21-df07-4825-b371-d8e049f69a8e.png)

Set the storgare to 500 GB single disk as shown below:

![image](https://user-images.githubusercontent.com/74465527/181063100-6a99bec7-799d-4b52-b9b3-6c892aca7a4f.png)

Review your setting and launch the new instance:

![image](https://user-images.githubusercontent.com/74465527/181063280-08eb8a8b-02b9-4fac-9bbc-a35e3ee3b166.png)

Once launched you can access view the new servers information by clicking view instances and going to the new server:

![image](https://user-images.githubusercontent.com/74465527/181063473-d47ae431-e33c-4e7d-b2e7-aa93f64b072b.png)

In this example we will use Mobaxterm as the terminal software. Setting are as follows for this example:

![image](https://user-images.githubusercontent.com/74465527/181063839-bc6ac809-7766-4f31-b174-d83b7ff387cb.png)

Click okay and double click the new entry and you should now connect to the new near node:

![image](https://user-images.githubusercontent.com/74465527/181064182-6a81c42b-96c7-499b-a851-04ffadc27fdd.png)


#### Install required software & set the configuration


##### Prerequisites:
Before you start, you may want to confirm that your machine has the right CPU features. 

```
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
```
> Supported


##### Install developer tools:
```
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python-is-python3 docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo ntp
```

#####  Install Python pip:

```
sudo apt install python3-pip
```
##### Set the configuration:

```
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
```

##### Install Building env
```
sudo apt install clang build-essential make
```

##### Install Rust & Cargo
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

You will see the following:

![image](https://user-images.githubusercontent.com/74465527/181067257-219d2295-c11b-466f-a105-1a3156c1d29e.png)


Press 1 and press enter.

##### Source the environment
```
source $HOME/.cargo/env
```

#### Clone `nearcore` project from GitHub
First, clone the [`nearcore` repository](https://github.com/near/nearcore).

```
git clone https://github.com/near/nearcore
cd nearcore
git fetch
```

Checkout to the commit needed. Please refer to the commit defined in [this file](https://github.com/near/stakewars-iii/blob/main/commit.md). 
```
git checkout <commit>
```

#### Compile `nearcore` binary
In the `nearcore` folder run the following commands:

```
cargo build -p neard --release --features shardnet
```
The binary path is `target/release/neard`. If you are seeing issues, it is possible that cargo command is not found. Compiling `nearcore` binary may take a little while.

#### Initialize working directory

In order to work properly, the NEAR node requires a working directory and a couple of configuration files. Generate the initial required working directory by running:

```
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
```

![image](https://user-images.githubusercontent.com/74465527/181067415-4718490f-2702-4153-a07b-5456bbb64bd7.png)

This command will create the directory structure and will generate `config.json`, `node_key.json`, and `genesis.json` on the network you have passed. 

- `config.json` - Configuration parameters which are responsive for how the node will work. The config.json contains needed information for a node to run on the network, how to communicate with peers, and how to reach consensus. Although some options are configurable. In general validators have opted to use the default config.json provided.

- `genesis.json` - A file with all the data the network started with at genesis. This contains initial accounts, contracts, access keys, and other records which represents the initial state of the blockchain. The genesis.json file is a snapshot of the network state at a point in time. In contacts accounts, balances, active validators, and other information about the network. 

- `node_key.json` -  A file which contains a public and private key for the node. Also includes an optional `account_id` parameter which is required to run a validator node (not covered in this doc).

- `data/` -  A folder in which a NEAR node will write it's state.

#### Replace the `config.json`

From the generated `config.json`, there two parameters to modify:
- `boot_nodes`: If you had not specify the boot nodes to use during init in Step 3, the generated `config.json` shows an empty array, so we will need to replace it with a full one specifying the boot nodes.
- `tracked_shards`: In the generated `config.json`, this field is an empty. You will have to replace it to `"tracked_shards": [0]`

```
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```

#### Get latest snapshot

**IMPORTANT: NOT REQUIRED TO GET SNAPSHOT AFTER HARDFORK ON SHARDNET DURING 2022-07-18**

Install AWS Cli
```
sudo apt-get install awscli -y
```

Download snapshot (genesis.json)
```
// IMPORTANT: NOT REQUIRED TO GET SNAPSHOT AFTER HARDFORK ON SHARDNET DURING 2022-07-18
cd ~/.near
wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
```

If the above fails, AWS CLI may be oudated in your distribution repository. Instead, try:
```
pip3 install awscli --upgrade
```

#### Run the node
To start your node simply run the following command:

```
cd ~/nearcore
./target/release/neard --home ~/.near run
```

![image](https://user-images.githubusercontent.com/74465527/181067749-57cb2ef0-e860-4965-8792-cf446c244baf.png)
The node is now running you can see log outputs in your console. Your node should be find peers, download headers to 100%, and then download blocks.

----



### Activating the node as validator
##### Authorize Wallet Locally
A full access key needs to be installed locally to be able to sign transactions via NEAR-CLI.


* You need to run this command:

```
near login
```

> Note: This command launches a web browser allowing for the authorization of a full access key to be copied locally.

1 – Copy the link in your browser


![image](https://user-images.githubusercontent.com/74465527/181067949-41580187-5567-4fcd-9a1d-65d382fb40b8.png)

2 – Grant Access to Near CLI

![image](https://user-images.githubusercontent.com/74465527/181068051-d78c945b-0599-4edd-9873-3b7e4ff23d0a.png)

3 – After Grant, you will see a page like this, go back to console

![image](https://user-images.githubusercontent.com/74465527/181068113-775f287f-7df6-48f3-b304-818d0a4583fe.png)

4 – Enter your wallet and press Enter

![image](https://user-images.githubusercontent.com/74465527/181068273-4884fcff-898b-43e9-839d-1ec742cd805e.png)


#####  Check the validator_key.json
* Run the following command:
```
cat ~/.near/validator_key.json
```


> Note: If a validator_key.json is not present, follow these steps to create one

Create a `validator_key.json` 

*   Generate the Key file:

```
near generate-key <pool_id>
```
<pool_id> ---> xx.factory.shardnet.near WHERE xx is you pool name

* Copy the file generated to shardnet folder:
Make sure to replace <pool_id> by your accountId
```
cp ~/.near-credentials/shardnet/YOUR_WALLET.json ~/.near/validator_key.json
```
* Edit “account_id” => xx.factory.shardnet.near, where xx is your PoolName
* Change `private_key` to `secret_key`

> Note: The account_id must match the staking pool contract name or you will not be able to sign blocks.\

File content must be in the following pattern:
```
{
  "account_id": "xx.factory.shardnet.near",
  "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
  "secret_key": "ed25519:****"
}
```

#####  Start the validator node

```
target/release/neard run
```
* Setup Systemd
Command:

```
sudo nano /etc/systemd/system/neard.service
```
Paste:

```
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=ubuntu
#Group=near
WorkingDirectory=/home/ubuntu/.near
ExecStart=/home/ubuntu/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

> Note: Change USER to your paths

Command:

```
sudo systemctl enable neard
```
Command:

```
sudo systemctl start neard
```
If you need to make a change to service because of an error in the file. It has to be reloaded:

```
sudo systemctl reload neard
```
###### Watch logs
Command:

```
journalctl -n 100 -f -u neard
```
Make log output in pretty print

Command:

```
sudo apt install ccze
```
View Logs with color

Command:

```
journalctl -n 100 -f -u neard | ccze -A
```
