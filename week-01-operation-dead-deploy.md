# Investigating a Suspicious Deployment in Azure

## Scenario
Junior intern with temporary contributor access created a test environment without following company governance standards. The investigation shows what was created, by who, and how it was created. 

## Environment
Live multi-user Azure training tenant, Reader access

## Investigation
1. Went to Resource Manager and found a Resource Group with a name that is inconsistent established naming convention.
<img width="1451" height="540" alt="Screenshot 2026-08-27 at 18 42 15" src="https://github.com/user-attachments/assets/0bfd2719-beff-4d74-b4d2-97fc17a87577" />

2. Inspected for any assigned Tags and found some that were assigned.
<img width="1445" height="456" alt="Screenshot 2026-08-27 at 18 51 59" src="https://github.com/user-attachments/assets/29a35274-a36a-48a6-85c3-1ff296277411" />

3. Checked deployments and found which had created the intern’s resource group.
<img width="1142" height="555" alt="Screenshot 2026-08-27 at 18 59 15" src="https://github.com/user-attachments/assets/7f2229ad-4a86-45b8-bc1d-052aad9346e9" />

4. Moved to the Resource Group’s Policy Compliance page and noticed a non-compliant policy.
<img width="1328" height="621" alt="Screenshot 2026-08-27 at 19 07 05" src="https://github.com/user-attachments/assets/1f362bcc-e1bf-4f1f-abcd-c8b5f9bef18c" />

5. Navigated to Authoring, Assignments, and the appropriate policy under Policy name. Upon reviewing the policy, the Effect parameter is set to allow non-compliant requests.
<img width="1209" height="520" alt="Screenshot 2026-08-27 at 19 12 54" src="https://github.com/user-attachments/assets/789765e0-df81-423c-9701-28b045b3d8d2" />


## What surprised me
What surprised me was how a seemingly simple misconfiguration can potentially major implications.  

## Findings and recommendations
Found improperly assigned policy parameters. Recommend adjusting parameters to meet expected policy outcomes.

## What I learned
- I gained familiarization of Azure policy assignments, subscriptions and tagging.
- Learned navigating between and differences of Subscriptions, Resource Groups and resources.
