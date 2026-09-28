# GSP342 - Implement Cloud Security Fundamentals on Google Cloud: Challenge Lab

Google Cloud manual solution for the **Implement Cloud Security Fundamentals on Google Cloud: Challenge Lab** --- creating a custom IAM security role, creating and configuring a dedicated service account, deploying a private Kubernetes Engine cluster, and testing application deployment from the management jumphost.

> Replace the lab-specific values with the values shown in your own lab panel before following along. The names used in the commands below are examples for one lab session.

---

## Author

**Er. Aayush Shrestha (Er. Shinux)** - Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

[![Watch on YouTube](https://img.shields.io/badge/Watch_on_YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@Er.Shinux)

---

## Lab Overview

This challenge lab has five tasks:

1. **Task 1:** Create a custom security role with the required Cloud Storage permissions.
2. **Task 2:** Create a dedicated service account for the private Kubernetes Engine cluster.
3. **Task 3:** Bind the required built-in monitoring/logging roles and the custom security role to the service account.
4. **Task 4:** Create and configure a private Kubernetes Engine cluster in the Orca Build VPC and subnet.
5. **Task 5:** Connect to the private cluster from the management jumphost and deploy a test application.

> **Important:** This is a **manual solution**. You do not need to download or execute any GitHub script. Use the values provided by your own lab session.

---

## ⚠️ About Your Lab Variables

The resource names in a challenge lab can change between sessions. Set the variables at the beginning of the Cloud Shell session using the values shown in your lab panel.

The following values are used in this guide as an example:

```bash
export CUSTOM_SECURIY_ROLE=orca_storage_editor_208
export SERVICE_ACCOUNT=orca-private-cluster-712-sa
export CLUSTER_NAME=orca-cluster-361
export VPC_NAME=orca-build-vpc
export SUBNET_NAME=orca-build-subnet
export JUMPHOST_NAME=orca-jumphost
export ZONE=us-central1-a
```

---

# TASK 1 - Create a Custom Security Role

## Step 1 - Set the Compute Zone

Run the following command:

```bash
gcloud config set compute/zone $ZONE
```

---

## Step 2 - Create the Custom Role Definition

Create the role definition file:

```bash
vi role-definition.yaml
```

Add the following content:

```yaml
title: "Custom-Security-Role"
description: "Permissions"
stage: "ALPHA"
includedPermissions:
  - storage.buckets.get
  - storage.objects.get
  - storage.objects.list
  - storage.objects.update
  - storage.objects.create
```

Save and exit the file.

The custom role provides the required permissions to access and create/update objects in Google Cloud Storage.

---

## Step 3 - Create the Custom IAM Role

Run:

```bash
gcloud iam roles create $CUSTOM_SECURIY_ROLE --project $DEVSHELL_PROJECT_ID --file role-definition.yaml
```

The role will be created in your current project.

> **Important:** Make sure `$CUSTOM_SECURIY_ROLE` matches the role name required by your lab session.

---

> ## Click "Check my progress" for Task 1 - Create a custom security role

---

# TASK 2 - Create a Service Account

## Step 4 - Create the Dedicated Service Account

Create the service account using:

```bash
gcloud iam service-accounts create $SERVICE_ACCOUNT --display-name "Orca Private Cluster Service Account"
```

This service account will later be attached to the private Kubernetes Engine cluster.

> **Important:** Use the service account name required by your lab session.

---

> ## Click "Check my progress" for Task 2 - Create a service account

---

# TASK 3 - Bind a Custom Security Role to a Service Account

The service account requires three built-in roles for monitoring and logging, as well as the custom Cloud Storage role created in Task 1.

## Step 5 - Grant the Monitoring Viewer Role

Run:

```bash
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
  --member=serviceAccount:$SERVICE_ACCOUNT@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
  --role roles/monitoring.viewer
```

---

## Step 6 - Grant the Monitoring Metric Writer Role

Run:

```bash
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
  --member=serviceAccount:$SERVICE_ACCOUNT@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
  --role roles/monitoring.metricWriter
```

---

## Step 7 - Grant the Logging Log Writer Role

Run:

```bash
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
  --member=serviceAccount:$SERVICE_ACCOUNT@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
  --role roles/logging.logWriter
```

---

## Step 8 - Bind the Custom Security Role

Run:

```bash
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
  --member serviceAccount:$SERVICE_ACCOUNT@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
  --role projects/$DEVSHELL_PROJECT_ID/roles/$CUSTOM_SECURIY_ROLE
```

The service account now has:

- `roles/monitoring.viewer`
- `roles/monitoring.metricWriter`
- `roles/logging.logWriter`
- Your custom Cloud Storage security role

---

> ## Click "Check my progress" for Task 3 - Bind the required roles

---

# TASK 4 - Create and Configure a New Kubernetes Engine Private Cluster

## Step 9 - Find the Internal IP of the Jumphost

The private cluster must allow the internal IP address of the management jumphost.

Run:

```bash
gcloud compute instances describe $JUMPHOST_NAME \
  --zone=$ZONE \
  --format="get(networkInterfaces[0].networkIP)"
```

The command will return an internal IP address, for example:

```text
192.168.10.2
```

> **Important:** The IP address shown above is only an example. **Use the actual IP returned by your command.**

---

## Step 10 - Create the Private Kubernetes Engine Cluster

Use the internal IP returned from the previous command in `--master-authorized-networks`.

For example, if the command returned `192.168.10.2`, run:

```bash
gcloud container clusters create $CLUSTER_NAME \
  --num-nodes 1 \
  --master-ipv4-cidr=172.16.0.64/28 \
  --network $VPC_NAME \
  --subnetwork $SUBNET_NAME \
  --enable-master-authorized-networks \
  --master-authorized-networks 192.168.10.2/32 \
  --enable-ip-alias \
  --enable-private-nodes \
  --enable-private-endpoint \
  --service-account $SERVICE_ACCOUNT@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
  --zone $ZONE
```

> **Note:** Cluster creation can take some time. Wait for the command to finish successfully before continuing.

---

> ## Click "Check my progress" for Task 4 - Create and configure a new Kubernetes Engine private cluster

---

# TASK 5 - Deploy an Application to a Private Kubernetes Engine Cluster

## Step 11 - Connect to the Jumphost

Connect to the management jumphost:

```bash
gcloud compute ssh --zone $ZONE $JUMPHOST_NAME
```

You should now be working inside the `orca-jumphost` instance.

---

## Step 12 - Set the Compute Zone

Inside the jumphost, run:

```bash
gcloud config set compute/zone us-central1-a
```

> **If your lab provides a different zone, use that zone instead.**

---

## Step 13 - Get Credentials for the Private Cluster

Because this is a private cluster with a private endpoint, retrieve its credentials using the `--internal-ip` flag:

```bash
gcloud container clusters get-credentials $CLUSTER_NAME --internal-ip
```

> **Important:** Do not omit `--internal-ip`. The cluster's private endpoint is accessed from the jumphost.

---

## Step 14 - Install the GKE Authentication Plugin

Run:

```bash
sudo apt-get update
```

Then install the GKE authentication plugin:

```bash
sudo apt-get install google-cloud-cli-gke-gcloud-auth-plugin
```

If prompted, confirm the installation.

> **Important:** The `gke-gcloud-auth-plugin` is required for continued use of `kubectl`.

---

## Step 15 - Deploy the Test Application

Create the deployment:

```bash
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```

This deploys the sample `hello-server` application.

---

## Step 16 - Expose the Application

Expose the deployment using a LoadBalancer service:

```bash
kubectl expose deployment hello-server \
  --name orca-hello-service \
  --type LoadBalancer \
  --port 80 \
  --target-port 8080
```

The application listens on port `8080`, while the service exposes it on port `80`.

---

> ## Click "Check my progress" for Task 5 - Deploy an application to a private Kubernetes Engine cluster

---

### ⚠️ Disclaimer

- **This guide and instructions are provided strictly for educational purposes to help you learn Google Cloud services and advance your engineering skills. Please review all commands thoroughly to understand the underlying infrastructure before execution. Always comply with Google Cloud Skills Boost Terms of Service and YouTube Community Guidelines. This material is designed to enhance hands-on learning, not to bypass lab challenges.**

### © Credit & Attribution

- **All educational content, lab scenarios, and original resources belong to [Google Cloud Skills Boost](https://www.cloudskillsboost.google/). No copyright infringement is intended. If you are a copyright owner and have concerns, please reach out via direct message for proper attribution or immediate content removal.** 🙏
