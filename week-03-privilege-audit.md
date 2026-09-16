| Method               | Sees                                        | Misses                              |
|----------------------|---------------------------------------------|-------------------------------------|
| IAM Blade / Export   | Active assignments at scope, inherited      | Group members, orphaned principals  |
| Azure CLI            | Same data, scriptable, null principalName   | One scope per run                   |
| Resource Graph (KQL) | Whole tenant in one query                   | Eligible assignments.               |
| PIM Export           | Eligible vs Active, activation history      | Assignments outside PIM             |

# Auditing Azure Resource RBAC
 
## Scenario
Completed a point-in-time audit of Azure environment. Used four methods to cover gaps during the audit. 
 
## Environment
Live multi-user Azure training tenant, Reader access.
 
## Investigation

1. Started off reviewing the IAM Roles_Assignment report that revealed an overprivisioned account with redunant Owner privileges over many resources.

<img width="1314" height="783" alt="Screenshot 2026-09-14 at 18 17 13" src="https://github.com/user-attachments/assets/7e6866ce-e7f9-404e-9d57-2f85ebefd2b0" />


2. Moving to the Azure CLI I searched for deleted principalName or unknown accounts. The review found orphaned assignments on a deleted principal.
<img width="1468" height="364" alt="Screenshot 2026-09-14 at 18 37 26" src="https://github.com/user-attachments/assets/86563939-c448-4534-aacb-2aebd3954c32" />
<img width="1335" height="878" alt="Screenshot 2026-09-14 at 18 31 50" src="https://github.com/user-attachments/assets/6943d6f3-ae26-4e08-831f-0e39cf9a2cb9" />


3. Next I reviewed the Resource Graph which allows for all active accounts on the tenant to be searchable with a single query. 

<img width="917" height="363" alt="Screenshot 2026-09-14 at 19 05 05" src="https://github.com/user-attachments/assets/c1e89cc2-8d41-4b5f-b34b-5a320db68f8c" />


4. Then I checked PIM to review both active and eligible assignments. I activated the eligible role assignment to a hidden resource found in Azure resources.

<img width="1398" height="341" alt="Screenshot 2026-09-16 at 11 00 00" src="https://github.com/user-attachments/assets/86710fc7-563d-4613-8482-00b899734cf4" />


5. After activation was approved, I was able to find the account in the Azure CLI, Resource Graph and PIM Azure Resources Active Assignments. Updated active assignment returned helpful results for the updated KQL search.

<img width="1450" height="474" alt="Screenshot 2026-09-14 at 20 17 50" src="https://github.com/user-attachments/assets/04f5a3e8-af00-444a-b345-16ca40ed6926" />

<img width="1098" height="329" alt="Screenshot 2026-09-16 at 13 44 20" src="https://github.com/user-attachments/assets/476d1d36-55c7-41c3-a341-c7a718cef617" />

<img width="1448" height="612" alt="Screenshot 2026-09-16 at 12 41 02" src="https://github.com/user-attachments/assets/5eb85ee6-60cb-4fa6-88c9-9e0d4db8a380" />


 
## What broke / what surprised me

What took me a while to find and understand was finding the eligible roles for a resource, requesting activation, and then reviewing the membership roles of that activated resource. I was trying to run the script to find the flag before I had requested activiation and it kept turning up empty. What also kept slowing me down was remembering that each scope has its own IAM page. This kept confusing me as I was navigating through Azure looking for this account. 
 
## Findings and recommendations

Severity of findings:

F1: High — orphaned assignment on a deleted principal

F2: High - redundant Owner privileges across scopes assigned by a script. 

F3: High — standing privileged access that should be PIM-eligible. 

F4: Medium — over-provisioned job-function account found in the hunt. 


R1 - Remove scripts assigning individual account Ownership of resources.

R2 - Revoke orphaned assignments

R3 - Replace redundant Owner grants with narrowest job-function and scope.

R4 - Move standing privilege access to PIM eligible. 

 
## What I learned

- Azure RBAC is keyed by object ID. Deleting the Entra object does not delete the assignment. 
- The variety of methods to investigate IAM and the limitations of available tools.
- Next time I'd double check that I have access to relevant available scopes by making active all eligible assignments before beginning my search.
