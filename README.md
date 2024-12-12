# Jimbos-Switchboard-Oracle-Setup-Guide

This is a guide for setting up a linux server to be a Switchboard oracle that generates Sol by providing dApps with random numbers and /or price feed datas.

_Discalimer: This guide could contain errors ror flaws. Use at your own risk!_

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




