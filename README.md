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

# Create an instance template.
    gcloud compute instance-templates create web-server-template \
          --metadata-from-file startup-script=startup.sh \
          --network nucleus-vpc \
          --machine-type g1-small \
          --region us-east1


# Create a target pool and Create a managed instance group.
    gcloud compute instance-groups managed create web-server-group \
              --base-instance-name web-server \
              --size 2 \
              --template web-server-template \
              --region us-east1


# Create a firewall rule named as Firewall rule to allow traffic (80/tcp).
    gcloud compute firewall-rules create Firewall rule \
              --allow tcp:80 \
              --network nucleus-vpc

# Create a health check
    gcloud compute http-health-checks create http-basic-check

# Create a backend service, and attach the managed instance group with named port (http:80)
    gcloud compute instance-groups managed \
              set-named-ports web-server-group \
              --named-ports http:80 \
              --region us-east1


    gcloud compute backend-services create web-server-backend \
              --protocol HTTP \
              --http-health-checks http-basic-check \
              --global

    gcloud compute backend-services add-backend web-server-backend \
              --instance-group web-server-group \
              --instance-group-region us-east1 \
              --global

# Create a URL map, and target the HTTP proxy to route requests to your URL map.

    gcloud compute url-maps create web-server-map \
              --default-service web-server-backend

    gcloud compute target-http-proxies create http-lb-proxy \
              --url-map web-server-map

# Create a forwarding rule
    gcloud compute forwarding-rules create http-content-rule \
            --global \
            --target-http-proxy http-lb-proxy \
            --ports 80

    gcloud compute forwarding-rules list