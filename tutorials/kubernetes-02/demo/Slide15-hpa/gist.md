## Relevant Documentation
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
## Lesson Reference
Let's autoscale our application.
For this one we'll need the metric server already installed in our minikube.

Install Kubernetes Metrics Server.
```
kubectl apply -f https://raw.githubusercontent.com/linuxacademy/content-cka-resources/master/metrics-server-components.yaml
```
Using the `deployment.yaml` in this folder run
```
kubectl apply -f deployment.yaml
```
Check if metrics are comming in. Note: It takes a bit of time to collect metrics.
```
kubectl top pods
```

We have our application deployed lets add some load to it and Scale it out.
```
kubectl apply -f traffic-generator.yaml
```

Get a terminal to the traffic-generator
```
kubectl exec -it traffic-generator -- sh
```

Install wrk
```
apk add --no-cache wrk
```

Simulate some load
```
wrk -c 5 -t 5 -d 99999 -H "Connection: Close" http://application-cpu
```

You can scale to pods manually and see roughly 6-7 pods will satisfy resource requests.
```
kubectl scale deploy/application-cpu --replicas 7
```

Now lets spice things up, lets add an Autoscaler.

- Scale the deployment back down to 2.
```
kubectl scale deploy/application-cpu --replicas 2
```
- Deploy the autoscaler
```
kubectl autoscale deploy/application-cpu --cpu-percent=95 --min=1 --max=10
```
Pods should scale to roughly 6-7 to match criteria of 95% of resource requests.
Lets check the results.
```
kubectl get pods
kubectl top pods
kubectl get hpa/application-cpu  -owide
```
Check the hpa object.
```
kubectl describe hpa/application-cpu 
```
