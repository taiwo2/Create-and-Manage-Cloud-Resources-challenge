# Create-and-Manage-Cloud-Resources-challenge

# Task 1: Create a project jumphost instance
Run command:

    gcloud compute instances create nucleus-jumphost-712 \
              --network nucleus-vpc \
              --zone europe-west4-a  \
              --machine-type e2-micro  \
              --image-family debian-11  \
              --image-project debian-cloud \
              --scopes cloud-platform \
              --no-address

# Task 2: Create a Kubernetes service cluster
Run command:

    gcloud container clusters create nucleus-backend \
              --num-nodes 1 \
              --network nucleus-vpc \
              --region europe-west4
    gcloud container clusters get-credentials nucleus-backend \
              --region europe-west4

    kubectl create deployment hello-server \
              --image=gcr.io/google-samples/hello-app:2.0

    kubectl expose deployment hello-server \
              --type=LoadBalancer \
              --port 8082

# Task 3: Set up an HTTP load balancer
Run command:

    cat << EOF > startup.sh
    #! /bin/bash
    apt-get update
    apt-get install -y nginx
    service nginx start
    sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
    EOF