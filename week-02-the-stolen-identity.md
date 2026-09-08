# Investigating an Identity Attack in Entra ID

## Scenario

Attacker gained access to the tenant within the last 24 hours without triggering any alerts. The team flagged new items found in an old internal legacy connector app. Investigation found weak control and monitoring of Entra apps and OAuth configurations.

## Environment
Live multi-user Azure training tenant, Reader access.

## Investigation
1. Compromised account had retained Ownership access to a legacy service app. This legacy service was used to gain access and establish a foothold in the environment. 
<img width="903" height="545" alt="Screenshot 2026-09-07 at 11 22 52" src="https://github.com/user-attachments/assets/54e088a6-0a13-47cf-9394-051d5214dffc" />


2. Attacker gained elevated privileges by minting a new client secret on the legacy app, allowing the inheritance of the service principal’s app directory permissions. Note Client secret expiry date.
<img width="1295" height="632" alt="Screenshot 2026-09-07 at 11 46 55" src="https://github.com/user-attachments/assets/a2553a9c-334f-4842-9169-809bc4d73bb7" />


3. The attacker then created a new app registration and established its service principal as an Owner of the exploited legacy app.
<img width="1344" height="613" alt="Screenshot 2026-09-07 at 12 03 27" src="https://github.com/user-attachments/assets/6e046a24-80f1-4d49-8412-b3783d23dd28" />
<img width="953" height="504" alt="Screenshot 2026-09-07 at 12 08 08" src="https://github.com/user-attachments/assets/e38efc97-041a-43c8-9fdd-9c7bdd683a74" />


4. To establish persistence, the attacker created and configured a custom scope via the “Expose an API” blade on the legacy service app that enables them to regain access by using the custom exposed API and their rogue app credentials.
<img width="1251" height="731" alt="Screenshot 2026-09-07 at 13 16 07" src="https://github.com/user-attachments/assets/b80a72fc-f58c-437f-8175-c1ad2f5c4c02" />


5. To collect tokens from users, the attackers created a phishing URL to execute confused deputy attacks and sidestep security controls.
<img width="1446" height="629" alt="Screenshot 2026-09-07 at 13 43 17" src="https://github.com/user-attachments/assets/732f1068-f193-4295-884f-37f490997603" />


## What surprised me
The default privileges that are set for these accounts/apps and the lack of alerts that are generated. Without careful attention to how policies are set up to configure scope/permissions, limitations preventing read/write access are minimal.

## Findings and recommendations
Found a rogue app with a redirect URI and a legacy app with a custom scope on the Expose an API blade.

- Disable/delete rogue app.
- Delete unauthorized scope.
- Rotate credentials.
- Hunt logs to/from the rogue app.
- Restrict user consent.

## What I learned
- Expose an API is an attack vector. Adding a custom scope can turn a trusted app into a permission publisher.
- Internal apps need the same control and oversight as privileged roles/users.
- OAuth phishing can bypass conditional access controls since the user has already passed the regular checks.
