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


## Step 4) Runnin The Scripts

We're on the final stretch now. Let's go!

Remeber, we're following the [Bare Metal With Kubernetes SGX Guide](https://docs.switchboard.xyz/docs/switchboard/running-switchboard-oracles/installation-setup-via-scripts/bare-metal-with-kubernetes-k3s-sgx-only) and running the scripts located in the `infra-external/install/bare-metal/kubernetes` folder.


First up is the docker install script:
```
./infra-external/install/bare-metal/kubernetes/00-docker-install.sh
```

Yes, I know it's a bit confusing that we need to install docker even though we're using the "kubernetes" scripts and not the docker scripts. With this setup we are NOT running the oracle within the docker compose call, BUT we still need some primitive things from the docker development tooling in order to enable containerized application deployments... hopefully that makes sense. 😅


Next,let's run the helm install script:
```
./infra-external/install/bare-metal/kubernetes/01-helm-install.sh 
```

_Note: [Helm](https://helm.sh/) is a package manager for Kubernetes. It simplifies the deployment, management, and sharing of Kubernetes applications by providing a way to define, install, and upgrade even the most complex Kubernetes applications._

Should finish by printing, "HELM: succesfully installed"

Next, let's install SGX:
```
./infra-external/install/bare-metal/kubernetes/10-sgx-install.sh
```

The above command installs Intel SGX components and libraries on your system for secure enclave operations.


```
./infra-external/install/bare-metal/kubernetes/11-sgx-mcu-setup.sh 
```

^ (may need sudo)


```
./infra-external/install/bare-metal/kubernetes/12-sgx-mcu-check.sh
```

^ prints some junk in a list?

```
./infra-external/install/bare-metal/kubernetes/13-sgx-check-sa.sh
```

^ might just give a weird error


Then let's start installing and running the actual switchboard oracle stuff

```
./infra-external/install/bare-metal/kubernetes/40-oracle-ctr-sol.sh
```

^ (may need sudo)

_Note, running 40 will "drop you in a temporary container that will have all the necessary tools to run the following step"_

So don't be afraid if you type ls and see ONLY the 41 command...

Just run it!
```
./41-oracle-create-sol-account.sh devnet
```

Hmmm... didn't work for me:

root@5acdfdae9499:/app# ./41-oracle-create-sol-account.sh 
bash: ./41-oracle-create-sol-account.sh: Is a directory

```
sudo touch devnet_payer.json
```

may need to install solana cli in the temporary container also (for solana-keygen errors) (script 40)
```
sh -c "$(curl -sSfL https://release.anza.xyz/stable/install)"
```
and manually export it on the PATH of temp container:
```
export PATH="/root/.local/share/solana/install/active_release/bin:$PATH"
```

note the pubkey (Gx61GcPxiUWLqFb2B3VUxhTd8DDdUuwWwFA35W7tgMDg) and seed phrase (so you can cash out later! 🤑)

This wallet is where the sol will build up!

If the airdrop fails, try manually putting this new public address into the sol devnet faucet.



Once you've noted the account if, let's exit the temporary container and move onto the next step!
```
exit
```

This will drop you down into another temporary container:
```
./50-oracle-ctr-sb.sh
```

Then run 51
```
./51-oracle-prepare-request.sh devnet
```

Do you want to register an oracle? y
Do you want to register a guardian? n

Note the stuff. eg:

  -> Solana cluster: devnet
  -> queueKey: EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
 
# ORACLE - account - 69W7RxMqPvXxxdLZAqL7MYEGWVFr1xT7NYHMuiP3L3gH
# ORACLE - request tx signature - 65GH4ZfWNEPHJhhxQ9thajz9XQLWrrz3dMd7yQPhRaeih2fHriPFcAhfPeh64Eb1uZHCprm4QMvnNgGHaKdWuVT
PULL_ORACLE=69W7RxMqPvXxxdLZAqL7MYEGWVFr1xT7NYHMuiP3L3gH
PULL_QUEUE=EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
# ORACLE - confirming transaction...
# ORACLE - transaction confirmed.


Then run 52
```
./52-oracle-check-perms.sh devnet
```

Did you also register a guardian, in addition to the oracle? n

Should print a bunch of colorful junk.


Now let's exit this temporary container and move onto the Kubernetes stuff!
```
exit
```

Notice that when you exit a new easter egg appears!!

!!! IMPORTANT !!!
The output from last command represents your Oracle/Guardian public keys (and related data).
It's all public data, so no harm in sharing it. Now you need to proceed with two steps:

1 - Edit infra-external/cfg/00-vars.cfg and add the entire output from above at the end of the file

2 - Copy Submit a request for acceptance of your new Oracle/Guardian data to -> https://forms.gle/2xWwFQ8XPBGu9DRL6

Once you your Oracle/Guardian data will be approved, you can proceed with the last step.


Let's do this...

```
vim infra-external/cfg/00-vars.cfg-var
```

I just pasted these:
```
PULL_ORACLE=69W7RxMqPvXxxdLZAqL7MYEGWVFr1xT7NYHMuiP3L3gH
PULL_QUEUE=EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
```

Also, fill out the google form!


Ok, then we can proceed with the scripts:
```
./infra-external/install/bare-metal/kubernetes/60-k3s-install.sh
```


70's...



We've got all the pieces together... now let's get it running!


```
./99-k8s-oracle-install.sh devnet
```

Getting lots of errors about "Kubernetes cluster unreachable"

====
 
WARN[0000] Unable to read /etc/rancher/k3s/k3s.yaml, please start server with --write-kubeconfig-mode or --write-kubeconfig-group to modify kube config permissions 
error: error loading config file "/etc/rancher/k3s/k3s.yaml": open /etc/rancher/k3s/k3s.yaml: permission denied
KUBECTL: creating Namespace switchboard-oracle-devnet
WARN[0000] Unable to read /etc/rancher/k3s/k3s.yaml, please start server with --write-kubeconfig-mode or --write-kubeconfig-group to modify kube config permissions 
error: error loading config file "/etc/rancher/k3s/k3s.yaml": open /etc/rancher/k3s/k3s.yaml: permission denied
azureuser@second-try-oracle-devnet-sb-DCdsv3-k8s:~/infra-external/install/bare-metal/kubernetes$ sudo !!
sudo ./99-k8s-oracle-install.sh devnet
====
 
HELM: Installing Switchboard Oracle under namespace switchboard-oracle-devnet
Error: Kubernetes cluster unreachable: Get "http://localhost:8080/version": dial tcp 127.0.0.1:8080: connect: connection refused




./99-k8s-oracle-install.sh 
====
 
cp: cannot create regular file '/tmp/helm_values.yaml': Permission denied
azureuser@second-try-oracle-devnet-sb-DCdsv3-k8s:~/infra-external/install/bare-metal/kubernetes$ sudo !!
sudo ./99-k8s-oracle-install.sh 
====
 
HELM: Installing Switchboard Oracle under namespace switchboard-oracle-devnet
Error: Kubernetes cluster unreachable: Get "http://localhost:8080/version": dial tcp 127.0.0.1:8080: connect: connection refused


Arrrgggghghhhhh :(



Third try!

Friday the 13th luck 😉


Create a new server instance: Standard DC2ds v3 (2 vcpus, 16 GiB memory)

_Don't use generate keys!!_


ssh in:
```
ssh azureuser@ip-address
```

clone everything into the /home/ folder!! (not user specific folder)



 
Creating new Oracle/Guardian permission request on Solana for:
  -> Solana cluster: devnet
  -> queueKey: EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
 
# ORACLE - account - CFnWwNWQU4SH1ZrFHEP2fWy2MG1qGMDKkfJuxAXeenGp
# ORACLE - request tx signature - 3iSaHLShmoqtZ8SN7qqP6viqor3dvVmgv1XhsCjgoR9DpZNYjUj4xH4zVcKVgVg9MBbw91EBow2ZBLYbZ1SikDVV
PULL_ORACLE=CFnWwNWQU4SH1ZrFHEP2fWy2MG1qGMDKkfJuxAXeenGp
PULL_QUEUE=EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
# ORACLE - confirming transaction...
# ORACLE - transaction confirmed.
 

 
Oracle being checked:
  -> Solana cluster: devnet
  -> queueKey: EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
 
(node:47) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)
## Queue                            EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
authority                           2KgowxogBrGqRcgXQEmqFvC3PGtCu66qERNJevYW8Ajh
mrEnclavesLen                       1
mrEnclaves                          
    mrEnclave-0                     7a7bab2c90c3071215a2c0937e2807f3ee0cd81cc34d9adbb6979b29393cc5b2
oracleKeysLen                       17
oracleKeys                          
    oracleKey-0                     84RXhHV3b9tRpsXh2MZTn28USJsAhXHvZpZ1q9JsbfT8
        gateway                     https://az-neu-01-1.switchboard-oracles.xyz/devnet
    oracleKey-1                     3Zzkn14g5tv7ekEuevSbpy3LVe2uny9wGznvGCMxfgSh
        gateway                     https://az-neu-01-0.switchboard-oracles.xyz/devnet
    oracleKey-2                     6S5V5WJVtGp6AgFm3fYHyEvQ45TS4Fy8UN5V2Q7wvUAY
        gateway                     https://az-neu-01-3.switchboard-oracles.xyz/devnet
    oracleKey-3                     Ahy8e91sPy3MM39ia4RbLYUhfTXkcRH7MfmvoWbA8ezZ
        gateway                     https://az-neu-01-4.switchboard-oracles.xyz/devnet
    oracleKey-4                     6LkpJH6hZGNpuBVJcpjcAbgdQkYcfb1UFwK6FxeCGyV
        gateway                     https://az-neu-01-6.switchboard-oracles.xyz/devnet
    oracleKey-5                     Q4ATwgwZr1nqbsTYP5g6sVbcHgn666EdXhk1GhL7CDs
        gateway                     https://az-neu-01-5.switchboard-oracles.xyz/devnet
    oracleKey-6                     3WdPQFqJiUyAtTKZdtrE5sUn1FUBG544DGmVatRkR7yD
        gateway                     https://65.20.113.49.xip.switchboard-oracles.xyz/devnet
    oracleKey-7                     7W4m2ssaDs4cYH6WThvjNsygqVJdxDCdxsHmRabJ6MaZ
        gateway                     https://vu-man-01.switchboard-oracles.xyz/devnet
    oracleKey-8                     3HdF8xK42ZKv5d7nJGfZUYwdyDz9akw3n47RReq5X3ux
        gateway                     https://57.129.38.6.xip.switchboard-oracles.xyz/devnet
    oracleKey-9                     7Eg6PFHteYGPr4PwpcDVibFtAihuzyB5BgqYK4DB8u3Q
        gateway                     https://vu-ams-02.switchboard-oracles.xyz/devnet
    oracleKey-10                    9K981VjRtKBKvZqvHLzcbEtjExVnR3KmuYr9dnbbpio5
        gateway                     https://65.20.113.49.xip.switchboard-oracles.xyz/devnet
    oracleKey-11                    AmFiYQBMU1J3ctgC1LuQhcPRsyHZndWmhU86weoS5KQZ
        gateway                     https://212.126.35.132.xip.switchboard-oracles.xyz/devnet
    oracleKey-12                    DqCgWiVHNiDc2f6gcHZCwNGbbbiTKukWHPwdrJFt5tjr
        gateway                     https://198.244.230.159.xip.switchboard-oracles.xyz/devnet
    oracleKey-13                    GWheNiCceL7WvRVnXAF6ZHrk8LteQouBUMLY85JN51g4
        gateway                     https://az-neu-01-2.switchboard-oracles.xyz/devnet
    oracleKey-14                    3soPe1b9mNFmweoXSsBnU1NBKpFJSwYn8L7HTWWiAb88
        gateway                     https://34.147.45.125.xip.switchboard-oracles.xyz/devnet
    oracleKey-15                    DT5d4F1idF6qRQjHa4fXkU5Dsd2qaL95v3ZUEzFfkWZU
        gateway                     https://34.91.12.152.xip.switchboard-oracles.xyz/devnet
    oracleKey-16                    9reg9rafod2moQfAE7VAvypK1UZvoqmy1Qm8kq3E6yJu
        gateway                     https://57.129.18.101.xip.switchboard-oracles.xyz/devnet
maxQuoteVerificationAge             604800
lastHeartbeat                       2024-12-13T17:45:36.000Z
nodeTimeout                         300
oracleMinStake                      0
allowAuthorityOverrideAfter         600
reward                              1000000
currIdx                             0
gcIdx                               0
requireAuthorityHeartbeatPermission 1
requireAuthorityVerifyPermission    0
requireUsagePermissions             0
mint                                N/A

 ---

 <br/>

## DJ Khalid- Anoha One

### Fourty Morty

Ok, we're back, and I learned a few things from Lele responding to my q's in the discord channel. 👍

This time, let's use the scripts in `cloud/azure` and 



Project details
Select the subscription to manage deployed resources and costs. Use resource groups like folders to organize and manage all your resources.
Subscription
Azure subscription 1
Resource group
(New) fourty-morty_group
Create new
Instance details
Virtual machine name
fourty-morty
Region
(US) East US
Availability options
No infrastructure redundancy required




Security type
Trusted launch virtual machines



Configure security features
Image

Ubuntu Server 22.04 LTS - x64 Gen2
See all images | Configure VM generation
VM architecture
Arm64
x x64
Run with Azure Spot discount
You are in the free trial period. Costs associated with this VM can be covered by any remaining credits on your subscription.Learn more
Size
Standard_DC2ds_v3 - 2 vcpus, 16 GiB memory ($164.98/month)
See all sizes
Enable Hibernation
Hibernate does not currently support Trusted launch and Confidential virtual machines for Linux images.Learn more
Administrator account
Authentication type
SSH public key
Password
Azure now automatically generates an SSH key pair for you and allows you to store it for future use. It is a fast, simple, and secure way to connect to your virtual machine.
Username
azureuser
x SSH public key source
Use existing public key
Ed25519 and RSA SSH formats are supported for the selected VM image. Ed25519 provides a fixed security level of no more than 128 bits for 256-bit key, while RSA could offer better security with keys longer than 3072 bits.


Public inbound ports
None
x Allow selected ports
Select inbound ports
HTTP (80), HTTPS (443), SSH (22)
All traffic from the internet will be blocked by default. You will be able to change inbound port rules in the VM > Networking page.


ssh in:
```
ssh azureuser@<your ip>
```

go to home folder:
```
cd /home
```

clone the repo:
```
git cone git@github.com:switchboard-xyz/infra-external.git
```

edit the config (may need sudo):
```
vim infra-external/cfg/00-common-vars.cfg
```

update EMAIL and IP4.

Then exit.




Run helm install:
```
./01-helm-install.sh
```

```
./40-oracle-ctr-sol.sh
```

If above gives you docker error, run docker install command from the bare-metal folders:
```
./install/bare-metal/kubernetes/00-docker-install.sh
```


<br/>
<br/>

## Try Five: Five D

<br/>

1) Made a new server on azure (Ubuntu Server 22.04 LTS, Standard_DC2ds_v3 - 2 vcpus, 16 GiB memory ($164.98/month)

2) ssh into it

