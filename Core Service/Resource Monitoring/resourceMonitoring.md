# Resource Monitoring

### 1. Create a Cloud Monitoring workspace

Three VM instances have been created for you that you will monitor.

- In the Cloud Console, on the Navigation menu (Navigation menu icon), click Compute Engine > VM instances. Notice the nginxstack-1, nginxstack-2 and nginxstack-3 instances.

Create a Monitoring workspace

1. On the Google Cloud console title bar, type Monitoring in the Search field, then click Monitoring in the Products & Page section.

2. Wait for your workspace to be provisioned.

### 2. Custom dashboards

Create a dashboard

1. In the left pane, click Dashboards.

2. Click +Create Dashboard.

3. For New Dashboard Name, type My Dashboard.

Add a chart

1. Click Add Widget,and then select Line.

2. For Widget Title, give your chart a name (you can revise this before you save based on the selections you make).

3. Type CPU utilization or CPU usage in Metric field dropdown, Click VM Instance > Instance. Select CPU utilization or CPU usage and click Apply.

> Note: Uncheck Active if you are unable to find the CPU utilization

4. Click + Add Filter and explore the various options.

5. Click Apply to create the chart.

Metrics Explorer

1. In the left pane, click Metrics explorer.

2. Select a Metric from the dropdown.

3. Explore the various options and try to recreate the chart you created earlier.

### 3. Alerting policies

Create an alert and add the first condition

1. In the left pane, select Alerting.

2. Click + Create Policy.

3. Click on Select a metric dropdown. Uncheck the Active option.

4. Type VM Instance in filter by resource and metric name and click on VM Instance > Instance. Select CPU usage or CPU Utilization and click Apply.

5. Set Rolling windows to 1 min.

6. Click Next. Set Threshold position to Above Threshold and set 20 as your Threshold value.

Add a second condition

1. Click +ADD ALERT CONDITION.

2. Repeat the steps above to specify the second condition for this policy. For example, repeat the condition for a different instance. Click Next.

3. In Multi-condition trigger, select All conditions are met.

4. Click Next.

Configure notifications and finish the alerting policy

1. Click on the dropdown arrow next to Notification Channels, then click on Manage Notification Channels.

2. The Notification channels page will open in a new tab.

3. Scroll down the page and click on ADD NEW for Email.

4. Enter your personal email in the Email Address field and a Display name.

5. Click Save.

6. Go back to the previous Configure notifications and finalize alert tab.

7. Click on Notification Channels again, then click on the Refresh icon to get the display name you mentioned in the previous step. Click Notification Channels again if needed.

8. Now, select your Display name and click OK.

9. Enter a name of your choice in the Alert policy name field.

10. Click Next.

11. Review the alert and click Create Policy.

### 4. Resource groups

1. In the left pane, click Groups.

2. Click + Create Group.

3. Enter a name for the group. For example: VM instances

4. In the Criteria section, type nginx in the Value field below Contains.

5. Click DONE.

6. Click CREATE.

7. Review the dashboard Cloud Monitoring created for your group.

### 5. Uptime monitoring

1. In the Monitoring tab, click on Uptime Checks.

2. Click + Create Uptime Check.

3. Specify the following, and leave the remaining settings as their defaults:

| Property        | Value             |
| --------------- | ----------------- |
| Protocol        | HTTP              |
| Resource Type   | Instance          |
| Applies To      | Group             |
| Group           | Select your group |
| Check Frequency | 1 minute          |

4. Click on Continue to leave the other details to default. In Alert & Notification tab, select your Notification Channels from the dropdown.

5. Click Continue.

6. For Title, enter a name of your choice for the uptime check.

7. Click Test to verify that your uptime check can connect to the resource.

8. When you see a green check mark everything can connect. Click Create.

### 6. Disable the alert

1. Navigate to the Alerting section.

2. Click view all Alert policies, on your Policies details page click on the Enabled toggle at the top of the page.

3. You will be asked to confirm that you want to turn off the alerting policy - click Turn Off.
