# Hosting a Web App on Google Cloud Using Compute Engine

### 1. Enable Compute Engine API

```bash
gcloud services enable compute.googleapis.com
```

### 2. Create Cloud Storage bucket

```bash
gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID
```

### 3. Clone source repository

Clone the source code and then navigate

```bash
git clone https://github.com/googlecodelabs/monolith-to-microservices.git
```

```bash
cd ~/monolith-to-microservices
```

Run the initial build of the code to allow the application to run locally

```bash
./setup.sh
```

Ensure Cloud Shell is running a compatible nodeJS version

```bash
nvm install --lts
```

Next, run the following to test the application, switch to the microservices directory, and start the web server

```bash
cd microservices
npm start
```

### 4. Create Compute Engine instances

Create the startup script

```bash
touch ~/monolith-to-microservices/startup-script.sh
```

Click Open Editor in the Cloud Shell ribbon to open the Code Editor. Navigate to the monolith-to-microservices folder.

Add the following code to the startup-script.sh file.

> Note: Find the text [DEVSHELL_PROJECT_ID] in the file and replace it with your Project ID

```bash
#!/bin/bash

# Install logging monitor. The monitor will automatically pick up logs sent to
# syslog.
curl -s "https://storage.googleapis.com/signals-agents/logging/google-fluentd-install.sh" | bash
service google-fluentd restart &

# Install dependencies from apt
apt-get update
apt-get install -yq ca-certificates git build-essential supervisor psmisc

# Install nodejs
mkdir /opt/nodejs
curl https://nodejs.org/dist/v16.14.0/node-v16.14.0-linux-x64.tar.gz | tar xvzf - -C /opt/nodejs --strip-components=1
ln -s /opt/nodejs/bin/node /usr/bin/node
ln -s /opt/nodejs/bin/npm /usr/bin/npm

# Get the application source code from the Google Cloud Storage bucket.
mkdir /fancy-store
gsutil -m cp -r gs://fancy-store-[DEVSHELL_PROJECT_ID]/monolith-to-microservices/microservices/* /fancy-store/

# Install app dependencies.
cd /fancy-store/
npm install

# Create a nodeapp user. The application will run as this user.
useradd -m -d /home/nodeapp nodeapp
chown -R nodeapp:nodeapp /opt/app

# Configure supervisor to run the node app.
cat >/etc/supervisor/conf.d/node-app.conf << EOF
[program:nodeapp]
directory=/fancy-store
command=npm start
autostart=true
autorestart=true
user=nodeapp
environment=HOME="/home/nodeapp",USER="nodeapp",NODE_ENV="production"
stdout_logfile=syslog
stderr_logfile=syslog
EOF

supervisorctl reread
supervisorctl update
```

Save the startup-script.sh file, but do not close it yet.

> NOte: Look at the bottom right of Cloud Shell Code Editor, and ensure "End of Line Sequence" is set to "LF" and not "CRLF".

Return to Cloud Shell Terminal and run the following to copy the startup-script.sh

```bash
gsutil cp ~/monolith-to-microservices/startup-script.sh gs://fancy-store-$DEVSHELL_PROJECT_ID
```

Copy the cloned code into your bucket

```bash
cd ~
rm -rf monolith-to-microservices/*/node_modules
gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
```

Deploy the backend instance

```bash
gcloud compute instances create backend \
    --zone=$ZONE \
    --machine-type=e2-standard-2 \
    --tags=backend \
   --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
```

Configure a connection to the backend

```bash
gcloud compute instances list
```

Copy the External IP for the backend. In the Cloud Shell Explorer, navigate to monolith-to-microservices > react-app.

> Note: In the Code Editor, select View > Toggle Hidden Files in order to see the .env file.

In the .env file, replace localhost with your [BACKEND_ADDRESS]

```bash
REACT_APP_ORDERS_URL=http://[BACKEND_ADDRESS]:8081/api/orders
REACT_APP_PRODUCTS_URL=http://[BACKEND_ADDRESS]:8082/api/products
```

In Cloud Shell, run the following to rebuild react-app, which will update the frontend code

```bash
cd ~/monolith-to-microservices/react-app
npm install && npm run-script build
```

Then copy the application code into the Cloud Storage bucket

```bash
cd ~
rm -rf monolith-to-microservices/*/node_modules
gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
```

Deploy the frontend instance

```bash
gcloud compute instances create frontend \
    --zone=$ZONE \
    --machine-type=e2-standard-2 \
    --tags=frontend \
    --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
```

Create firewall rules to allow access to port 8080 for the frontend, and ports 8081-8082 for the backend.

```bash
gcloud compute firewall-rules create fw-fe \
    --allow tcp:8080 \
    --target-tags=frontend
```

```bash
gcloud compute firewall-rules create fw-be \
    --allow tcp:8081-8082 \
    --target-tags=backend
```

