SCENARIO 5: CLOUDWATCH CPU ALARM WITH EMAIL ALERT
=================================================

PROBLEM
A server's CPU reaches 100% at night and nobody notices until customers
complain. I need an automatic email alert.

SOLUTION
A CloudWatch alarm on the EC2 CPUUtilization metric. When CPU goes above
the limit, the alarm tells an SNS topic, which sends me an email.

WHAT I USED
- EC2 instance: monitor-server (Amazon Linux, t3.micro, 2 CPU cores)
- SNS topic: cpu-alert
- CloudWatch alarm: high-cpu-alarm (CPU >= 40%)
- Region: ap-southeast-2


STEP 1: CREATE THE EMAIL ALERT (SNS)
1. Console -> SNS -> Topics -> Create topic
2. Type: Standard. Name: cpu-alert. Create topic.
3. Inside the topic -> Create subscription
   Protocol: Email. Endpoint: my email address.
4. Open my email and click "Confirm subscription" (check spam).
5. SNS -> Subscriptions: status must say Confirmed.
Without confirming, no alert will ever arrive.


STEP 2: LAUNCH THE SERVER
1. EC2 -> Launch instance. Name: monitor-server.
2. Amazon Linux, t3.micro, my key pair.
3. Security group: SSH (22) from My IP.
4. Wait until the state is Running.


STEP 3: CREATE THE ALARM
1. CloudWatch -> Alarms -> Create alarm -> Select metric
2. EC2 -> Per-Instance Metrics -> tick CPUUtilization for my instance
   (match the Instance ID) -> Select metric
3. Statistic: Average
   Period: 5 MINUTES   (important, see the problem list below)
   Condition: Greater/Equal. Threshold: 40
4. Next -> Notification: In alarm -> Select an existing SNS topic
   -> cpu-alert
5. Alarm name: high-cpu-alarm -> Create alarm
State starts as Insufficient data, then becomes OK after a few minutes.

<img width="1365" height="631" alt="Screenshot 2026-11-20 111401" src="https://github.com/user-attachments/assets/aca2db6b-3f15-47b0-bf39-fd669e4243d4" />



STEP 4: MAKE THE CPU BUSY ON PURPOSE
Connect to the server:
   ssh -i "ebb.pem" ec2-user@<public-ip>

Commands:
   nproc                  -> number of CPU cores (mine: 2)
   yes > /dev/null &      -> start one CPU load; run it twice for 2 cores
   top                    -> watch the CPU live (press q to leave)
   jobs                   -> list the background jobs

Meaning of: yes > /dev/null &
   yes          prints the letter y forever (keeps a CPU core busy)
   > /dev/null  throws the output away so the screen does not fill up
   &            runs it in the background so I get my prompt back

In top I saw 0.0 idle, which means the CPU was about 100% busy.
Do not close the terminal while waiting.

<img width="1365" height="767" alt="Screenshot 2026-11-20 113006" src="https://github.com/user-attachments/assets/96b3fc11-252d-4d3a-8889-e2a3f15f9252" />



STEP 5: WAIT AND WATCH
1. CloudWatch checks every 5 minutes, so wait 5 to 15 minutes.
2. Refresh the alarm page: OK -> In alarm.
3. Check my email for the alert.

<img width="1365" height="767" alt="Screenshot 2026-11-20 115704" src="https://github.com/user-attachments/assets/3265fa97-d282-48d1-b8eb-ff2ab08b7d47" />

<img width="1365" height="767" alt="Screenshot 2026-11-20 115645" src="https://github.com/user-attachments/assets/132c6a17-beea-41eb-9cf2-21b482ddcabf" />



STEP 6: STOP THE LOAD
   killall yes        (if not found: pkill yes)
Check with top that no yes lines remain.
After 5 to 15 minutes the alarm goes back to OK.

<img width="1365" height="631" alt="Screenshot 2026-11-20 111401" src="https://github.com/user-attachments/assets/407d44a3-8d69-4956-9e5a-1d144824f229" />



PROBLEMS I FACED AND FIXES
1. Alarm stayed on "Insufficient data", graph empty.
   Cause: the alarm period was 1 minute, but basic EC2 monitoring sends
   data only every 5 minutes.
   Fix: Actions -> Edit -> change Period to 5 minutes -> Update alarm.
2. More yes processes than I started.
   Cause: old processes from an earlier attempt were still running.
   Fix: killall yes.


IF THE ALARM DOES NOT CHANGE, CHECK IN THIS ORDER
1. Period is 5 minutes and threshold is 40 (Greater/Equal).
2. The alarm uses the SAME Instance ID as my running server.
3. EC2 and CloudWatch are in the same AWS account and region.
4. The SNS subscription says Confirmed.
5. top shows high CPU (no idle).


KEY CONCEPTS
- Metric: a number measured over time (CPU)
- Alarm: watches a metric. States: OK, ALARM, INSUFFICIENT_DATA
- SNS: sends the alert by email or SMS
- Basic monitoring: every 5 minutes. Detailed: every 1 minute (extra cost)
- EC2 does not report memory or disk by default (needs CloudWatch agent)
- Alarm = alerts me. Dashboard = shows me graphs on one screen.


CLEAN UP (avoid charges)
1. killall yes
2. Delete the alarm
3. Delete the SNS topic
4. Terminate the EC2 instance


INTERVIEW ANSWER
"I created a CloudWatch alarm on an EC2 instance's CPUUtilization metric
with a 40% threshold, connected to an SNS topic that emails me. I tested
it by running CPU-heavy processes, watched the alarm go from OK to ALARM,
received the email, and confirmed it returned to OK after I stopped the
load. I also fixed an empty graph by matching the alarm period to the
5-minute basic monitoring interval."

RESUME BULLET
Configured Amazon CloudWatch alarms with SNS email notifications to
monitor EC2 CPU utilization, and tested them by simulating high CPU load.


SCREENSHOT CHECKLIST
[ ] 1 alarm settings, state OK
[ ] 2 top showing high CPU
[ ] 3 alarm in the In alarm state
[ ] 4 alert email
[ ] 5 alarm back to OK
