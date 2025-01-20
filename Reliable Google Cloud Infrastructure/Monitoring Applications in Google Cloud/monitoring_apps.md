# Monitoring Applications in Google Cloud

### 1. Download a sample app from Github

1. In the Cloud Console, click Activate Cloud Shell.

2. If prompted, click Continue. Once connected to Cloud Shell, you should see that you are already authenticated and that the project is already set to your project ID.

3. Run the following command in Cloud Shell to confirm that you are authenticated. If prompted, click Authorize:

```bash
gcloud auth list
```

- Command output:

> Note: The gcloud command-line tool is the powerful and unified command-line tool in Google Cloud. It comes preinstalled in Cloud Shell. Among its features, gcloud offers tab completion in the shell. For more information, see [gcloud command-line tool overview](https://cloud.google.com/sdk/gcloud).

4. Run the following command to confirm that you are using the correct project for this lab:

```bash
gcloud config list project
```

- Command output

5. If the correct project is not listed, you can set it with this command:

```bash
gcloud config set project [PROJECT_ID]
```

- Command output:

6. To create a folder called gcp-logging, run the following command:

```bash
mkdir gcp-logging
```

7. Change to the folder you just created:

```bash
cd gcp-logging
```

8. Clone a simple Python Flask app from Github:

```bash
git clone https://GitHub.com/GoogleCloudPlatform/training-data-analyst.git
```

9. Change to the deploying-apps-to-gcp folder:

```bash
cd training-data-analyst/courses/design-process/deploying-apps-to-gcp
```

10. In Cloud Shell, click Open Editor.

11. Expand the gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp folder in the navigation pane, and then click main.py to open it.

12. Add the following import statement at the top of the file (line 2):

```bash
import googlecloudprofiler
```

> Note: Profiler allows you to monitor the resources your applications use. For more information, refer to the [Google Cloud Profiler documentation](https://cloud.google.com/profiler/docs/).

13. After the main() function, add the following code snippet to start Profiler (after line 11):

```py
try:
    googlecloudprofiler.start(verbose=3)
except (ValueError, NotImplementedError) as exc:
    print(exc)
```

- Profiler will continuously report application metrics. Your code should look like this:

14. You also have to add the Profiler library to your requirements.txt file. Open that file in the code editor and add the following:

```bash
google-cloud-profiler==3.0.6
protobuf==3.20.1
```

The file should look like this:

15. Profiler has to be enabled in the project. In Cloud Shell, enter the following command:

```bash
gcloud services enable cloudprofiler.googleapis.com
```

16. To test the program, enter the following command to build a Docker container of the image:

```bash
docker build -t test-python .
```

17. To run the Docker image, enter the following command:

```bash
docker run --rm -p 8080:8080 test-python
```

18. To see the program running, click Web Preview in the Google Cloud Shell toolbar. Then select Preview on port 8080.

The program should be displayed in a new browser tab.

19. In Cloud Shell, type Ctrl+C to stop the program.

### 2. Deploy an application to App Engine and examine the Cloud logs

1. In the Cloud Shell code editor, in the Explorer pane, select the gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp folder.

2. Click New File, and then name the file app.yaml.

3. Paste the following into the file you just created:

```bash
runtime: python39
```

4. Save your changes.

5. In a project, an App Engine application has to be created. This is done just once using the gcloud app create command and specifying the region where you want the app to be created. In Cloud Shell, type the following command:

```bash
gcloud app create --region=REGION
```

6. Now deploy your app with the following command:

```bash
gcloud app deploy --version=one --quiet
```

7. On the Google Cloud console title bar, type App Engine in the Search field, then click App Engine in the Products & Pages section.

8. Click App Engine > Dashboard. The upper-right corner of the dashboard should display a link to your application similar to this:

> Note: By default, the URL to an App Engine instance is in the form of `https://project-id/appspot.com`.

9. Click on the link to test your program.

10. Refresh your browser a few times to make some requests.

11. Return to the Console and click the App Engine > Versions.

12. In Diagnose column of the table click Logs.

13. The logs should indicate that Profiler has started and profiles are being generated. If you get to this point too quickly, wait a minute and click Refresh.

### 3. View Profiler information

1. On the Google Cloud console title bar, type Profiler in the Search field, then click Profiler in the Products & Pages section. The screen should look similar to this:

2. On the Navigation menu, click Compute Engine.

3. Click Create Instance to create a virtual machine.

4. Select the region to `Region`

5. When the VM is ready, click SSH to log in to it.

6. If prompted allow SSH-in-browser to connect to VMs, click Authorize.

7. You will generate some traffic to your App Engine app using the web testing tool called Apache Bench. Enter the following commands to install it:

```bash
sudo apt update
sudo apt install apache2-utils -y
```

8. Update <your-project-id> with your PROJECT_ID from connection details panel and enter the following command to generate some traffic to your App Engine application:

```bash
ab -n 1000 -c 10 https://<your-project-id>.appspot.com/
```

The command will make a thousand requests, 10 at a time, to your application.

9. When the requests are finished, on the Google Cloud console title bar, type Profiler in the Search field, then click Profiler in the Products & Pages section.

### 4. Explore Cloud Trace

1. Every request to your application is added to the Trace list. On the Google Cloud console title bar, type Trace Explorer in the Search field, then click Trace Explorer in the Products & Pages section.

2. Return to the SSH window where you entered the Apache Bench command previously.

3. Enter the ab command again:

```bash
ab -n 1000 -c 10 https://<your-project-id>.appspot.com/
```

You can also experiment with different values for the -n and -c parameters.

4. Repeat this a couple of times, and then return to the Trace Explorer page.

### 5. Monitor resources using Dashboards

1. On the Google Cloud console title bar, type Monitoring in the Search field, then click Monitoring (Infrastructure and application quality checks) in the Products & Pages section.

2. In the left pane, click Dashboards. Cloud Monitoring analyzes the resources used in your projects and generates some default dashboards for you. In this exercise you have used App Engine and Compute Engine virtual machines, so a table similar to the one shown below should be displayed:

3. Click on the App Engine dashboard, and then select your project name. A dashboard of pertinent information for your App Engine application will appear.

4. In the left pane, click Dashboards.

5. Click on the VM Instances dashboard, and then select your instance. A dashboard for your VM will appear.

> Note: If you don't see VM Instances right away, wait a minute and refresh your browser.

6. Alternatively, return to the Dashboards page and click the +Create Dashboard. Try to create a custom dashboard.

7. For New Dashboard Name, type the custom dashboard name you have chosen. You can continue with your custom dashboard by adding the charts.

### 6. Create uptime checks and alerts

1. In the left pane, click Uptime checks, and then click the + Create Uptime Check link at the top. Fill out the form as follows:

| Property        | Value                           |
| --------------- | ------------------------------- |
| Protocol        | HTTPS                           |
| Resource Type   | URL                             |
| Hostname        | `<your-project-id>`.appspot.com |
| Path            | /                               |
| Check Frequency | 1 minute                        |

2. Click Continue and in Review section enter `App Engine Uptime Check` in Title.

3. Click Test to verify that your uptime check can connect to the resource. When you see a green check mark everything can connect. Click Create.

4. In the Uptime checks page click on 3 vertical dots next to your uptime check and select Add alert policy.

5. In Notifications and name click on the drop down arrow next to Notification Channels, then click on Manage Notification Channels. A Notification channels page will open in a new tab.

6. Scroll down the page and click on ADD NEW for Email.

7. In the Create Email Channel dialog box, enter your personal email address in the Email Address field and a Display name.

8. Click on Save.

9. Go back to the previous tab. Click on Notification Channels again, then click on the Refresh icon to get the display name you mentioned in the previous step.

10. Now, select your Display name and click OK.

11. Name the alert policy as Uptime Check Alert.

12. Click Create Policy. The uptime check you configured takes a while for it to become active.

13. Return to the open App Engine tab in order to disable the application to see whether your uptime check and alerting policy work.

14. Click Settings.

15. Click Disable application. Follow the instructions to disable the application.

16. Return to the App Engine Dashboard and test the URL. It shouldn't work anymore.

17. Return to the tab that contains Monitoring, and then click Uptime checks. Your uptime check should be failing. If you get there too fast, wait a minute and click refresh.

18. Click Alerting. An incident should have been fired.

19. Check your email. You should get a message from Cloud Monitoring.

20. Return to App Engine Settings and re-enable your application.Then return to the Uptime checks page. The uptime check should be working again. If not, wait a minute and then click refresh.

21. Return to the Alerting page. Your incident should be resolved. As before, you might have to wait a minute and then click refresh.

22. Check your email again. You should get a second email indicating that the alert recovered.

23. To make sure you don't get any emails after the project is deleted, delete your alerting policy and then delete your notification channel. At the top of the Alerting page, click Edit Notification Channels.

24. Find your email address and click the trash can icon to delete it.

25. Now click Uptime checks and delete your App Engine Uptime check.
