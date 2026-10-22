# GSP321 - Develop your Google Cloud Network: Challenge Lab

[![Start
Lab](https://img.shields.io/badge/Start_Lab-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://www.skills.google/course_templates/625/labs/592706)

Google Cloud manual solution for **GSP321: Develop your Google Cloud
Network: Challenge Lab**. This guide covers VPC creation, a dual-NIC
bastion, Cloud SQL for WordPress, GKE setup, WordPress deployment,
uptime monitoring, and project access.

## Author

**Aayush Shrestha (Shinux)** - Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/)
\| **GitHub:** [aayush105](https://github.com/aayush105) \|
**LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

[![Watch on
YouTube](https://img.shields.io/badge/Watch_on_YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@Er.Shinux)

---

## Lab Overview

This challenge lab has nine tasks:

1.  **Task 1:** Create the development VPC and subnets.
2.  **Task 2:** Create the production VPC and subnets.
3.  **Task 3:** Create a bastion host connected to both VPCs.
4.  **Task 4:** Create and configure the Cloud SQL instance.
5.  **Task 5:** Create the GKE cluster.
6.  **Task 6:** Prepare Kubernetes secrets and storage.
7.  **Task 7:** Deploy WordPress and expose it through a LoadBalancer
    service.
8.  **Task 8:** Create an uptime check.
9.  **Task 9:** Grant Editor access to the additional engineer.

> **Important:** Use the lab-provided `REGION`, `ZONE`, project ID, and
> second user account. Replace placeholders before running commands. Use
> only the temporary lab account.

---

# Set the lab region and zone

```bash
export REGION=asia-east1
export ZONE=asia-east1-b
```

# TASK 1 - Create development VPC manually

## Step 1 - Create the VPC

```bash
gcloud compute networks create griffin-dev-vpc \
  --subnet-mode=custom
```

## Step 2 - Create the development subnets

```bash
gcloud compute networks subnets create griffin-dev-wp \
  --network=griffin-dev-vpc \
  --range=192.168.16.0/20 \
  --region=$REGION

gcloud compute networks subnets create griffin-dev-mgmt \
  --network=griffin-dev-vpc \
  --range=192.168.32.0/20 \
  --region=$REGION
```

Click **Check my progress** for Task 1.

---

# TASK 2 - Create production VPC manually

## Step 1 - Create the VPC

```bash
gcloud compute networks create griffin-prod-vpc \
  --subnet-mode=custom
```

## Step 2 - Create the production subnets

```bash
gcloud compute networks subnets create griffin-prod-wp \
  --network=griffin-prod-vpc \
  --range=192.168.48.0/20 \
  --region=$REGION

gcloud compute networks subnets create griffin-prod-mgmt \
  --network=griffin-prod-vpc \
  --range=192.168.64.0/20 \
  --region=$REGION
```

Click **Check my progress** for Task 2.

---

# TASK 3 - Create a bastion host

## Step 1 - Create the bastion with two network interfaces

```bash
gcloud compute instances create griffin-bastion \
  --zone=$ZONE \
  --machine-type=e2-medium \
  --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt \
  --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt \
  --tags=allow-ssh
```

## Step 2 - Allow SSH to the bastion

```bash
gcloud compute firewall-rules create griffin-dev-allow-ssh \
  --network=griffin-dev-vpc \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:22 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=allow-ssh

gcloud compute firewall-rules create griffin-prod-allow-ssh \
  --network=griffin-prod-vpc \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:22 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=allow-ssh
```

> For production environments, restrict SSH to trusted source IPs. Use
> the access rules required by the lab.

Verify the VM has both network interfaces:

```bash
gcloud compute instances describe griffin-bastion --zone=$ZONE
```

Click **Check my progress** for Task 3.

---

# TASK 4 - Create and configure Cloud SQL

## Step 1 - Create the MySQL instance

```bash
gcloud sql instances create griffin-dev-db \
  --database-version=MYSQL_8_0 \
  --region=$REGION \
  --tier=db-n1-standard-1
```

Wait for the instance to finish provisioning.

## Step 2 - Create the WordPress database and user

Connect using Cloud SQL Studio in the Google Cloud console, or the SQL
connection method available in the lab. Run:

```sql
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;
```

Click **Check my progress** for Task 4.

---

# TASK 5 - Create the Kubernetes cluster

```bash
gcloud container clusters create griffin-dev \
  --zone=$ZONE \
  --num-nodes=2 \
  --machine-type=e2-standard-4 \
  --network=griffin-dev-vpc \
  --subnetwork=griffin-dev-wp
```

Fetch credentials:

```bash
gcloud container clusters get-credentials griffin-dev --zone=$ZONE
kubectl get nodes
```

Click **Check my progress** for Task 5.

---

# TASK 6 - Prepare the Kubernetes cluster

## Step 1 - Copy the supplied configuration files

```bash
gsutil -m cp -r gs://spls/gsp321/wp-k8s .
cd wp-k8s
ls
```

## Step 2 - Configure the WordPress secrets and volume

```bash
nano wp-env.yaml
```

Set the database username to `wp_user` and password to
`stormwind_rules`. Ensure the required secrets and volume configuration
are present, then save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

Apply the file:

```bash
kubectl apply -f wp-env.yaml
```

## Step 3 - Create the Cloud SQL service account key and Kubernetes secret

```bash
gcloud iam service-accounts keys create key.json \
  --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com

kubectl create secret generic cloudsql-instance-credentials \
  --from-file=key.json
```

Click **Check my progress** for Task 6.

---

# TASK 7 - Create the WordPress deployment

## Step 1 - Get the Cloud SQL instance connection name

```bash
gcloud sql instances describe griffin-dev-db \
  --format="value(connectionName)"
```

Copy the returned connection name, which has the form
`PROJECT_ID:$REGION:griffin-dev-db`.

## Step 2 - Update the deployment configuration

```bash
nano wp-deployment.yaml
```

Replace `YOUR_SQL_INSTANCE` with the exact Cloud SQL instance connection
name. Save and exit.

## Step 3 - Deploy WordPress and create the service

```bash
kubectl apply -f wp-deployment.yaml
kubectl apply -f wp-service.yaml
```

Check the deployment, pods, and service:

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

Wait until the service's external IP is assigned. Open that IP in a
browser and confirm the WordPress site installer appears.

Click **Check my progress** for Task 7.

---

# TASK 8 - Enable monitoring

1.  Open **Monitoring → Uptime checks** in the Google Cloud console.
2.  Click **Create Uptime Check**.
3.  Name it `griffin-wordpress-uptime`.
4.  Select an HTTP check and configure it to check the WordPress
    service's external IP.
5.  Set the path to `/` and port to `80`, as appropriate.
6.  Create the uptime check and allow it to initialize.

Click **Check my progress** for Task 8.

---

# TASK 9 - Provide access for an additional engineer

## Step 1 - Find the second lab user

Use the second user account provided in the lab instructions or
credentials panel. Do not use your personal account.

## Step 2 - Grant the Editor role

Replace `SECOND_USER_EMAIL` with the second lab account email:

```bash
gcloud projects add-iam-policy-binding "$GOOGLE_CLOUD_PROJECT" \
  --member="user:SECOND_USER_EMAIL" \
  --role="roles/editor"
```

Click **Check my progress** for Task 9.

---

### ⚠️ Disclaimer

- **This guide and instructions are provided strictly for educational purposes to help you learn Google Cloud services and advance your engineering skills. Please review all commands thoroughly to understand the underlying infrastructure before execution. Always comply with Qwiklabs/Google Cloud Skills Boost Terms of Service and YouTube Community Guidelines. This material is designed to enhance hands-on learning, not to bypass lab challenges.**

### © Credit & Attribution

- **All educational content, lab scenarios, and original resources belong to [Google Cloud Skills Boost](https://www.cloudskillsboost.google/). No copyright infringement is intended. If you are a copyright owner and have concerns, please reach out via direct message for proper attribution or immediate content removal.** 🙏