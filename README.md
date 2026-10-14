# VMware vCenter (VCSA) Integration

vCenter cannot send webhooks — an alarm action can only send mail, fire an SNMP trap, or run a script on the appliance. This integration installs a small package on the vCenter Server Appliance, which the alarm runs to forward the alarm's details to Zenduty.

Detailed documentation is available [here](https://www.xurrent.com/imr-help/vcsa-integration-guide).

### In Zenduty

1. Copy the webhook URL of your vCenter integration (**Integrations → your vCenter integration → Configure**). It looks like `https://events.zenduty.com/integration/<account>/vcsa/<integration_key>/`.

### On the vCenter Server Appliance

2. Connect to the appliance over SSH and clone the repository:

   ```
   git clone https://github.com/xurrent/imr-VMware-integration.git
   cd imr-VMware-integration
   ```

3. Install the package. This installs the forwarder at `/usr/local/bin/vcsa2zenduty`:

   ```
   sudo rpm -i zenduty-vcsa-1.0.0-1.noarch.rpm
   ```

4. Open the configuration file and set `url` to the webhook URL you copied above, exactly as Zenduty generated it:

   ```
   sudo vi /etc/zenduty/conf/zenduty-integration.conf
   ```

### In the vSphere Client

5. Go to the object you want to monitor — a virtual machine, host, datastore, cluster, or the data center itself — and open **Configure → Alarm Definitions**.

6. Click **Add** to create a new alarm, or select an existing alarm and click **Edit**.

7. Give the alarm a name and set its trigger conditions as you normally would.

8. Under the alarm's actions, add an action, choose **Run script**, and enter the full path to the forwarder:

   ```
   /usr/local/bin/vcsa2zenduty
   ```

9. Enable the action on the status transitions you want to send. Each action has a column per transition — green to yellow, yellow to red, red to yellow, and yellow to green.

   > **Important:** enable the action on the recovery transitions (red to yellow and yellow to green) as well as the escalating ones. Zenduty resolves an incident when it receives the alarm's return to a healthy status. If the action only fires on the escalating transitions, incidents will open but never resolve on their own.

10. Save the alarm, then trigger it to test. The alert appears under **Alerts** on the Service in Zenduty within a few seconds. Return the object to a healthy state and the incident resolves on its own.

Repeat steps 5 to 10 for each alarm you want to send to Zenduty. The same forwarder is used by every alarm — you only install the package once.