3) 
<br/>


51 output:

===================================================
=                !!! IMPORTANT !!!                =
=         COPY/SAVE THE OUTPUT FROM HERE          =
===================================================
 
Creating new Oracle/Guardian permission request on Solana for:
  -> Solana cluster: devnet
  -> queueKey: EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
 
# ORACLE - account - Fzy7CfkXcitAMSoW3gq3TcAx27GAZuibYeMXrgXv2nM5
# ORACLE - request tx signature - 565CHE736Cnnx6mN8PKceA81kax1W3TqfTqsArGGBw8P7ZVtTmiHBKL3TWvCQs1nFZMzbfzfZd7PSZYXwjbSVjsP
PULL_ORACLE=Fzy7CfkXcitAMSoW3gq3TcAx27GAZuibYeMXrgXv2nM5
PULL_QUEUE=EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
# ORACLE - confirming transaction...
# ORACLE - transaction confirmed.
 
===================================================
=                !!! IMPORTANT !!!                =
=  COPY/SAVE THE OUTPUT ABOVE, BEFORE PROCEEDING  =
=  THEN TYPE 'exit' TO LEAVE THIS TMP CONTAINER.  =
===================================================
 



52 Output:

Did you also register a guardian, in addition to the oracle? (y/n) n
===================================================
=       CHECKING THAT YOUR ORACLE IS WORKING      =
===================================================
 
