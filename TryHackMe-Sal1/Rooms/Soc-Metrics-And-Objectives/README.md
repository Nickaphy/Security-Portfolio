## Scenario 1: Unhappy Customer

### Problem
*"One of our biggest customers, was dissatisfied with how we handle their breaches.
Their CFO's Email and Entra ID account were breached, it took us 6 hours to boot out the hacker
from the mailbox and the threat actors had enough time to dump all the emails and leak them on 
the Darknet. Looking at the report it seems we had a critical alert and spent 5 hours trying to reset the
victim's Entra ID password and MFA."*

### Cause
MTTR was too high, we caught the alert in a timely manner,
but containment of the breach was far too slow. That caused the victim's emails to be dumped and leaked on the darknet. 

### Recommended Fix
Our bottleneck has been determined to be our SOC team's  knowledge on how to properly perform credential rotations. We will assign the research and creation of a (credential rotation workbook) to the L2 analyst that handled the incident.

### Reasoning
If the team had a clear, precise workbook for credential rotation, MTTR on this class of breach would likely drop significantly, reducing the exposure window that allowed the mailbox dump. This fix assumes the bottleneck was knowledge, not access/permissions - if analysts lacked the actual privileges to contain the breach quickly, a workbook alone wouldn't fix the problem, and we would need to address access instead.

--- 

## Scenario 2: Delayed Alerts
### Problem
*We ran a SOC demo for our top management. They loved the ransomware simulation and were shocked at how our team managed to stop the attack
in 40 minutes. However, for the first 20 minutes, the team was just looking at the screen, waiting for some alerts to appear. They would like to 
see if we could somehow reduce this delay.*

### Cause
The root cause of the abnormally large dwell time is that we have our SIEM configured to run
its detection rules on 20-minute intervals.

### Recommended Fix
The solution in our case is to tell our SOC engineer to tune our SIEM to run its detection rules at 5-minute intervals.
This would lower our average MTTD by 15 minutes.

### Reasoning
In our case here the team sat ready to triage any incoming alerts, and did so quickly and efficiently when one came.
Our bottleneck was the SIEM, which simply had too large detection intervals.
The reason lowering the detection intervals to 5 minutes is that a low MTTD can be crucial in high-risk attacks.
The faster we detect and mediate an attack, the lower chance of the attack causing any damage.

--- 
