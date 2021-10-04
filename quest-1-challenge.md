# Task 1: Create a project jumphost instance

```
gcloud compute instances create nucleus-jumphost \
--zone us-east1-b \
--machine-type f1-micro \
--image-family debian-9 \
--image-project debian-cloud

```

# Task 3: Set up an HTTP load balancer

```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

```
gcloud compute instance-templates create web-server-template \
--region=us-east1 \
--network=default \
--subnet=default \
--tags=allow-health-check \
--machine-type g1-small \
--metadata-from-file startup-script=startup.sh
```

```
gcloud compute instance-groups managed create web-server-group \
   --template=web-server-template --size=2 \
   --zone us-east1-b
```

```
gcloud compute firewall-rules create web-server-firewall \
    --network=default \
    --allow tcp:80
```

```
gcloud compute http-health-checks create http-basic-check
```

```
gcloud compute instance-groups managed \
          set-named-ports web-server-group \
          --named-ports http:80 \
          --zone us-east1-b
```

```
gcloud compute backend-services create web-server-backend \
          --protocol HTTP \
          --http-health-checks http-basic-check \
          --global
```

```
gcloud compute backend-services add-backend web-server-backend \
          --instance-group web-server-group \
          --instance-group-zone us-east1-b \
          --global
```

```
gcloud compute url-maps create web-server-map \
          --default-service web-server-backend
```

```
gcloud compute target-http-proxies create http-lb-proxy \
          --url-map web-server-map
```

```
gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
```

```
gcloud compute forwarding-rules list
```