Oracle being checked:
  -> Solana cluster: devnet
  -> queueKey: EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
 
(node:61) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)
## Queue                            EYiAmGSdsQTuCw413V5BzaruWuCCSDgTPtBGvLkXHbe7
authority                           2KgowxogBrGqRcgXQEmqFvC3PGtCu66qERNJevYW8Ajh
mrEnclavesLen                       1
mrEnclaves                          
    mrEnclave-0                     7a7bab2c90c3071215a2c0937e2807f3ee0cd81cc34d9adbb6979b29393cc5b2
oracleKeysLen                       17
oracleKeys                          
    oracleKey-0                     84RXhHV3b9tRpsXh2MZTn28USJsAhXHvZpZ1q9JsbfT8
        gateway                     https://az-neu-01-1.switchboard-oracles.xyz/devnet
    oracleKey-1                     3Zzkn14g5tv7ekEuevSbpy3LVe2uny9wGznvGCMxfgSh
        gateway                     https://az-neu-01-0.switchboard-oracles.xyz/devnet
    oracleKey-2                     6S5V5WJVtGp6AgFm3fYHyEvQ45TS4Fy8UN5V2Q7wvUAY
        gateway                     https://az-neu-01-3.switchboard-oracles.xyz/devnet
    oracleKey-3                     Ahy8e91sPy3MM39ia4RbLYUhfTXkcRH7MfmvoWbA8ezZ
        gateway                     https://az-neu-01-4.switchboard-oracles.xyz/devnet
    oracleKey-4                     6LkpJH6hZGNpuBVJcpjcAbgdQkYcfb1UFwK6FxeCGyV
        gateway                     https://az-neu-01-6.switchboard-oracles.xyz/devnet
    oracleKey-5                     Q4ATwgwZr1nqbsTYP5g6sVbcHgn666EdXhk1GhL7CDs
        gateway                     https://az-neu-01-5.switchboard-oracles.xyz/devnet
    oracleKey-6                     3WdPQFqJiUyAtTKZdtrE5sUn1FUBG544DGmVatRkR7yD
        gateway                     https://65.20.113.49.xip.switchboard-oracles.xyz/devnet
    oracleKey-7                     7W4m2ssaDs4cYH6WThvjNsygqVJdxDCdxsHmRabJ6MaZ
        gateway                     https://vu-man-01.switchboard-oracles.xyz/devnet
    oracleKey-8                     3HdF8xK42ZKv5d7nJGfZUYwdyDz9akw3n47RReq5X3ux
        gateway                     https://57.129.38.6.xip.switchboard-oracles.xyz/devnet
    oracleKey-9                     7Eg6PFHteYGPr4PwpcDVibFtAihuzyB5BgqYK4DB8u3Q
        gateway                     https://vu-ams-02.switchboard-oracles.xyz/devnet
    oracleKey-10                    9K981VjRtKBKvZqvHLzcbEtjExVnR3KmuYr9dnbbpio5
        gateway                     https://65.20.113.49.xip.switchboard-oracles.xyz/devnet
    oracleKey-11                    AmFiYQBMU1J3ctgC1LuQhcPRsyHZndWmhU86weoS5KQZ
        gateway                     https://212.126.35.132.xip.switchboard-oracles.xyz/devnet
    oracleKey-12                    DqCgWiVHNiDc2f6gcHZCwNGbbbiTKukWHPwdrJFt5tjr
        gateway                     https://198.244.230.159.xip.switchboard-oracles.xyz/devnet
    oracleKey-13                    GWheNiCceL7WvRVnXAF6ZHrk8LteQouBUMLY85JN51g4
        gateway                     https://az-neu-01-2.switchboard-oracles.xyz/devnet
    oracleKey-14                    3soPe1b9mNFmweoXSsBnU1NBKpFJSwYn8L7HTWWiAb88
        gateway                     https://34.147.45.125.xip.switchboard-oracles.xyz/devnet
    oracleKey-15                    DT5d4F1idF6qRQjHa4fXkU5Dsd2qaL95v3ZUEzFfkWZU
        gateway                     https://34.91.12.152.xip.switchboard-oracles.xyz/devnet
    oracleKey-16                    9reg9rafod2moQfAE7VAvypK1UZvoqmy1Qm8kq3E6yJu
        gateway                     https://57.129.18.101.xip.switchboard-oracles.xyz/devnet
maxQuoteVerificationAge             604800
lastHeartbeat                       2024-12-17T03:10:11.000Z
nodeTimeout                         300
oracleMinStake                      0
allowAuthorityOverrideAfter         600
reward                              1000000
currIdx                             9
gcIdx                               0
requireAuthorityHeartbeatPermission 1
requireAuthorityVerifyPermission    0
requireUsagePermissions             0
mint                                N/A
 

