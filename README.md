# Jimbos-Switchboard-Oracle-Setup-Guide

This is a guide for setting up a linux server to be a Switchboard oracle that generates Sol by providing dApps with random numbers and /or price feed datas.

_Discalimer: This guide could contain errors ror flaws. Use at your own risk!_

<br/>

## Links

- Official switchboard docs for running an oracle: https://docs.switchboard.xyz/docs/switchboard/running-switchboard-oracles

- Some info about SEV on AMD processors: https://www.amd.com/content/dam/amd/en/documents/epyc-technical-docs/tuning-guides/58207-using-sev-with-amd-epyc-processors.pdf

- Helius advanced RPC provider: https://www.helius.dev/pricing

- Triton - a very expensive alternative to helius: https://triton.one/triton-rpc/#pricing-section

-  Switchboard solana stats (https://flipsidecrypto.xyz/DoctorBlocks/switchboard-solana-stats-PBx1CB)

- azure 



## Cloud Server Provider Comparison

Being forced to run on SEV-enabled AMD is pretty limiting, and there are only a few recommended in the Switchboard docs.


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





