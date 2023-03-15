# Requirements
- minikube
- k9s
- kubectl
- vscode with kuberentes yaml extension
- docker

# Create your onw stuff

Lets create an app that can connect to a database and read you neighboards apis
This application needs to be production ready, for that reason we'll take all the precotions to make sure it's resilient and scalable.
## **Command Tips:**
```bash
  create deployment: kubectl create  deployment <deployment_name> --image=<image> --port=5000 --replicas=33 --dry-run=client -oyaml
  expose deployment: kubectl expose deployments.apps <name_of_deployment> --type=<type_of_expose> --dry-run=client -o yaml
  get logs: kubectl logs -c <pod_name> -n <name_space>
  auto scale: kubectl autoscale deploy/<you_deployment_name> --cpu-percent=<number> --min=<number> --max=<number>
 ```
#
## Creating a namespace
### Lets start by creating a namespace with your name and first letter of your lastname, example: leandrol


---
## Create the DB
### Create a deployment with a mysql database use the image: **mysql:5.7**
- The deployment doesn't need mutiple replicas, only one
- Make sure tha the deployment strategy is set to Recreate
- We need secrets to store the root password. Create a secret using `stringData` and attach that secret into your database deployment for creating the variable  `MYSQL_ROOT_PASSWORD`
- To persist data lets mount volume to the minibuke docker volume, using `hostPath` to mount `/mnt/data` and the mountPath in the pods is `/var/lib/mysql`
- Don't forget to expose(create a service) the deployment internally so our app can access this deployment later.

---

## Create the APP
### Create the app deployment with the following image: **landmeyer/kubernetes-tutorial-03:latest**
- Make sure it has 2 replicas 
### Adding Health Checks
- livenessProbe to make sure the app is healthy
- readinessProbe to check if the app has started, you can use this path /healthy

### Resource managment
Let's add resource managment to make sure the application doesn't use up all the resources in our cluster.
- Requests cpu: 100m memoery: 100Mi
- Limit cpu: 200 memoery: 200Mi

### Expose the app
- Create the app using Service object remeber to use selector
- Make sure the service is using the type nodePort
- tip: use the `kubectl expose` command to create the service and with dry-run put that into code.
- now to check if the app is running. Use `kubectl port forward` or `minikube service <your_service_name> --url`. This will expose your service so you can access from outside of the cluster.
  - to actually test if the app is running try hitting this path `/healthy`

### Adding the configMap and secrets
Let create a configMap that hosts the url to access the database. 
- The variable holding the database url should be called DB_URL
- Add two variable called MYSQL_ROOT_PASSWORD and MYSQL_ROOT_USER the value for these variables should be pulled from the same secret use in the Database Deployment. 

## RBAC
### For our application to work propperly it needs some access to our kubernetes cluster.
- We need to add a service account to our deployment
- Create a ClusterRole with the propper permissions they would be `get` and `list` with the verb  `pods`
- Attach this role you your serviceAccount using ClusterRoleBinding

## Side Cart
### Let's make sure the application only start once it can reach mysql.
- Using initContainer lets check if the db service is responding on port `3306`
- use busy box with the following command:
```yaml
command: ['sh', '-c', 'echo -e "Checking for the availability of MySQL Server deployment"; while ! nc -z mysql 3306; do sleep 1; printf "-"; done; echo -e "  >> MySQL DB Server has started";']
```
 **Make sure you adapt the mysql check command according with namespace and variables used to connect to the database**

## Application Check Paths

`/healthy` - simple health check to see if app is running.

`/get/<you_namespace_name>` - checks if your RBAC was setup properly.

`/ping` - Queries the database and tells you if a connection was stablished.

---
## Final Chalenge
### Auto-Scale
- Create your HPA object scaling more pods once cpu reaches 85% with a minimum of 2 pods and a max of 10
- After auto-scale is enabled lets test it with wrk
### Loadtest
To be able to loadtest we need to have metrics in our server so first things first, lets install metrics server
```
kubectl apply -f https://raw.githubusercontent.com/linuxacademy/content-cka-resources/master/metrics-server-components.yaml
```
Check if metrics are comming in. Note: It takes a bit of time to collect metrics.
```
kubectl top pods
```
We have our application deployed lets add some load to it and Scale it out.
```
kubectl apply -f loadtest
```

Get a terminal to the traffic-generator
```
kubectl exec -it traffic-generator -- sh
```

Install wrk
```
apk add --no-cache wrk
```

Simulate some load, **don't forget the port you are using to expose the deployment**
```
wrk -c 5 -t 5 -d 99999 -H "Connection: Close" http://<your_service_url>/ping
```