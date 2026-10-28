# GSP330 - Implement DevOps Workflows in Google Cloud: Challenge Lab

[![Start Lab](https://img.shields.io/badge/Google%20Cloud-Start%20Lab-blue?style=for-the-badge&logo=googlecloud)](https://www.skills.google/course_templates/716/labs/647664)

This guide is a concise supplement for **GSP330: Implement DevOps Workflows in Google Cloud**. It summarizes the challenge and focuses on the additional configuration, checks, and deployment steps that support the lab instructions.

## Author

**Aayush Shrestha (Shinux)** - Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/)
\| **GitHub:** [aayush105](https://github.com/aayush105) \|
**LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

[![Watch on
YouTube](https://img.shields.io/badge/Watch_on_YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@Er.Shinux)

---

## Lab Overview

In this challenge lab, you build a CI/CD workflow for a sample Go application using Google Kubernetes Engine (GKE), Artifact Registry, Cloud Build, and the dedicated Git server provided by the lab.

The six tasks cover:

1. Creating the lab resources and GKE cluster.
2. Initializing the sample application's Git repository.
3. Creating development and production Cloud Build triggers.
4. Deploying the first application versions.
5. Updating and deploying the second versions.
6. Rolling production back to the first version.

Use the region, zone, project, image paths, and other values assigned by your active lab. The official lab instructions are available from the **Start Lab** link above.

---

## Lab Solution

### Task 1 - Create the lab resources

#### Step 1: Set the lab environment

In Cloud Shell, run the environment setup command shown in **Task 1 of the official lab instructions**. It sets the project ID and number, region, zone, Git server IP, and the `gcloud` defaults. Use the values assigned to your active lab.

#### Step 2, 3, 4: Enable APIs, grant access, and configure Git

Run the API, IAM, and Git setup commands from the official lab instructions.

#### Step 5: Create the Artifact Registry repository

```bash
gcloud artifacts repositories create my-repository \
  --repository-format=docker \
  --location="$REGION" \
  --description="Docker repository for the sample application"
```

#### Step 6: Create the GKE Standard cluster

The lab specifies a Regular release channel, three initial nodes, and autoscaling between two and six nodes.

```bash
export CLUSTER_NAME="hello-cluster"

gcloud container clusters create "$CLUSTER_NAME" \
  --zone="$ZONE" \
  --release-channel=regular \
  --enable-autoscaling \
  --num-nodes=3 \
  --min-nodes=2 \
  --max-nodes=6
```

#### Step 6.1: Connect to the cluster

```bash
gcloud container clusters get-credentials "$CLUSTER_NAME" \
  --zone="$ZONE" \
  --project="$PROJECT_ID"
```

#### Step 7: Create namespaces

```bash
kubectl create namespace prod
kubectl create namespace dev
```

Check the Task 1 progress indicator in the lab.

### Task 2: Connect to the Git Repository on the Git Server

#### Step 1, 2, 3, 4: Copy the sample application

Follow the official lab instructions to copy the sample application, replace the region/zone/version placeholders, initialize the repository, and push the `master` and `dev` branches. Those commands are intentionally not repeated here.

#### Step 5: Verify the branches

```bash
cd ~/sample-app
git branch
git ls-remote --heads origin
```

### Task 3: Create the Cloud Build Triggers

#### Step 1: Create the production trigger

Create the production and development triggers using the commands in **Task 3 of the official lab instructions**.

#### Step 2: Verify the triggers

```bash
gcloud builds triggers list --region="$REGION"
```

### Task 4: Deploy the First Versions of the Application

#### A: Build the first development deployment

##### Step 1: Inspect and copy the development image path

```bash
cd ~/sample-app
git checkout dev
grep -nE 'images:|image:|docker.pkg.dev|gcr.io|<version>' \
  cloudbuild-dev.yaml dev/deployment.yaml
```

Copy the complete image path from `cloudbuild-dev.yaml`, including its repository and `v1.0` tag. You will paste it into the development deployment manifest.

#### Step 2: Get the project ID

```bash
export PROJECT_ID="$(gcloud config get-value project)"
echo "$PROJECT_ID"
```

#### Step 2.1: Update and verify the development manifest

```bash
nano dev/deployment.yaml
```

In the container's `image:` field, replace `<todo>` with the image path copied from `cloudbuild-dev.yaml`. Replace any literal `PROJECT_ID` in the image name with the project ID printed in Step 2, keeping the repository and `v1.0` tag unchanged. Save and exit.

Verify the image reference:

```bash
grep -n 'image:' dev/deployment.yaml
```

Confirm it matches the development image path and contains neither `<todo>` nor the literal `PROJECT_ID`.

#### Step 3: Commit and push the development changes

```bash
cd ~/sample-app
git checkout dev
git add .
git commit -m "Deploy v1.0 on dev"
git push http://giteaadmin:GiteaPassword123@${GIT_SERVER_IP}:3000/giteaadmin/sample-app.git dev
```

#### Step 4: Run the development build

```bash
cd ~/sample-app
gcloud builds submit --config=cloudbuild-dev.yaml .
```

#### Step 5: Check Cloud Build History, then inspect the development deployment and pods:

```bash
kubectl get deployments -n dev
kubectl get pods -n dev
```

#### Step 6: Expose and verify the development deployment

Check the application's Dockerfile for the container port. The example below uses `8080`; change the target port if the Dockerfile specifies a different port.

```bash
kubectl expose deployment development-deployment \
  --type=LoadBalancer \
  --name=dev-deployment-service \
  --port=8080 \
  --target-port=8080 \
  -n dev
```

#### Step 7: Retrieve the external IP and verify the `/blue` endpoint:

```bash
kubectl get service dev-deployment-service -n dev
```

Wait for the deployments and services to become ready, then open `http://EXTERNAL_IP:8080/blue` using the corresponding service IP.

#### B: First production deployment

##### Step 1: Inspect and copy the production image path

```bash
git checkout master
grep -nE 'images:|image:|docker.pkg.dev|gcr.io|<version>' \
  cloudbuild.yaml prod/deployment.yaml
```

Copy the complete image path from `cloudbuild.yaml`, including its repository and `v1.0` tag. You will paste it into the production deployment manifest.

#### Step 2: Get the project ID

```bash
export PROJECT_ID="$(gcloud config get-value project)"
echo "$PROJECT_ID"
```

#### Step 2.1: Update and verify the production manifest

```bash
nano prod/deployment.yaml
```

In the container's `image:` field, replace `<todo>` with the image path copied from `cloudbuild.yaml`. Replace any literal `PROJECT_ID` in the image name with the project ID printed in Step 2, keeping the repository and `v1.0` tag unchanged. Save and exit.

Verify the image reference:

```bash
grep -n 'image:' prod/deployment.yaml
```

Confirm it matches the production image path and contains neither `<todo>` nor the literal `PROJECT_ID`.

#### Step 3: Commit and push the production changes

```bash
cd ~/sample-app
git checkout master
git add .
git commit -m "Deploy v1.0 on master"
git push http://giteaadmin:GiteaPassword123@${GIT_SERVER_IP}:3000/giteaadmin/sample-app.git master
```

#### Step 4: Run the production build

```bash
cd ~/sample-app
gcloud builds submit --config=cloudbuild.yaml .
```

##### Step 5: Check Cloud Build History, then inspect the production deployment and pods:

```bash
kubectl get deployments -n prod
kubectl get pods -n prod
```

#### 6. Expose and verify the production deployment

Check the Dockerfile for the application's container port. The example below uses `8080`; adjust `--target-port` if needed.

```bash
kubectl expose deployment production-deployment \
  --type=LoadBalancer \
  --name=prod-deployment-service \
  --port=8080 \
  --target-port=8080 \
  -n prod
```

#### Step 7: Retrieve the external IP and verify the `/blue` endpoint:

```bash
kubectl get service prod-deployment-service -n prod
```

Wait for the deployments and services to become ready, then open `http://EXTERNAL_IP:8080/blue` using the corresponding service IP.

### Task 5: Deploy the Second Versions of the Application

The second version adds a `/red` endpoint and updates the image tag to `v2.0` in both development and production.

#### A. Update the development version

##### Step 1: Switch to dev

```bash
cd ~/sample-app
git checkout dev
```

##### Step 2: Update `main.go`

```bash
nano main.go
```

Update `main()` so both endpoints are registered:

```go
func main() {
    http.HandleFunc("/blue", blueHandler)
    http.HandleFunc("/red", redHandler)
    http.ListenAndServe(":8080", nil)
}
```

##### Step 3: Add the red handler, keeping the existing `blueHandler` and required imports:

```go
func redHandler(w http.ResponseWriter, r *http.Request) {
    img := image.NewRGBA(image.Rect(0, 0, 100, 100))
    draw.Draw(img, img.Bounds(),
        &image.Uniform{color.RGBA{255, 0, 0, 255}},
        image.ZP, draw.Src)
    w.Header().Set("Content-Type", "image/png")
    png.Encode(w, img)
}
```

##### Step 4, 5 Update the development image tag

Edit both files:

```bash
nano cloudbuild-dev.yaml
nano dev/deployment.yaml
```

Change the image version from `v1.0` to `v2.0` in both files. Keep the full image path consistent; do not replace unrelated text indiscriminately.

Verify:

```bash
grep -nE 'v1.0|v2.0|docker.pkg.dev|gcr.io' \
  cloudbuild-dev.yaml dev/deployment.yaml
```

##### Step 6, 7: Commit, push, and build

```bash
cd ~/sample-app
git checkout dev
git add .
git commit -m "Deploy v2.0 on dev"
git push http://giteaadmin:GiteaPassword123@${GIT_SERVER_IP}:3000/giteaadmin/sample-app.git dev
```

```bash
cd ~/sample-app
gcloud builds submit --config=cloudbuild-dev.yaml .
```

##### Step 8, 9: Verify the development resources:

```bash
kubectl get deployments -n dev
kubectl get pods -n dev
kubectl get service dev-deployment-service -n dev
```

Open `http://EXTERNAL_IP:8080/red` using the development service IP and verify that the endpoint works after the rollout.

#### B. Update the production version

##### Step 1, 2, 3: Switch to master

```bash
git checkout master
```

Make the same `main.go` changes in the `master` branch: register `/red` and add `redHandler`, while preserving the existing code and required imports.

##### Step 4, 5: Update the production image tag

Edit:

```bash
nano cloudbuild.yaml
nano prod/deployment.yaml
```

Change the production image version from `v1.0` to `v2.0` in both files. Confirm the image paths match.

##### Step 6, 7: Commit, push, and build

```bash
cd ~/sample-app
git checkout master
git add .
git commit -m "Deploy v2.0 on master"
git push http://giteaadmin:GiteaPassword123@${GIT_SERVER_IP}:3000/giteaadmin/sample-app.git master
```

```bash
cd ~/sample-app
gcloud builds submit --config=cloudbuild.yaml .
```

##### Step 8, 9: Verify the production resources:

```bash
kubectl get deployments -n prod
kubectl get pods -n prod
kubectl get service prod-deployment-service -n prod
```

Open `http://EXTERNAL_IP:8080/red` using the production service IP and verify that the endpoint works. Allow a few minutes for the rollout and load balancer to update. Check the Task 5 progress indicator.

### Task 6. Roll Back the Production Deployment

The objective is to return production to `v1.0`. Since `/red` was added in `v2.0`, it should return `404 Not Found` after rollback.

##### Step 1: Inspect the current production image and container name

```bash
kubectl get deployment production-deployment -n prod \
  -o jsonpath='{.spec.template.spec.containers[*].name}{"\n"}'

kubectl get deployment production-deployment -n prod \
  -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
```

Record the actual container name and image path.

##### Step 2: Roll back to the v1.0 image

Use the same image path as the current production image, changing only the tag to `v1.0`.

```bash
kubectl set image deployment/production-deployment \
  CONTAINER_NAME=YOUR_CONFIRMED_IMAGE_PATH:v1.0 \
  -n prod
```

Replace `CONTAINER_NAME` with the actual container name from Step 1, and replace `YOUR_CONFIRMED_IMAGE_PATH` with the image path confirmed from the deployment. Do not enter placeholders literally.

Alternatively, edit `prod/deployment.yaml` to use the confirmed `v1.0` image and apply it:

```bash
kubectl apply -f prod/deployment.yaml -n prod
```

##### Step 3: Wait for the rollout

```bash
kubectl rollout status deployment/production-deployment -n prod
kubectl get deployment production-deployment -n prod
kubectl get pods -n prod
```

##### Step 4: Verify the production endpoints

Get the production service IP:

```bash
kubectl get service prod-deployment-service -n prod
```

Test `/red`:

```bash
curl -i http://EXTERNAL_IP:8080/red
```

Replace `EXTERNAL_IP` with the production service's actual external IP. The expected result is `404 Not Found`.

Confirm `/blue` still works:

```bash
curl -I http://EXTERNAL_IP:8080/blue
```

Check the final progress indicator in the lab.

### ⚠️ Disclaimer

- **This guide and instructions are provided strictly for educational purposes to help you learn Google Cloud services and advance your engineering skills. Please review all commands thoroughly to understand the underlying infrastructure before execution. Always comply with Qwiklabs/Google Cloud Skills Boost Terms of Service and YouTube Community Guidelines. This material is designed to enhance hands-on learning, not to bypass lab challenges.**

### © Credit & Attribution

- **All educational content, lab scenarios, and original resources belong to [Google Cloud Skills Boost](https://www.cloudskillsboost.google/). No copyright infringement is intended. If you are a copyright owner and have concerns, please reach out via direct message for proper attribution or immediate content removal.** 🙏
