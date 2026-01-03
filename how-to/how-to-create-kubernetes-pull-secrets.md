# K8S: How to Create Pull Secrets


## Prerequisites:

Before you see how to perform the above two steps, ensure you have the following prerequisites:

- Access to a container registry.
- The kubectl command-line tool configured to communicate with your cluster using your cluster’s KUBECONFIG configuration file.

in Kubernetes, a node’s Kubelet and container runtime manage the containerized workloads. The Kubelet ensures containers are running and communicates with the container runtime how those containers should run.

When you create a deployment, the Kubelet reads the PodSpec (a YAML or JSON object that describes a Pod) and then instructs the container runtime using the CRI (Container Runtime Interface) to spin up containers to satisfy that spec. 

The container runtime pulls the image from the specified container registry and runs it. If you don’t specify a container registry hostname, Kubernetes will assume the image is in the Default Docker Registry. 




## Create the Secret


### via kubectl
To create the Secret, in your terminal window, run the following command:
```
$ kubectl create secret docker-registry <secret_name> \
    --docker-server='<your_registry_url>' \
    --docker-username='<your_registry_username>' \
    --docker-password='<your_registry_auth_password>' \
    --docker-email='<your_email_address>'
    -n <namespace>
```

 In the above command:
- `<secret_name>` will be the Secret name of your choice, for example, registry-secret. You will use the name when referring to the Secret in your resource manifest file. 
- `<your_registry_url>` will be the URL to your container Registry. In most cases, it will include the region it is hosted in. For example, if your container registry is Amazon ECR, your registry URL will take the following form: <aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com.
- `<your_registry_username>` will be your username when accessing the container registry. 
- `<your_registry_auth_password>` will be the password to your username.
- `<your_email_address>` will be your email address. 
- `<namespace>` - kubernetes namespace were secret will be created

### via resource file
- To create the kuberntes secret resource `secret.yaml`, in your terminal window, run the following command:
```
$ kubectl create secret docker-registry <secret_name> \
    --docker-server='<your_registry_url>' \
    --docker-username='<your_registry_username>' \
    --docker-password='<your_registry_auth_password>' \
    --docker-email='<your_email_address>' \
    -n <namespace> \
    --dry-run=client \
    -o yaml > secret.yaml
```

- You will get secret.yaml with encoded in base64 content
```
## secret.yaml                                 
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJ5b3VewyX3JlZ2lzdHJ5X3VybCI6eyJ1c2VybmFtZSI6InlvdXJfcmVnaXN0cnlfdXNlcm5hbWUiLCJwYXNzd29yZCwewI6InlvdXJfcmVnaXN0cnlfYXV0aF9wYXNzd29yZCIsImVtYWlsIjoieW91cl9lbWFpbF9hZGRyZXNzIiwiYXV0fdjbDl5WldkcGMzUnllVjkxYzJWeWJtRnRaVHA1YjNWeVgzSmxaMmx6ZEhKNVgyRjFkR2hmY0dGemMzZHZjbVE9In19fQ==
kind: Secret
metadata:
  creationTimestamp: null
  name: regcred
  namespace: namespace
type: kubernetes.io/dockerconfigjson
```

- Create secret in kubernetes
```
$ kubectl apply -f secret.yaml
```

## Update Kubernetes deployment by pull secret
Add pull secrets into .spec.template.spec

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx
    spec:
      imagePullSecrets: # Put there
      - name: regcred
      containers:
      - name: iam
        image: my-registry/nginx:latest
        imagePullPolicy: Always
...
```