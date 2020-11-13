# KubeCon 2020
##  Putting Apache Cassandra&reg; on Automatic with Kubernetes™

TODO UPDATE PICTURE
![OK](https://github.com/DataStax-Academy/kubecon-cassandra-workshop/blob/master/3-materials/images/00-screenplay.png?raw=true)

In this repository, you'll find everything for the Cassandra Kubernetes Workshop:
- Materials used during presentations
- Hands-on exercises

Feel free to bookmark this page for future reference!

## Materials and Communications

* [TODO Workshop recording](https://youtu.be/nRf2M4OjGpU)
* [TODO  Presentation](3-materials/presentation.pdf)
* [Discord chat](https://bit.ly/cassandra-workshop)
* [Q&A: community.datastax.com](https://community.datastax.com)

## Exercises

To follow along with the hands-on exercises during the workshop, you need to have a docker-ready machine with at least a 4-core + 8 GB RAM. **KubeCon attendees can request a training cloud instance** using [this link](https://kubecon2020.datastaxtraining.com/). Notice that training cloud instances will be available only during the workshop and to be terminated 24 hours later.

* If you are using **your own computer** or your own cloud node, please check the requirements and install the missing tools as explained [Here](https://github.com/DataStax-Academy/kubecon2020/blob/main/setup_local.md).
* If you are using **a cloud training instance** provided by DataStax, relax as we have you covered: prerequisites are installed already. Please do the very last steps as explained [there](./0-setup-your-cluster-datastax)

* IMPORTANT NOTE.  Everywhere in this repo you see <YOURADDRESS> replace with the URL for the instance you were given 

| Title  | Description
|---|---|
| **1 - Setting up and Monitoring Cassandra** | [Instructions](#1-Setting-up-and-Monitoring-Cassandra)  |
| **2 - Working with data** | [Instructions](#2-Working-with-data)  |
| **3 - Scaling up and down** | [Instructions](#3-Scaling-up-and-down)  |
| **4 - Running repairs** | [Instructions](#4-Running-repairs)  |
| **5 - Backing up and Restoring data** | [Instructions](#5-Backing-up-and-Restoring-data)  |

# 1. Setting up and Monitoring Cassandra

### ✅  Set up the K8ssandra Repo
```
helm repo add k8ssandra https://helm.k8ssandra.io/
helm repo update
```

### ✅  Install K8ssandra via Helm commands
```
helm install k8ssandra-tools k8ssandra/k8ssandra
helm install k8ssandra-cluster-a k8ssandra/k8ssandra-cluster \
     --set ingress.traefik.enabled=true \
     --set ingress.traefik.repair.host=repair.127.0.0.1.xip.io
```

### ✅  Monitor things as they come up
Navigate to <YOURADDRESS>:9090 to start seeing metrics

once everything is up run 
```
kubectl get pods
```

TODO add correct output.

# 2. Working with data

### Deploy Pet Clinic App
TODO 
Pet Clinic app Github: https://github.com/spring-petclinic/spring-petclinic-reactive

Install the Nginx Ingress controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

Make sure the Cassandra Operator is initialized by trying to extract the password.
```
kubectl get secret k8ssandra-superuser -o yaml | grep password | cut -d " " -f4 | base64 -d
```
If this command returns an error, wait a few seconds and try it again until you see the random password.

Add the Cassandra password to the PetClinic manifest by running the following sed script.
```
sed -i "s/asdfadsfasdfdasdfds/$(kubectl get secret k8ssandra-superuser -o yaml | grep password | cut -d " " -f4 | base64 -d)/g" petclinic.yaml
```

Deploy the PetClinic app by applying the manifest.
```
kubectl apply -f petclinic.yaml
```

Navigate to <YOURADDRESS>:8081/ to interact with the pet clinic app

# 3. Scaling up and down
### ✅  Get current running config
For many basic config options you can change values in the operator's cassdc.yaml file.  Next we will scale our cluster using this method.

First let's check what our current running values are using the `helm get manifest k8ssandra` command.  This command is used to expose all the current running values in the system. 

In the command line type `helm get manifest k8ssandra` and press enter.

Notice how each of the yaml files that make up the deployment is displayed here.

### ✅  Scale the cluster up
In the command line type the following
```
helm get manifest k8ssandra-cluster-a | grep size
``` 

Notice the value of `size: 1`.  This is the current number of cassandra nodes.  Next we are going to scale this up to 3 nodes. 


```
helm upgrade k8ssandra-cluster-a k8ssandra/k8ssandra-cluster --set size=3
helm get manifest k8ssandra-cluster-a | grep size
```

Notice the `size: 3` in the output

### ✅  Scale the cluster down
If there is a need to make a config change without needing to edit a file the `--set` flag can be used from the CLI. Run the following command:

```
helm upgrade k8ssandra-cluster-a k8ssandra/k8ssandra-cluster --set size=1
helm get manifest k8ssandra-cluster-a | grep size
```

Notice the `size: 1` in the output again.

# 4. Running repairs
Repairs are a critical anti-entropy operation in Cassandra.  In the past there were many custom solutions to manage them outside of your main Cassandra Installation.  In K8ssandra there is a tool called Reaper that eliminates the need for a custom solution.  Just like K8ssandra makes Cassandra setup easy, Reaper makes configuration of repairs even easier. 

Before we access the Reaper UI, look at the Traefik Ingress controller configuration. It's described in the [Reaper UI](https://k8ssandra.io/docs/topics/accessing-services/traefik/repair-ui/) topic, which links to the values.yaml file provided by k8ssandra. Recall that when we created the `k8ssandra-cluster` in a prior step, we referenced the preconfigured Traefik Ingress with --set flags:

```
helm install k8ssandra-cluster-a k8ssandra/k8ssandra-cluster \
     --set ingress.traefik.enabled=true \
     --set ingress.traefik.repair.host=repair.127.0.0.1.xip.io
```

With that configuration running, we can point our browser to the Reaper UI. Example:

http://repair.127.0.0.1.xip.io/webui

The Reaper documentation illustrates common tasks you can perform in the UI:

* [Check a cluster’s health](http://cassandra-reaper.io/docs/usage/health/)
* [Run a cluster repair](http://cassandra-reaper.io/docs/usage/single/)
* [Schedule a cluster repair](http://cassandra-reaper.io/docs/usage/schedule/)
* [Monitor Cassandra diagnostic events](http://cassandra-reaper.io/docs/usage/cassandra-diagnostics/) (beta - looking ahead to 4.0)

For more reading visit https://medium.com/rahasak/orchestrate-repairs-with-cassandra-reaper-26094bdb59f6

# 5. Backing up and Restoring data
TODO
