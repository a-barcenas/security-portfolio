# Pre-Production Security Review of Cloud Deployment

## Scenario
Intern shipped a new notification service Friday at 4:57 PM. I conducted a deploy review on the intern’s service against a production baseline and offered recommendation for immediate remediation.

## Environment
Live multi-user Azure training tenant, with Reader access.

## Review
Method: Compared intern's configuration against a deployed baseline.

PRIOTITY CALL: Close the active exposure in the intern's container.

Recommended Priority for Remediation:
1.	Close active exposure.
2.	Attach a User assigned managed identity.
3.	Redeploy into the proper region.
4.	Add an inbound deny rule and address restrictions.

Findings:

1.	THE WHERE. After reviewing the Overview of func-inter-notify, it is set up in the wrong region:

<img width="1258" height="359" alt="Screenshot 2026-09-21 at 19 12 49" src="https://github.com/user-attachments/assets/3fed3461-574b-47e9-b0a1-bc576229adfe" />

Screenshot of baseline region provided for reference.
<img width="1465" height="331" alt="Screenshot 2026-09-21 at 19 41 00" src="https://github.com/user-attachments/assets/c4a27bd2-abcf-47cd-b47f-04b6581b13dd" />


2.	THE WHO. Next I checked the identity it is assigned for authentication and found that none existed.

System assigned: Off
<img width="1192" height="414" alt="Screenshot 2026-09-21 at 19 34 53" src="https://github.com/user-attachments/assets/3d126e9c-61c6-4c06-ad18-c397fd1b91fd" />

User assigned: None
<img width="1395" height="436" alt="Screenshot 2026-09-21 at 19 35 35" src="https://github.com/user-attachments/assets/2db236b4-6ff9-4187-a315-9b620a31d517" />


Screenshot of the baseline provided for reference.
<img width="1323" height="445" alt="Screenshot 2026-09-21 at 19 36 25" src="https://github.com/user-attachments/assets/baa3924c-b678-4c6b-a422-0b3e68d5ba0a" />


3.	THE LEAK. Checking the intern’s storage account, I found the access level of their notes wasn't set to Private.

<img width="1190" height="431" alt="Screenshot 2026-09-21 at 19 46 20" src="https://github.com/user-attachments/assets/48f5fa74-6f4d-45fd-a605-8d2980ca5cb3" />


Looking into the blob, the URL is exposed to the open Internet and contains information left there by the intern. Anyone who's able to acquire the URL would have free external access.
<img width="826" height="391" alt="Screenshot 2026-09-21 at 19 52 53" src="https://github.com/user-attachments/assets/f6331497-0a4b-4f0f-921c-9b411fba4d39" />


Proof of exposure.

<img width="450" height="113" alt="Screenshot 2026-09-21 at 19 50 01" src="https://github.com/user-attachments/assets/89710c41-b6ab-4119-9d89-ff7f867649e0" />


4.	THE DOOR. Reviewing the network access of the intern’s Function App found that there were in fact, no restrictions at all. Public Network Access was enabled from all networks, Unmatched rule action set to allow and no matched rule restrictions.

<img width="1190" height="825" alt="Screenshot 2026-09-21 at 20 04 53" src="https://github.com/user-attachments/assets/65b8ba10-4bf5-467a-8cf2-ad35aa340ae4" />


For reference, baseline is set to enable access only to select addresses and by default will deny all unmatched rules. 
<img width="1270" height="824" alt="Screenshot 2026-09-21 at 20 07 25" src="https://github.com/user-attachments/assets/37fccccc-2f91-42b5-9276-9cc04a0db95e" />


5.	THE CALL. Close the active exposure. 
Change the container access level from anonymous/blob to private. Re-test from an unauthenticated private window to confirm the object no longer loads. Then check access logs for unexpected access while exposure was present.


## What broke / what surprised me
What's surprising is that there aren't policies in place to prevent the deployment with some of these configurations. I was also surprised to learn that zero inbound restrictions is the platform default. 

## What I learned

1.	The Function App had no assigned identity, so it's relying on stored credentials which can be vulnerable to discovery and exploitation.
2.	Not all of these findings can be prevented automatically by policies, like a compute resource calling data from external regions.
3.	Set up and enforce configurations that are eligible, such as the network access restrictions and public containers/storage accounts.
