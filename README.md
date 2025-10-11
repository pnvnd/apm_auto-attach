# Minikube Setup
Before you start, install Docker and download [Minikube](https://github.com/kubernetes/minikube/releases). 
Once you have Minikube somewhere in your PATH run these commands:
```shell
minikube start -p my-cluster
```

Go to New Relic and go through the Kubernetes Instrumentation (guided install) to get the script/helm chart to install `nri-bundle` and the `k8s-agents-operator`.

# Instrumentation Files
To modify your cluster with these `yaml` files, do this:

```shell
kubectl apply -f ./instrumentation.yaml -n newrelic
```

## Java
Simplified `instrumentation-java.yaml` file with comments removed for clarity. Note here the only real change from the default is the addition of `newrelic/` on line 9 before the image. This was added to avoid issues downloading the image. Assume this is for a `java-spring` app and rename the labels if needed.

```yaml
apiVersion: newrelic.com/v1alpha2
kind: Instrumentation
metadata:
  name: newrelic-instrumentation-java
  namespace: newrelic
spec:
  agent:
    language: java
    image: newrelic/newrelic-java-init:latest
    env:
  podLabelSelector:
    matchExpressions:
      - key: "app"
        operator: "In"
        values: ["java-spring"]
```

Apply changes using this:
```shell
kubectl apply -f ./instrumentation-java.yaml -n newrelic
```


# Deploy Multiple Applications
Check to make sure your pods are showing up:


```
kubectl get pods -A
```
Open up a new tab and create a new Kubernetes dashboard in a separate terminal.


```
minikube dashboard -p my-cluster
```
Use this dashboard to help you navigate your cluster. For example, check the Custom Resource Definition section to see your instrumentation applied above. Otherwise use:


```
kubectl get instrumentation -n newrelic
```
Be sure to create a tunnel to actually get to these apps in a browser:


```
minikube tunnel -p my-cluster
```
Java Deployment
This is for a sample java application with a container image from my GitHub repository: https://github.com/pnvnd/java-spring/pkgs/container/java-spring


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-spring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-spring
  template:
    metadata:
      labels:
        app: java-spring
    spec:
      containers:
        - name: java-spring
          image: ghcr.io/pnvnd/java-spring:latest
          ports:
            - containerPort: 8080
          imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: java-spring
spec:
  selector:
    app: java-spring
  ports:
    - port: 8080
      targetPort: 8080
  type: LoadBalancer
```
Deploy using:

```shell
kubectl apply -f ./app_java-spring.yaml
```
# Results
Notice in each applications pod, there is a `/newrelic-instrumentation` directory and each applications has different files depending what agent was injected into the logs.

Also, each APM entity in New Relic has a `operator: auto-injection` tag.

Remarks
Update your `k8s-agents-operator` using this command (change versions as needed):


```
helm upgrade --install k8s-agents-operator k8s-agents-operator/k8s-agents-operator --namespace newrelic
```
 
