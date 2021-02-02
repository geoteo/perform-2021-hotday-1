## Failed SSH Attempts (Metrics API v2)

In this lab you'll learn how to ingest multiple metrics and dimensions more specifically a failed SSH login with the username and IP address via `Metrics API v2`. 

We're going to use the new endpoint `/api/v2/metrics/ingest​` to push locally retrieved metrics to Dynatrace over a secure and authenticated channel.

### Step 1: Find the metrics we want to feed into Dynatrace

1. Execute `sudo journalctl _SYSTEMD_UNIT=ssh.service` on your Linux host.

   ```bash
   (dtu.training@ip-10-0-0-X)$ sudo journalctl _SYSTEMD_UNIT=ssh.service
   ```

2. Open up another terminal and SSH into the host with the same user, but enter in a wrong password. 

3. Change the output to show failed SSH attempts that occurred in the last minute.

   ```bash
   (dtu.training@ip-10-0-0-X)$ sudo journalctl _SYSTEMD_UNIT=ssh.service --since "1min ago" | grep "Failed"
   ```

### Step 2. Feed custom metrics to Dynatrace via `Metrics API v2`

1. Provided data points must follow the metrics ingestion protocol. The general syntax of metric ingestion is the following:

   ```bash
   metric.key,dimensions payload
   ```

2. Change output to match format.

   ```bash
   (dtu.training@ip-10-0-0-X)$ sudo journalctl _SYSTEMD_UNIT=ssh.service --since "1min ago" | grep "Failed" | awk '{print "host.ssh.fail.sessions,user="$9",ip="$11" "1}'
   ```
3. Create an API Token with the following access scopes:
   -  API v2 -> Ingest Metrics

4. Create variable for data and curl `Metrics API v2`.
   - **NOTE:** Replace *TENANTID* with your tenant ID and *XXXXXXXXXXXXX* with your newly created API token.

   ```bash
   (dtu.training@ip-10-0-0-X)$ data=$(sudo journalctl _SYSTEMD_UNIT=ssh.service --since "1min ago" | grep "Failed" | awk '{print "host.ssh.fail.sessions,user="$9",ip="$11" "1}'); curl -d $data "https://TENANTID.live.dynatrace.com/api/v2/metrics/ingest" -H "Authorization: Api-Token XXXXXXXXXXXXX" -H "Content-Type: text/plain; charset=utf-8"
   ```

### Step 3. Automate the process! 

1. Run the following script with the edited command you made from our last step:

   ```bash
   while true; do data=$(sudo journalctl _SYSTEMD_UNIT=ssh.service --since "1min ago" | grep "Failed" | awk '{print "host.ssh.fail.sessions,user="$9",ip="$11" "1}'); curl -d "$data" "https://TENANTID.live.dynatrace.com/api/v2/metrics/ingest" -H "Authorization: Api-Token XXXXXXXXXXXXX" -H "Content-Type: text/plain; charset=utf-8"; sleep 60; done
   ```

### Step 4. Query Data in Data Explorer

1. Dynatrace UI -> Try it out -> host.ssh.fail.sessions -> Max -> Split by user, ip