```bash
gcloud compute instances list
```

### 5. Create managed instance groups

First, stop both instances

```bash
gcloud compute instances stop frontend --zone=$ZONE
```

```bash
gcloud compute instances stop backend --zone=$ZONE
```

Then, create the instance template from each of the source instances

```bash
gcloud compute instance-templates create fancy-fe \
    --source-instance-zone=$ZONE \
    --source-instance=frontend
```

```bash
gcloud compute instance-templates create fancy-be \
    --source-instance-zone=$ZONE \
    --source-instance=backend
```

Confirm the instance templates were created

```bash
gcloud compute instance-templates list
```

With the instance templates created, delete the backend vm to save resource space

```bash
gcloud compute instances delete backend --zone=$ZONE
```

Next, create two managed instance groups, one for the frontend and one for the backend

```bash
gcloud compute instance-groups managed create fancy-fe-mig \
    --zone=$ZONE \
    --base-instance-name fancy-fe \
    --size 2 \
    --template fancy-fe
```

```bash
gcloud compute instance-groups managed create fancy-be-mig \
    --zone=$ZONE \
    --base-instance-name fancy-be \
    --size 2 \
    --template fancy-be
```

For your application, the frontend microservice runs on port 8080, and the backend microservice runs on port 8081 for orders and port 8082 for products

```bash
gcloud compute instance-groups set-named-ports fancy-fe-mig \
    --zone=$ZONE \
    --named-ports frontend:8080
```

```bash
gcloud compute instance-groups set-named-ports fancy-be-mig \
    --zone=$ZONE \
    --named-ports orders:8081,products:8082
```

Create a health check that repairs the instance if it returns "unhealthy" 3 consecutive times for the frontend and backend

```bash
gcloud compute health-checks create http fancy-fe-hc \
    --port 8080 \
    --check-interval 30s \
    --healthy-threshold 1 \
    --timeout 10s \
    --unhealthy-threshold 3
```

```bash
gcloud compute health-checks create http fancy-be-hc \
    --port 8081 \
    --request-path=/api/orders \
    --check-interval 30s \
    --healthy-threshold 1 \
    --timeout 10s \
    --unhealthy-threshold 3
```

Create a firewall rule to allow the health check probes to connect to the microservices on ports 8080-8081

```bash
gcloud compute firewall-rules create allow-health-check \
    --allow tcp:8080-8081 \
    --source-ranges 130.211.0.0/22,35.191.0.0/16 \
    --network default
```

Apply the health checks to their respective services

```bash
gcloud compute instance-groups managed update fancy-fe-mig \
    --zone=$ZONE \
    --health-check fancy-fe-hc \
    --initial-delay 300
```

```bash
gcloud compute instance-groups managed update fancy-be-mig \
    --zone=$ZONE \
    --health-check fancy-be-hc \
    --initial-delay 300
```

### 6. Create load balancers

Create health checks that will be used to determine which instances are capable of serving traffic for each service

```bash
gcloud compute http-health-checks create fancy-fe-frontend-hc \
  --request-path / \
  --port 8080
```

```bash
gcloud compute http-health-checks create fancy-be-orders-hc \
  --request-path /api/orders \
  --port 8081
```

```bash
gcloud compute http-health-checks create fancy-be-products-hc \
  --request-path /api/products \
  --port 8082
```

Create backend services that are the target for load-balanced traffic. The backend services will use the health checks and named ports you created

```bash
gcloud compute backend-services create fancy-fe-frontend \
  --http-health-checks fancy-fe-frontend-hc \
  --port-name frontend \
  --global
```

```bash
gcloud compute backend-services create fancy-be-orders \
  --http-health-checks fancy-be-orders-hc \
  --port-name orders \
  --global
```

```bash
gcloud compute backend-services create fancy-be-products \
--http-health-checks fancy-be-products-hc \
--port-name products \
--global
```

Add the Load Balancer's backend services

```bash
gcloud compute backend-services add-backend fancy-fe-frontend \
  --instance-group-zone=$ZONE \
  --instance-group fancy-fe-mig \
  --global
```

```bash
gcloud compute backend-services add-backend fancy-be-orders \
  --instance-group-zone=$ZONE \
  --instance-group fancy-be-mig \
  --global
```

```bash
gcloud compute backend-services add-backend fancy-be-products \
  --instance-group-zone=$ZONE \
  --instance-group fancy-be-mig \
  --global
```

Create a URL map. The URL map defines which URLs are directed to which backend services

```bash
gcloud compute url-maps create fancy-map \
  --default-service fancy-fe-frontend
```

Create a path matcher to allow the /api/orders and /api/products paths to route to their respective services

```bash
gcloud compute url-maps add-path-matcher fancy-map \
   --default-service fancy-fe-frontend \
   --path-matcher-name orders \
   --path-rules "/api/orders=fancy-be-orders,/api/products=fancy-be-products"
```

