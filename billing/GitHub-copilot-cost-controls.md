<b>GitHub Copilot Billing Controls – Summary</b><br>
Overview<br>
Our eicorg organization uses GitHub Copilot Business, which is billed on a fixed per-user subscription model:<br>
$19 per user per month<br>
Total cost = number of assigned users × $19 <br>
There is no usage-based billing for standard Copilot features (e.g., number of prompts or completions).<br>
________________________________________
<b>Current Cost Structure (May, 2026)</b><br>
Current users: 4 (Seth, Latif, Andrey, Kunal)<br>
Monthly baseline cost: $76/month<br>
This cost is predictable and does not increase with heavier usage.<br>
________________________________________
<b>Managing Users</b><br>
User with administrative privileges on eicorg can go to their eicorg home page and select settings<br>
Select "Copilot" on the left side and click on "Access"<br>
Click the Add Seats button and enter the new user.<br>
To remove a user select user and choose 'Cancel Seat' from the ellipse menu next to their name<br>
________________________________________
<b>Risk of Unexpected Charges</b><br>
Unexpected charges can only occur from:<br>
Adding additional Copilot users (increases fixed monthly cost)<br>
Enabling usage-based GitHub services (e.g., Codespaces, Actions, premium AI features)<br>
________________________________________
<b>Controls Implemented</b><br>
1. Seat Management (Primary Control)<br>
Only organization owners (Kunal, Seth) can assign Copilot licenses<br>
This ensures total cost can only increase through explicit administrative action<br>
________________________________________
2. Hard Limits on Usage-Based Services<br>
We have configured strict budgets with enforcement enabled:<br>
Codespaces → Budget: $0 → Usage blocked<br>
GitHub Actions → Budget: $0 → Usage blocked<br>
Premium Copilot Features → Budget: $0 → Usage blocked<br>
Result:<br>
No variable or usage-based charges can be incurred<br>
Any attempt to use these services is automatically prevented<br>
________________________________________
3. Copilot Budget Monitoring (Informational Only — Not a Spending Cap)<br>
Copilot budget set to: $75/month<br>
This budget does not allow additional spending beyond the per-user subscription cost<br>
It is used only for alerting and visibility, not enforcement<br>
Important clarification:<br>
Copilot charges are strictly determined by the number of users (e.g., 4 × $19 = $76)<br>
The system cannot increase spending to $75 unless additional users are added<br>
The $75 value is simply a monitoring threshold, not an approved spending limit<br>
In other words:<br>
The budget does not authorize extra spending<br>
It only defines when alerts are triggered<br>
It could alert if a new user account was activated and the spending suddenly increased<br>
It is informational only and monitoring is not necessary<br>
________________________________________
<b>Resulting Cost Behavior</b><br>
Monthly spend is fully predictable and controlled<br>
Maximum expected cost = $19 × number of approved users<br>
No risk of overages from usage or activity levels<br>
________________________________________
<b>Conclusion</b><br>
The current configuration ensures:<br>
All variable-cost services are disabled or blocked<br>
Copilot costs are fixed and controlled via user assignment<br>
The Copilot budget is used only for monitoring and does not permit additional charges<br>
No unexpected charges can occur without deliberate administrative action<br>
This setup provides strong cost control with minimal operational overhead.<br>
