# Deploy Your Website on Cloud Run

![Cloud Run](https://cdn.qwiklabs.com/Ry1hidHMw9wjyWNTvENYln0NxFFyBJQyt1bPC%2Fdp0Qc%3D "Cloud Run")

## Task 0. Configure gcloud and enable APIs

gcloud config set compute/region us-central1
gcloud services enable artifactregistry.googleapis.com \
    cloudbuild.googleapis.com \
    run.googleapis.com

## Task 1. Clone the source repository

git clone https://github.com/googlecodelabs/monolith-to-microservices.git

cd ~/monolith-to-microservices
./setup.sh

cd ~/monolith-to-microservices/monolith
npm start

## Task 2. Create a Docker container with Cloud Build

gcloud artifacts repositories create monolith-demo \
    --repository-format=docker \
    --location=us-central1 \
    --description="Demo for monolithic web app" \
    --async
gcloud auth configure-docker us-central1-docker.pkg.dev
gcloud builds submit --tag us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0

## Task 3. Deploy the container to Cloud Run

gcloud run deploy monolith --image us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0 --region us-central1
gcloud run services list

## Task 4. Create new revision with lower concurrency

gcloud run deploy monolith --image us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0 --region us-central1 --concurrency 1
gcloud run deploy monolith --image us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0 --region us-central1 --concurrency 80

## Task 5. Make changes to the website

cd ~/monolith-to-microservices/react-app/src/pages/Home
mv index.js.new index.js
cat ~/monolith-to-microservices/react-app/src/pages/Home/index.js

cd ~/monolith-to-microservices/react-app
npm run build:monolith

cd ~/monolith-to-microservices/monolith

export TAG=2.0.2
gcloud builds submit --tag us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:${TAG}

## Task 6. Update website with zero downtime

gcloud run deploy monolith --image us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:${TAG} --region us-central1
gcloud run services describe monolith --platform managed --region us-central1
gcloud beta run services list
