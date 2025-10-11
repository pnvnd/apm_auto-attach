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

To check your instrumentation:
```shell
kubectl get instrumentation -n newrelic
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
## .NET
Similar to Java example above, but for .NET (rename the labels as needed).

```yml
apiVersion: newrelic.com/v1alpha2
kind: Instrumentation
metadata:
  name: newrelic-instrumentation-dotnet
  namespace: newrelic
spec:
  agent:
    language: dotnet
    image: newrelic/newrelic-dotnet-init:latest
    env:
  podLabelSelector:
    matchExpressions:
      - key: "app"
        operator: "In"
        values: ["dotnet-webapi"]
```

Apply changes with:
```shell
kubectl apply -f ./instrumentation-dotnet.yaml -n newrelic
```

## Python
Similar to the Java and .NET examples above but for python. Change labels as needed.

```shell
apiVersion: newrelic.com/v1alpha2
kind: Instrumentation
metadata:
  name: newrelic-instrumentation-python
  namespace: newrelic
spec:
  agent:
    language: python
    image: newrelic/newrelic-python-init:latest
    env:
  podLabelSelector:
    matchExpressions:
      - key: "app"
        operator: "In"
        values: ["python-eamusement"]
```

Apply changes with
```shell
kubectl apply -f ./instrumentation-python.yaml -n newrelic
```

## Ruby
Similar to the Java, .NET, and Python examples above but for Ruby. Change labels as needed.

```shell
apiVersion: newrelic.com/v1alpha2
kind: Instrumentation
metadata:
  name: newrelic-instrumentation-ruby
  namespace: newrelic
spec:
  agent:
    language: ruby
    image: newrelic/newrelic-ruby-init:latest
    env:
  podLabelSelector:
    matchExpressions:
      - key: "app"
        operator: "In"
        values: ["ruby-sinatra"]
```

Apply changes with
```shell
kubectl apply -f ./instrumentation-ruby.yaml -n newrelic
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

## Java Deployment
This is for a sample java application with a container image from my GitHub repository:
https://github.com/pnvnd/java-spring/pkgs/container/java-spring

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

## .NET Deployment
Another sample I created with container image in my GitHub repository:  
https://github.com/pnvnd/dotnet-webapi/pkgs/container/dotnet-webapi

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dotnet-webapi
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dotnet-webapi
  template:
    metadata:
      labels:
        app: dotnet-webapi
    spec:
      containers:
        - name: dotnet-webapi
          image: ghcr.io/pnvnd/dotnet-webapi:latest
          ports:
            - containerPort: 80
          imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: dotnet-webapi
spec:
  selector:
    app: dotnet-webapi
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer
```
Deploy using:

```shell
kubectl apply -f ./app_dotnet-webapi.yaml
```


## Python Deployment
Another application with a container image from my GitHub repository:  
https://github.com/pnvnd/pyeamu/pkgs/container/pyeamu

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-eamusement
spec:
  replicas: 1
  selector:
    matchLabels:
      app: python-eamusement
  template:
    metadata:
      labels:
        app: python-eamusement
    spec:
      containers:
        - name: python-eamusement
          image: ghcr.io/pnvnd/pyeamu:main
          ports:
            - containerPort: 8000
          imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: python-eamusement
spec:
  selector:
    app: python-eamusement
  ports:
    - port: 8000
      targetPort: 8000
  type: LoadBalancer
```
Deploy using:

```shell
kubectl apply -f ./app_python-eamusement.yaml
```

## Ruby Deployment
Another application with a container image from my GitHub repository:  
https://github.com/pnvnd/ruby-sinatra/pkgs/container/ruby-sinatra

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ruby-sinatra
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ruby-sinatra
  template:
    metadata:
      labels:
        app: ruby-sinatra
    spec:
      containers:
        - name: ruby-sinatra
          image: ghcr.io/pnvnd/ruby-sinatra:latest
          ports:
            - containerPort: 8888
          imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: ruby-sinatra
spec:
  selector:
    app: ruby-sinatra
  ports:
    - port: 8888
      targetPort: 8888
  type: LoadBalancer

```
Deploy using:

```shell
kubectl apply -f ./app_ruby-sinatra.yaml
```

# Results
Notice in each applications pod, there is a `/newrelic-instrumentation` directory and each applications has different files depending what agent was injected into the logs.

Also, each APM entity in New Relic has a `operator: auto-injection` tag.

Remarks
Update your `k8s-agents-operator` using this command (change versions as needed):

```
helm upgrade --install k8s-agents-operator k8s-agents-operator/k8s-agents-operator --namespace newrelic
```
 
