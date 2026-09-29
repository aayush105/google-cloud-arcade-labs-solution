# GSP313 - Implement Load Balancing on Compute Engine: Challenge Lab

Google Cloud manual solution for the **Implement Load Balancing on Compute Engine: Challenge Lab** --- creating multiple web server instances, configuring a regional network load balancer, and creating a global HTTP load balancer with a managed instance group.

> Replace the lab-specific values with the values shown in your own lab panel before following along. The values below match the lab session used for this guide.

---

## Author

**Er. Aayush Shrestha (Er. Shinux)** - Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

[![Watch on YouTube](https://img.shields.io/badge/Watch_on_YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@Er.Shinux)

---

## Lab Overview

This challenge lab has three tasks:

1. **Task 1:** Create three Compute Engine web server instances and allow HTTP traffic with a firewall rule.
2. **Task 2:** Configure a regional network load balancer for the web server instances.
3. **Task 3:** Create a global HTTP load balancer backed by a managed instance group.

> **Important:** This is a **manual solution**. You do not need to download or execute a script. Use the values provided by your own lab session.

## Challenge Scenario

You are a junior cloud engineer supporting network functionality for Compute Engine virtual machines in a Google Cloud VPC network. You need to create web servers and configure both a network load balancer and an HTTP load balancer to distribute traffic across the instances.

The lab uses the `default` network unless the instructions in your lab panel specify otherwise.

## Lab Variables

Set these variables at the beginning of the Cloud Shell session. Replace them if your lab provides different values.

```bash
export REGION=us-east4
export ZONE=us-east4-c
export NETWORK=default
```

---

# TASK 1 - Create Multiple Web Server Instances

Create three Compute Engine VM instances with Apache installed. Each instance uses the `network-lb-tag` network tag and serves a page containing its own name.

## Step 1 - Set the Compute Region and Zone

Run:

```bash
gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE
```

## Step 2 - Create the `web1` Instance

```bash
gcloud compute instances create web1 \
  --zone=$ZONE \
  --network=$NETWORK \
  --tags=network-lb-tag \
  --machine-type=e2-small \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web1</h3>" | tee /var/www/html/index.html'
```

## Step 3 - Create the `web2` Instance

```bash
gcloud compute instances create web2 \
  --zone=$ZONE \
  --network=$NETWORK \
  --tags=network-lb-tag \
  --machine-type=e2-small \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web2</h3>" | tee /var/www/html/index.html'
```

## Step 4 - Create the `web3` Instance

```bash
gcloud compute instances create web3 \
  --zone=$ZONE \
  --network=$NETWORK \
  --tags=network-lb-tag \
  --machine-type=e2-small \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web3</h3>" | tee /var/www/html/index.html'
```

## Step 5 - Create the Web Server Firewall Rule

The lab requires the firewall rule `www-firewall-network-lb` to allow HTTP traffic to instances with the `network-lb-tag` tag.

```bash
gcloud compute firewall-rules create www-firewall-network-lb \
  --network=$NETWORK \
  --action=ALLOW \
  --direction=INGRESS \
  --target-tags=network-lb-tag \
  --source-ranges=0.0.0.0/0 \
  --rules=tcp:80
```

## Step 6 - Verify Task 1

List the instances:

```bash
gcloud compute instances list
```

Get an external IP address:

```bash
gcloud compute instances describe web1 \
  --zone=$ZONE \
  --format="get(networkInterfaces[0].accessConfigs[0].natIP)"
```

Test the web server by replacing `IP_ADDRESS` with the returned address:

```bash
curl http://IP_ADDRESS
```

Repeat the test for `web2` and `web3` if needed. If an instance does not respond, wait for its startup script to finish or reset the VM before testing again.

> ## Click "Check my progress" for Task 1 - Create multiple web server instances

---

# TASK 2 - Configure the Load Balancing Service

Create a regional static external IP address, an HTTP health check, a target pool, and a regional forwarding rule.

## Step 7 - Reserve the Regional Static External IP

```bash
gcloud compute addresses create network-lb-ip-1 \
  --region=$REGION
```

## Step 8 - Create the HTTP Health Check

```bash
gcloud compute http-health-checks create basic-check
```

## Step 9 - Create the Target Pool

```bash
gcloud compute target-pools create www-pool \
  --region=$REGION \
  --http-health-check=basic-check
```

## Step 10 - Add the Web Servers to the Target Pool

```bash
gcloud compute target-pools add-instances www-pool \
  --instances=web1,web2,web3 \
  --instances-zone=$ZONE \
  --region=$REGION
```

## Step 11 - Create the Regional Forwarding Rule

The forwarding rule listens on port `80` and sends traffic to `www-pool`.

```bash
gcloud compute forwarding-rules create www-rule \
  --region=$REGION \
  --ports=80 \
  --address=network-lb-ip-1 \
  --target-pool=www-pool
```

## Step 12 - Verify Task 2

Retrieve the network load balancer IP:

```bash
gcloud compute addresses describe network-lb-ip-1 \
  --region=$REGION \
  --format="get(address)"
```

Send requests to the forwarding rule by replacing `LOAD_BALANCER_IP`:

```bash
curl http://LOAD_BALANCER_IP
```

Run the command several times. Responses should be served by different web server instances as traffic is distributed by the target pool.

> ## Click "Check my progress" for Task 2 - Configure the load balancing service

---

# TASK 3 - Create an HTTP Load Balancer

Create a zonal managed instance group and connect it to a global external HTTP load balancer. The backend instances serve the name of the instance that handled the request.

> The managed instance group is zonal. The backend service, URL map, HTTP proxy, and forwarding rule are global resources.

## Step 13 - Create the Instance Template

The template uses the same Debian image family and image project as the Task 1 instances.

```bash
export REGION=us-east4
export ZONE=us-east4-c
export NETWORK=default
```

```bash
gcloud compute instance-templates create lb-backend-template \
  --machine-type=e2-medium \
  --network=$NETWORK \
  --tags=allow-health-check \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
vm_hostname="$(curl -H "Metadata-Flavor:Google" http://169.254.169.254/computeMetadata/v1/instance/name)"
echo "Page served from: $vm_hostname" | tee /var/www/html/index.html
systemctl restart apache2'
```

## Step 14 - Create the Managed Instance Group

```bash
gcloud compute instance-groups managed create lb-backend-group \
  --template=lb-backend-template \
  --size=2 \
  --zone=$ZONE
```

## Step 15 - Create the Health Check Firewall Rule

Allow Google Cloud health-check traffic from the required source ranges to instances tagged `allow-health-check`.

```bash
gcloud compute firewall-rules create fw-allow-health-check \
  --network=$NETWORK \
  --action=ALLOW \
  --direction=INGRESS \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
```

## Step 16 - Reserve the Global External IP Address

```bash
gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global
```

## Step 17 - Create the HTTP Health Check

```bash
gcloud compute health-checks create http http-basic-check \
  --port=80
```

## Step 18 - Create the Global Backend Service

```bash
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
```

## Step 19 - Add the Managed Instance Group to the Backend Service

```bash
gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=$ZONE \
  --global
```

## Step 20 - Set the Named Port

```bash
gcloud compute instance-groups managed set-named-ports lb-backend-group \
  --zone=$ZONE \
  --named-ports=http:80
```

## Step 21 - Create the URL Map

```bash
gcloud compute url-maps create web-map-http \
  --default-service=web-backend-service
```

## Step 22 - Create the Target HTTP Proxy

```bash
gcloud compute target-http-proxies create http-lb-proxy \
  --url-map=web-map-http
```

## Step 23 - Create the Global Forwarding Rule

```bash
gcloud compute forwarding-rules create http-content-rule \
  --load-balancing-scheme=EXTERNAL \
  --network-tier=PREMIUM \
  --address=lb-ipv4-1 \
  --global \
  --target-http-proxy=http-lb-proxy \
  --ports=80
```

## Step 24 - Verify Task 3

Check the managed instance group:

```bash
gcloud compute instance-groups managed list-instances \
  lb-backend-group \
  --zone=$ZONE
```

Check backend health:

```bash
gcloud compute backend-services get-health \
  web-backend-service \
  --global
```

Wait until the backend instances report `HEALTHY`.

Retrieve the HTTP load balancer IP:

```bash
gcloud compute addresses describe lb-ipv4-1 \
  --global \
  --format="get(address)"
```

Test the load balancer by replacing `LOAD_BALANCER_IP`:

```bash
curl http://LOAD_BALANCER_IP
```

The response should contain text similar to:

```text
Page served from: lb-backend-group-xxxx
```

It can take three to five minutes for the HTTP load balancer to become ready and route traffic to healthy backends.

> ## Click "Check my progress" for Task 3 - Create an HTTP load balancer

---

### ⚠️ Disclaimer

- **This guide and instructions are provided strictly for educational purposes to help you learn Google Cloud services and advance your engineering skills. Please review all commands thoroughly to understand the underlying infrastructure before execution. Always comply with Google Cloud Skills Boost Terms of Service and YouTube Community Guidelines. This material is designed to enhance hands-on learning, not to bypass lab challenges.**

### © Credit & Attribution

- **All educational content, lab scenarios, and original resources belong to [Google Cloud Skills Boost](https://www.cloudskillsboost.google/). No copyright infringement is intended. If you are a copyright owner and have concerns, please reach out via direct message for proper attribution or immediate content removal.** 🙏
