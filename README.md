# k8-demo-2

For this tutorial we are going to be looking primarily around Helm and how it relates to installing software into your
Kubernets cluster.

#### Helm

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







#### Prom


