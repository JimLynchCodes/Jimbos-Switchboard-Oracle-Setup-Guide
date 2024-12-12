# Jimbos-Switchboard-Oracle-Setup-Guide

This is a guide for setting up a linux server to be a Switchboard oracle that generates Sol by providing dApps with random numbers and /or price feed datas.

_Disclaimer: This guide could contain errors or flaws. Use at your own risk!_

<br/>

## _Why?_

Starting and running a fleet of oracle servers correctly should produce profit and contribute to the availability of high quality off-chain data.

<br/>

## Links

- Official switchboard docs for running an oracle: https://docs.switchboard.xyz/docs/switchboard/running-switchboard-oracles

- Some info about SEV on AMD processors: https://www.amd.com/content/dam/amd/en/documents/epyc-technical-docs/tuning-guides/58207-using-sev-with-amd-epyc-processors.pdf

- Helius advanced RPC provider: https://www.helius.dev/pricing

- Triton - a very expensive alternative to helius: https://triton.one/triton-rpc/#pricing-section

-  Switchboard solana stats (https://flipsidecrypto.xyz/DoctorBlocks/switchboard-solana-stats-PBx1CB)

- azure portal: https://portal.azure.com/

- on-chain data Solana: https://www.theblock.co/data/on-chain-metrics/solana

- comparison metrics: XRP Transactions: https://xrpscan.com/metrics

- comparison metrics: Bitcoin Transactions: https://ycharts.com/indicators/bitcoin_transactions_per_day

- comparison metrics: Polygon Transactions: https://polygonscan.com/chart/tx
 
<br/> 

## Cloud Server Provider Comparison

Being forced to run on SEV-enabled AMD is pretty limiting, and there are only a few recommended in the Switchboard docs.


https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/general-purpose/dcdsv3-series?tabs=sizebasic

The DCdsv3 server we need with the specs with	1core	8gb memory,	4GiB EPC Memory is called "Standard_DC1ds_v3".

Image: Ubuntu Server 22.04 LTS - x64 Gen2

Select radio button for "x64".

Size: Standard_DC1ds_v3 - 1 vcpu, 8 GiB memory ($82.49/month) ~ 12/11/24

Ports: Select all three- HTTP (80), HTTPS (443), SSH (22)

Then click review, create, and download the .pem keypair file.

Go to the page for your new resource and locate the "Public IP address"

Now, let's go to the terminal...

---

<br/>

1) ssh in

might have to give permissions for keyfile:
```
chmod 600 /Users/jim/Documents/First-oracle-sb-devnet-Standard-DC1ds-v3_key.pem
```

Add key to keychain: 
```bash
ssh-agent bash
sudo ssh-add ~/.ssh/azure_key/First-oracle-sb-devnet-Standard-DC1ds-v3_key.pem
```

ssh into the new server 

```bash
ssh azureuser@server_ip_address
```

2) Get code

git should be already installed. 👍

___Make sure to use the https clone link!!___

```bash
git clone https://github.com/switchboard-xyz/infra-external.git
```




TLDR; I recommend going with DCadsv5 on Azure with 2 vCPUs and 8 GB of RAM for aroudn $75/month. 

- video on amd hypervision SEV Protection: https://www.youtube.com/watch?v=yr56SaJ_0QI

### Azure (Cheaper)

The Azure Standard_DC2as_v5 virtual machine, which includes 2 vCPUs and 8 GB of RAM, costs approximately $75 per month for Linux-based configurations in a pay-as-you-go model. The hourly rate is around $0.011 depending on the specific region, with some variations based on geographical location and subscription type​.

For precise pricing tailored to your usage, including potential discounts from reserved instances or spot pricing, Azure's pricing calculator is recommend

Azure page of switchbaord docs: https://docs.switchboard.xyz/docs/switchboard/running-switchboard-oracles/hardware-tested-providers-and-setup/hardware-amd-sev-tested-providers/azure


### GCP (Expesive)

The monthly costs for the Google Cloud Platform (GCP) machine types you asked about are as follows:

n2d-standard-8:

Located in us-central1, the cost is approximately $246.72 per month without discounts or commitments​
GOOGLE CLOUD MACHINE TYPE COMPARISON
​
ECONOMIZE CLOUD
.
n2d-highmem-8:

Located in us-central1, the cost is around $266.32 per month under similar conditions​
GOOGLE CLOUD MACHINE TYPE COMPARISON
.



After provisioning server


<br/>

# Server Setup

Get this server on Azure: (AMD SEV) DCasv5 vs (SGX) DCdsv3


In Azure.

Click "Create Resource"

Choose "Ubuntu 22", "Create"




Then ssh in with:
```bash
ssh 
```

## Firewall Setup

1. Enable UFW:

```Bash
sudo ufw enable
```
2. Allow Incoming Traffic on Ports 80 and 443:

```Bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

3. Allow All Outgoing Traffic:

```Bash
sudo ufw default allow outgoing
```

4. Verify the Firewall Rules:
```Bash
sudo ufw status
```

Add your email and IPV4 ip address to the config:
```
vim cfg/00-common-vars.cfg
```

Then start running the scripts in `install/cloud/azure`.


may need to manually install docker:
```
sudo apt update
sudo apt install docker.io docker-compose
```

may need to install solana cli also (for solana-keygen errors) (script 40)
```
sh -c "$(curl -sSfL https://release.anza.xyz/stable/install)"
```

may need to create the /data folder and file for it to use (add add proper permissions)
```
sudo mkdir /data
cd /data
sudo touch devnet_payer.json
sudo chmod 777 /data/devnet_payer.json
```

note the public key and pass phrase! request another airdrop at faucet.solana.com if the script one fails.

B53jgPT8kaSh19kNaqBaqhiLTJhW8oZFEH1gfit97Z7W


(51) keep runnign til it works?

note the output, something like this:
 -> Solana cluster: devnet
 -> queueKey: EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7


Running into error trying to start Kubernetes:

azureuser@First-oracle-sb-devnet-Standard-DC1ds-v3:~/infra-external/install/cloud/azure$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=172.172.195.4 --control-plane-endpoint=172.172.195.4 --ignore-preflight-errors=NumCPU,FileExisting-crictl,FileExisting-conntrack
I1212 03:39:49.571343    7394 version.go:261] remote version is much newer: v1.32.0; falling back to: stable-1.31
[init] Using Kubernetes version: v1.31.4
[preflight] Running pre-flight checks
	[WARNING NumCPU]: the number of available CPUs 1 is less than the required 2
	[WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR KubeletVersion]: couldn't get kubelet version: cannot execute 'kubelet --version': executable file not found in $PATH
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher



KUBERNETES error 😢


azureuser@First-oracle-sb-devnet-Standard-DC1ds-v3:~/infra-external/install/cloud/azure$ sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
crictl
azureuser@First-oracle-sb-devnet-Standard-DC1ds-v3:~/infra-external/install/cloud/azure$ rm -f crictl-$VERSION-linux-amd64.tar.gz
azureuser@First-oracle-sb-devnet-Standard-DC1ds-v3:~/infra-external/install/cloud/azure$ sudo systemctl enable kubelet.service
Failed to enable unit: Unit file kubelet.service does not exist.
azureuser@First-oracle-sb-devnet-Standard-DC1ds-v3:~/infra-external/install/cloud/azure$ which kubelet
azureuser@First-oracle-sb-devnet-Standard-DC1ds-v3:~/infra-external/install/cloud/azure$ sudo apt-get install -y kubelet
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package kubelet
azureuser@First-oracle-sb-devnet-Standard-DC1ds-v3:~/infra-external/install/cloud/azure$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=172.172.195.4 --control-plane-endpoint=172.172.195.4 --ignore-preflight-errors=NumCPU,FileExisting-crictl,FileExisting-conntrack


## Run Through number 2!

It's a new day, and I'm trying this again fresh from scratch!

The game plan: 

- Choose the cheapest DCdsv3 (x86 SGX) server on azure with 2 CPUs.

- ssh in and clone the switchboard infra-external repo.

- Update IP address in the configs.

- Proceed with the steps in `/install/bare-metal/kubernetes`


## 1) Provision the server

Let's begin in the [azure portal](https://portal.azure.com/#home).

1) Click "Create a resource +"

2) Choose Ubuntu Server 22.04 LTS

Top info: selections

Instance details

Virtual machine name - second-try-oracle-devnet-sb-DCdsv3-k8s
Region - (US) East US
Availability options - No infrastructure redundancy required
Security type - Standard
Image - Ubuntu Server 22.04 LTS - x64 Gen2
VM architecture - x64

Size -> click "See all sizes"
     -> expand DC Series
     -> choose DC2ds_v3 (2CPU, 16gb, ~$180/month)
     
Administrator Account section

- use SSH public key
- Keep username as azureuser (or change it if you feel like it)
- generate a new key or use one if you already have it

Inbound Ports
- expand the dropdown and select all three 80, 442, and 22


Click create.

Step 1 Complete!

<br/>

## 2 Diggin Into It

ssh into the server
```
ssh azureuser@IP_ADDRESS
```

_Using the https link(!),_ clone the infra-external repo.
```
git clone https://github.com/switchboard-xyz/infra-external.git
```

<br/>

## Step 3) Editing The Config


Edit the common vars config file:
```
vim 
```

Update __EMAIL__ to yours.

Update __IP4__ to the server public ip address

That's it!

Well, hey, this ain't too hard is it? Good thing you got ole' Jimbo sharing his wisdom out here. :)

<br/>




