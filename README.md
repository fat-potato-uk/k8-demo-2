# k8-demo-2

For this tutorial we are going to be looking primarily around Helm and how it relates to installing software into your
Kubernets cluster.

#### Helm

So what is Helm? Helm is the first application package manager running atop Kubernetes. 
It allows describing the application structure through convenient helm-charts and managing it 
with simple commands.

For the CentOS users amongst you, Helm is to Kubernetes as Yum is to CentOS. Ish.

It actually also provides us with a lot of flexibility and templating around our installations of software
as it provides its own templating language and supporting mechanics that allow us to make significant changes
to our installation without having to make major alterations to the `deployment` manifests.

#### Installing Helm

!!!!!

https://helm.sh/docs/intro/install/

```bash
brew install helm
```

```bash
tar -zxvf helm-v3.6.2-darwin-amd64.tar.gz
cd darwin-amd64/
./helm
```

#### Helm chart the first deployment

So, in our first tutorial we deployed the following to our cluster via a Kubernetes manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: single-pod
  labels:
    role: myrole
spec:
  containers:
    - name: server
      image: fpjack/sample-app
      ports:
        - name: web
          containerPort: 8080
          protocol: TCP
```

```bash
helm create mychart
```

Helm will create a new directory in your project called mychart with the structure shown below.
Let's navigate our new chart to find out how it works.

```yaml
mychart/
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

So, there is a lot of boiler plate that we don't care about installed here. For now, we are going to ignore most of this
and focus on just the `deployment.yaml` and `values.yaml` file.

First, update the `values.yaml` file so it now reads:

```yaml
...
image:
  repository: fpjack/sample-app
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "latest"
...
```

This is the image we used in the first tutorial. Now, if we run:

```bash
helm myfirsthelmchart mychart
```

We should get some output:

```bash
NAME: myfirsthelmchart
LAST DEPLOYED: Wed Jul 14 15:31:47 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=mychart,app.kubernetes.io/instance=myfirsthelmchart" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

To check if the chart is installed, run `helm ls`:

```bash
Andrews-MBP:k8s-demo-2 andrew$ helm ls
NAME            	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART        	APP VERSION
myfirsthelmchart	default  	1       	2021-07-14 15:31:47.400373 +0100 BST	deployed	mychart-0.1.0	1.16.0     
```

If we navigate to the dashboard, we can see that the application is installed and running:

![deployment](deployment.png "deployment")

Or not.

Well, lets see what's actually going on then! Clicking on the error icon we see:

![error](error.png "error")

As we mentioned above, there is A LOT of boilerplate in here. Including the `liveness` and `readiness` probes. These
are ways in which we can tell Kubernetes our services are "ready" to be used:

```yaml
livenessProbe:
httpGet:
  path: /
  port: http
readinessProbe:
httpGet:
  path: /
  port: http
```

The thing is, they require us to implement some behaviours in our application to support this. Which ours does not. 

So, lets just remove those offending lines from our deployment and re-install our chart:

```bash
helm delete myfirsthelmchart ; helm install myfirsthelmchart mychart
```

Now we get:

![deployment2](deployment2.png "deployment2")

Yay, it worked (if things have not gone so well for you, the version checked into this repo includes all of the 
above modifications).

You can also see the services has been included as part of the installation (similar to the last tutorial):

```bash
$ kubectl get services -n default
NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes                 ClusterIP   10.96.0.1       <none>        443/TCP   4h58m
myfirsthelmchart-mychart   ClusterIP   10.102.231.92   <none>        80/TCP    3m3s
```



#### Prom


