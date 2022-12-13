# Setup application insights

pip install opencensus-ext-logging

Follow the setps from 
"Exercise: Collecting Telemetry Data for an application deployed on AKS cluster"

# Deploy to VMSS

run setup-script.sh

Check if it is up and runing:
az vmss list-instance-connection-info --resource-group "acdnd-c4-project" --name "udacity-vmss"

```
{
  "instance 2": "20.237.205.143:50002",
  "instance 3": "20.237.205.143:50003"
}
```

Connect to one of the nodes:
```
ssh udacityadmin@20.237.205.143 -p 50002
```

Install python runtime:
```
sudo apt update
sudo apt install python3.7
python3 --version
sudo -H pip3 install --upgrade pip
```

Install Redis

```
wget https://download.redis.io/releases/redis-6.2.4.tar.gz
tar xzf redis-6.2.4.tar.gz
cd redis-6.2.4
make
redis-cli ping
```

Setup the app:

```
cd redis-6.2.4
git clone https://github.com/Penny79/py-performance.git azure-voting-app-redis
git checkout Deploy_to_VMSS
```

# AKS

create acr and push image
```
az acr create --resource-group acdnd-c4-project --name pennz79acr --sku Basic
az acr login --name pennz79acr
az acr show --name pennz79acr --query loginServer --output table
docker tag azure-vote-front:v1 pennz79acr.azurecr.io/azure-vote-front:v1
docker push pennz79acr.azurecr.io/azure-vote-front:v1
az acr repository list --name pennz79acr --output table
az aks update -n udacity-cluster -g acdnd-c4-project --attach-acr pennz79acr

az acr show --name pennz79acr --query loginServer --output table
```

```
# Deploy the application. Run the command below from the parent directory where the *azure-vote-all-in-one-redis.yaml* file is present. 
kubectl apply -f azure-vote-all-in-one-redis.yaml
kubectl set image deployment azure-vote-front azure-vote-front=pennz79acr.azurecr.io/azure-vote-front:v1

# Test the application at the External IP
# It will take a few minutes to come alive. 
kubectl get service

# Check the status of each node
kubectl get pods

# Define an autoscaler
kubectl autoscale deployment azure-vote-front --cpu-percent=50 --min=1 --max=10

az aks update --resource-group acdnd-c4-project --name udacity-cluster --update-cluster-autoscaler --cpu-percent=30 --min-count=1 --max-count=10
```

Produce load
```
 kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://20.245.227.104/; done"

 when failed - delete the pod again: kubectl delete pod load-generator
```
