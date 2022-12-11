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

```
   # Pull the base image
   FROM tiangolo/uwsgi-nginx-flask:python3.6
   # Install depndencies 
   RUN pip install redis
   RUN pip install opencensus
   RUN pip install opencensus-ext-azure
   RUN pip install opencensus-ext-flask
   RUN pip install flask
   # Copy the content of the current directory to the /app of the container
   ADD . /app  
```
