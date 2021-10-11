 
You have started a new role as a Junior Cloud Engineer for Jooli, Inc. You are expected to help manage the infrastructure at Jooli. Common tasks include provisioning resources for projects.

You are expected to have the skills and knowledge for these tasks, so step-by-step guides are not provided.

Some Jooli, Inc. standards you should follow:

Create all resources in the default region or zone, unless otherwise directed.

Naming normally uses the format team-resource; for example, an instance could be named nucleus-webserver1.

Allocate cost-effective resource sizes. Projects are monitored, and excessive resource use will result in the containing project's termination (and possibly yours), so plan carefully. This is the guidance the monitoring team is willing to share: unless directed, use f1-micro for small Linux VMs, and use n1-standard-1 for Windows or other applications, such as Kubernetes nodes.

TASK 1:
You will use this instance to perform maintenance for the project.

Requirements:

Name the instance nucleus-jumphost.
Use an f1-micro machine type.
Use the default image type (Debian Linux).

**Solution:**
1. Open Console, under Compute Engine go to VM Instances in Navigation Menu
2. The name of instance be nucleus-jumphost
3. Region be Default Region
4. Zone be Default Zone
5. Select series N1 type and then machine type be f1-micro.
6. Using the default image type (Debian Linux)
7. Clcik on Create

Task 2:
The team is building an application that will use a service running on Kubernetes. You need to:

Create a cluster (in the us-east1-b zone) to host the service.
Use the Docker container hello-app (gcr.io/google-samples/hello-app:2.0) as a place holder; the team will replace the container with their own work later.
Expose the app on port 8080.

**Solution:**
1. Activate Google Cloud Shell
2. Type This:
gcloud config set compute/zone us-east1-b
gcloud container clusters create nucleus-jumphost-webserver1
gcloud container clusters get-credentials nucleus-jumphost-webserver1
kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0
kubectl expose deployment hello-app --type=LoadBalancer --port 8080
kubectl get service

Task 3:
You will serve the site via nginx web servers, but you want to ensure that the environment is fault-tolerant. 
Create an HTTP load balancer with a managed instance group of 2 nginx web servers. 
Use the following code to configure the web servers; the team will replace this with their own configuration later.


cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

You need to:

1. Create an instance template.
2. Create a target pool.
3. Create a managed instance group.
4. Create a firewall rule to allow traffic (80/tcp).
5. Create a health check.
6. Create a backend service, and attach the managed instance group.
7. Create a URL map, and target the HTTP proxy to route requests to your URL map.
8. Create a forwarding rule.


**Solution:**
0. Paste this on console:

cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

1. Creating an instance template:

gcloud compute instance-templates create nginx-template \
--metadata-from-file startup-script=startup.sh

2. Creating a target pool:

gcloud compute target-pools create nginx-pool
Press 'n' and select us-east1 as region and us-east1-b as zone

3. Creating a managed instance group:

gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool 
gcloud compute instances list

4. Creating a firewall rule to allow traffic (80/tcp):

gcloud compute firewall-rules create www-firewall --allow tcp:80 
gcloud compute forwarding-rules create nginx-lb \
--region us-east1 \
--ports=80 \
--target-pool nginx-pool
gcloud compute forwarding-rules list

5. Creating a health check:
gcloud compute http-health-checks create http-basic-checkg
gcloud compute instance-groups managed \
set-named-ports nginx-group \
--named-ports http:80

6. Creating a backend service and attach the managed instance group:

gcloud compute backend-services create nginx-backend \
--protocol HTTP --http-health-checks http-basic-checkg --global
gcloud compute backend-services add-backend nginx-backend \
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global

7. Creating a URL map and target HTTP proxy to route requests to your URL map:

gcloud compute url-maps create web-map \
--default-service nginx-backend 
gcloud compute target-http-proxies create http-lb-proxy \
--url-map web-map

8. Creating a forwarding rule:

gcloud compute forwarding-rules create http-content-rule\
--global \
--target-http-proxy http-lb-proxy \
--ports 80 

Select global

gcloud compute forwarding-rules list
Wait for 5 mins!!