Create the proxy which ties to the URL map

```bash
gcloud compute target-http-proxies create fancy-proxy \
  --url-map fancy-map
```

Create a global forwarding rule that ties a public IP address and port to the proxy

```bash
gcloud compute forwarding-rules create fancy-http-rule \
  --global \
  --target-http-proxy fancy-proxy \
  --ports 80
```

In Cloud Shell, change to the react-app folder which houses the .env file that holds the configuration

```bash
cd ~/monolith-to-microservices/react-app/
```

Find the IP address for the Load Balancer

```bash
gcloud compute forwarding-rules list --global
```

Return to the Cloud Shell Editor and edit the .env file again to point to Public IP of Load Balancer. [LB_IP] represents the External IP address of the backend instance determined above

```bash
REACT_APP_ORDERS_URL=http://[LB_IP]/api/orders
REACT_APP_PRODUCTS_URL=http://[LB_IP]/api/products
```

Rebuild react-app, which will update the frontend code

```bash
cd ~/monolith-to-microservices/react-app
npm install && npm run-script build
```

Copy the application code into your bucket

```bash
cd ~
rm -rf monolith-to-microservices/*/node_modules
gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
```

Update the frontend instances

```bash
gcloud compute instance-groups managed rolling-action replace fancy-fe-mig \
    --zone=$ZONE \
    --max-unavailable 100%
```

Test the website

```bash
watch -n 2 gcloud compute backend-services get-health fancy-fe-frontend --global
```

### 7. Scaling Compute Engine

To create the autoscaling policy, execute the following

```bash
gcloud compute instance-groups managed set-autoscaling \
  fancy-fe-mig \
  --zone=$ZONE \
  --max-num-replicas 2 \
  --target-load-balancing-utilization 0.60
```

```bash
gcloud compute instance-groups managed set-autoscaling \
  fancy-be-mig \
  --zone=$ZONE \
  --max-num-replicas 2 \
  --target-load-balancing-utilization 0.60
```

Enable content delivery network

```bash
gcloud compute backend-services update fancy-fe-frontend \
    --enable-cdn --global
```

### 8. Update the website

Run the following command to modify the machine type of the frontend instance

```bash
gcloud compute instances set-machine-type frontend \
  --zone=$ZONE \
  --machine-type e2-small
```

Create the new Instance Template

```bash
gcloud compute instance-templates create fancy-fe-new \
    --region=$REGION \
    --source-instance=frontend \
    --source-instance-zone=$ZONE
```

Roll out the updated instance template to the Managed Instance Group

```bash
gcloud compute instance-groups managed rolling-action start-update fancy-fe-mig \
  --zone=$ZONE \
  --version template=fancy-fe-new
```

Wait 3 minutes, and then run the following to monitor the status of the update

```bash
watch -n 2 gcloud compute instance-groups managed list-instances fancy-fe-mig \
  --zone=$ZONE
```

Run the following to see if the virtual machine is using the new machine type (e2-small), where [VM_NAME] is the newly created instance

```bash
gcloud compute instances describe [VM_NAME] --zone=$ZONE | grep machineType
```

Run the following commands to copy the updated file to the correct file name

```bash
cd ~/monolith-to-microservices/react-app/src/pages/Home
mv index.js.new index.js
```

Print the file contents to verify the changes

```bash
cat ~/monolith-to-microservices/react-app/src/pages/Home/index.js
```

Run the following command to build the React app and copy it into the monolith public directory

```bash
cd ~/monolith-to-microservices/react-app
npm install && npm run-script build
```

Then re-push this code to the bucket

```bash
cd ~
rm -rf monolith-to-microservices/*/node_modules
gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
```

Now force all instances to be replaced to pull the update

```bash
gcloud compute instance-groups managed rolling-action replace fancy-fe-mig \
  --zone=$ZONE \
  --max-unavailable=100%
```

Wait 3 minutes after issuing the rolling-action replace command in order to give the instances time to be processed, and then check the status of the managed instance group.

```bash
watch -n 2 gcloud compute backend-services get-health fancy-fe-frontend --global
```

Browse to the website via http://[LB_IP] where [LB_IP] is the IP_ADDRESS specified for the Load Balancer, which can be found with the following command

```bash
gcloud compute forwarding-rules list --global
```

Simulate failure

```bash
gcloud compute instance-groups list-instances fancy-fe-mig --zone=$ZONE
```

Copy an instance name, then run the following to secure shell into the instance, where INSTANCE_NAME is one of the instances from the list

```bash
gcloud compute ssh [INSTANCE_NAME] --zone=$ZONE
```

Within the instance, use supervisorctl to stop the application

```bash
sudo supervisorctl stop nodeapp; sudo killall node
```

Monitor the repair operations

```bash
watch -n 2 gcloud compute operations list \
--filter='operationType~compute.instances.repair.*'
```
