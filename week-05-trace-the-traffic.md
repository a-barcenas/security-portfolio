# Tracking Network Traffic

## Scenario
App team submitted a ticket requesting an explanation of why their new service can reach a customer’s data storage account. I reviewed environment routing configuration to explain why traffic is allowed.

## Environment
Reader access on live multi-user Azure training tenant.

## Investigation
TICKET: “Our new service can reach the customer-data storage account, but I don't understand why or how. Can someone walk me through the network path?"


TRACE: Tracking the route.

The VNET: I started by reviewing the subnet where the workload is assigned to. 

<img width="935" height="357" alt="Screenshot 2026-09-28 at 16 42 56" src="https://github.com/user-attachments/assets/c58b973e-18bd-4e6f-92ef-75abc0fe58b8" />



The NSG: Next I moved to inspect the NSG that the subnet is attached to. Found were two custom inbound rules in addition to Azure’s default rules. One custom rule points directly to a single IP address and the other catches and denies all other internet traffic.

<img width="1443" height="442" alt="Screenshot 2026-09-28 at 17 12 18" src="https://github.com/user-attachments/assets/88b2a0da-3d41-4e10-8446-aca31e43183c" />



Only default outbound rules are in place.

<img width="1454" height="419" alt="Screenshot 2026-09-28 at 17 13 22" src="https://github.com/user-attachments/assets/eff2df55-c572-43ce-af4b-fca2601bda44" />



The Route: Then I checked the subnet’s Route table. Present was a single UDR viewable in the picture below. This forced tunnel directs traffics next hop straight to a virtual appliance for inspection.

<img width="1450" height="583" alt="Screenshot 2026-09-28 at 17 20 12" src="https://github.com/user-attachments/assets/92df3357-2351-45e1-a98d-dfdd49cd9c4d" />



The Private Endpoint: Then I check the DNS configuration of pe-lab-storage which has a NIC/Private IP that sits in snet-workload. Calls from local clients on the Private Link succeed and calls from public clients fail.

<img width="920" height="550" alt="Screenshot 2026-09-28 at 18 11 13" src="https://github.com/user-attachments/assets/47d8d04f-73a8-4da6-8a82-530b0b09ac5f" />



The Name: Lastly, I checked the Recordsets in the private DNS zone. The app makes requests for the storage account by hostname, not IP. The DNS zone is linked to the vnet, so Azure DNS answers from the private zone, not the public internet. If zone and vnet were not linked, the name+zone would resolve to public and the call would be dropped.

<img width="1449" height="553" alt="Screenshot 2026-09-28 at 18 26 35" src="https://github.com/user-attachments/assets/0457f12d-8d70-426e-a286-f9752349fe52" />


ANSWER:
Name lookup was rewritten to a private IP in the DNS zone. That IP is a PE NIC that sits in the same subnet. Same-VNet traffic is allowed by the default outbound rules in the NSG. The PE’s host route is more specific than the forced default route. /32 is more narrow/specific than /0.

FLAG: If the control objective is to inspect all outbound traffic, adding a more specific UDR so this prefix goes to the appliance may be prudent. 


## What I learned

- System routes are silent by default.
- Private DNS is what turned the app’s hostname into a PE IP
- Correct paths can still leave control gaps. 
