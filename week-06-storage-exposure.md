# Finding Storage Exposures

## Scenario
A developer on the app team asked how their service accesses a database connection string from the Key Vault. They requested to know what the app is accessing, how it’s accessing it and if any of what’s being accessed is reachable by the public internet.

## Environment
Reader access on a live multi-user Azure training tenant.


## Investigation
1. First, I checked the storage account to verify account match, storage type and blob endpoint address.

<img width="1453" height="408" alt="Screenshot 2026-10-05 at 13 15 14" src="https://github.com/user-attachments/assets/9f7623fb-5899-413e-a4e7-3dcf97b27044" />

<img width="512" height="145" alt="Screenshot 2026-10-05 at 13 13 36" src="https://github.com/user-attachments/assets/8872062d-1b8b-49c6-8f8b-19650983e56a" />


2. Then I went configuration under settings to check if Allow storage account key access was disabled. This means an appropriately scoped managed identity is required for access.

<img width="762" height="824" alt="Screenshot 2026-10-05 at 13 26 49" src="https://github.com/user-attachments/assets/c7f9c1d1-c209-4255-afe0-3193c2cb3509" />


Once confirmed, I moved to IAM to see which Managed Identities has resource permissions for the storage account.

<img width="1447" height="520" alt="Screenshot 2026-10-05 at 13 35 31" src="https://github.com/user-attachments/assets/599be2e6-b068-4dd2-94bc-240a0d86427b" />
<img width="1295" height="355" alt="Screenshot 2026-10-05 at 20 02 02" src="https://github.com/user-attachments/assets/9e1e5950-20a3-4608-8d67-64df3bd82daa" />



3. Next I went to the storage account’s Networking blade. There I checked to see the public network access and private endpoint configurations. Public network access is set to disabled and in Private Endpoints there’s a private endpoint assigned to the same resource group. This is a proper configuration as even if someone were able to call from the public, it would be dropped.

<img width="760" height="245" alt="Screenshot 2026-10-05 at 13 52 24" src="https://github.com/user-attachments/assets/778174fb-3e01-494c-8d3f-ba806f03e241" />
<img width="1101" height="442" alt="Screenshot 2026-10-05 at 13 56 05" src="https://github.com/user-attachments/assets/ddeaa8fd-0ddc-4bd2-ab08-7fd3ca2e21e7" />
<img width="1235" height="573" alt="Screenshot 2026-10-05 at 13 54 40" src="https://github.com/user-attachments/assets/d7cfe0d6-b01e-4c49-9206-0f14657efd1d" />

4. Finally, I checked the Key Vault to review where the app is directed at startup. The secret is accessible by the managed identity and not a connection string, SAS token or hardcoded anywhere else. 

<img width="1346" height="513" alt="Screenshot 2026-10-05 at 16 27 42" src="https://github.com/user-attachments/assets/d7e798d8-2e84-4cc8-9f4c-8e61789b5d8a" />
<img width="803" height="713" alt="Screenshot 2026-10-05 at 16 26 21" src="https://github.com/user-attachments/assets/d4654c4e-2a81-4ee4-8cf2-1c9278139e88" />


5.  After reviewing the storage account, I noticed another storage account in the same resource group named “stmadhatlabbackup”. Digging into the backup, we see that public network access is enabled and public blob access is allowed.

<img width="1364" height="430" alt="Screenshot 2026-10-05 at 16 36 40" src="https://github.com/user-attachments/assets/009f4bd7-1814-4b6b-be8d-8e9e4726db3d" />
<img width="756" height="793" alt="Screenshot 2026-10-05 at 16 36 24" src="https://github.com/user-attachments/assets/e28a62d9-29be-423b-8035-776fabc84d3d" />

With the storage account configured this way, we're able to access it directly from the public internet leaving it susceptible to exposure to anyone that acquires the direct link.


## What broke / what surprised me
What took me a while to understand was the Key Vault. Understanding how the steps pieced together made it difficult for me to conceptualize what the secret was even for. Finally, I realized that the app loads its connection target from Key Vault, allowed by its managed identity. In this case, that target is the storage account.



## Findings and recommendations
The primary storage was found to be properly secured, but there is a backup storage account that is publicly accessible to the external internet. 

First recommendation would be to delete the backup.
Next I’d consider assigning Azure Policy to deny public blob access and public network access on storage


## What I learned
1. I learned that a connection string in the key vault gives direction, not the credentials since that was responsible to the managed identity. 
2. The private endpoint acts as a road, managed identity the key, and both are required for access.
3. Hardening techniques don’t follow copying/backups.

