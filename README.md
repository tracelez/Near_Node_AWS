# Near_Node_AWS
Near Chunk Node Installation Guide for AWS



### Useful links

Wallet: https://wallet.shardnet.near.org/

Explorer: https://explorer.shardnet.near.org/ 

Mobatek:  https://www.mobatek.net/


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
