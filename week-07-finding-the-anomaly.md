# Title: Investigating an Anomaly

## Scenario
Assigned ticket number SOC-1058, which contained minimal context, “an account is misbehaving, somewhere in this workspace.” Discovered the misbehaving account and investigated suspicious activity. 

## Environment
Live multi-user Azure training tenant, Reader access.

## Investigation
1. Due to lack of context from the ticket, I started the investigation by running a broad sweep with a KQL search of the ticket ID to narrow down the investigation. This returned results of a notes table of a handoff log from the night shift. 

<img width="1460" height="454" alt="Screenshot 2026-10-22 at 19 25 59" src="https://github.com/user-attachments/assets/557b2754-29ea-4f27-acad-88decc6cf06e" />


2. Next I searched for all unsuccessful login attempts and found suspicious behavior from one UPN, which displayed numerous failed logins in rapid succession before we see a successful login. 

<img width="557" height="319" alt="Screenshot 2026-10-22 at 19 38 44" src="https://github.com/user-attachments/assets/3c033268-f80c-4b1c-bff8-7a2ec4d84b24" />

Successful login.
<img width="906" height="380" alt="Screenshot 2026-10-22 at 19 49 27" src="https://github.com/user-attachments/assets/8329a94e-d656-4246-8fe8-8b1282d6d160" />

3. Before reviewing logs for activity after successful login, I wanted to check to see if the relevant resources were properly logging the activity. 

<img width="1203" height="359" alt="Screenshot 2026-10-22 at 20 10 36" src="https://github.com/user-attachments/assets/acde7f9c-3744-4204-ab75-9af94f4050c7" />

After seeing all logging was disabled, I went to check the AzureActivity table to see if/when the settings were disabled, where we observe that the settings were indeed destroyed. 

<img width="1087" height="377" alt="Screenshot 2026-10-22 at 20 28 37" src="https://github.com/user-attachments/assets/c33e673b-d712-432a-93e2-356e9366c162" />

4. Next I wanted to check why there were no alerts triggered after detecting the anomaly in the KQL query. I went to the alert rules to check the Conditions of the alert rule and saw that the threshold was set to an excessively high number. 

<img width="1436" height="672" alt="Screenshot 2026-10-23 at 18 54 53" src="https://github.com/user-attachments/assets/565b3dea-c016-4af4-9151-ad5bed65755f" />

5. Finally I reviewed the associated Workbooks for sign-on activity. Fortunately, the Workbook proved to be a useful backup to the poorly tuned alert thresholds. 

<img width="1449" height="742" alt="Screenshot 2026-10-23 at 19 23 12" src="https://github.com/user-attachments/assets/0f1b3260-d27d-4eea-8eb6-6b2216e9a72b" />



## What broke / what surprised me

What surprised me is that there were no lock-out rules for the numerous failed login attempts and that a rule with such a high threshold was ever implemented. I'm also surprised that features like disabling/enabling the diagnostics don't trigger any sort of notification by default. 


## Findings and recommendations
Burst failed login was observed via KQL query and workbook. 

- Enable MFA. 
- Fix Alert threshold to a reasonable number.
- Tune smart lockout thresholds.
- Create an alert for changes in diagnostics settings on critical resources.

## What I learned
- Global searches are a quick and easy way to orient yourself in an unfamiliar workspace. 
- Workbooks can act as a safety net with ongoing monitoring to display spikes in activity. 
