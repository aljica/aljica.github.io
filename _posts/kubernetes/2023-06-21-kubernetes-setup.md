---
layout: post
title: Installing Kubernetes and Deploying a Flask App with NGINX
categories: kubernetes
---

<!--
![Image](/docs/assets/images/sysadmin-handbook/ch2/1.png)

![Image](/docs/assets/images/sysadmin-handbook/ch2/2.png)

![Image](/docs/assets/images/sysadmin-handbook/ch2/3.png)

![Image](/docs/assets/images/sysadmin-handbook/ch2/4.png)

![Image](/docs/assets/images/sysadmin-handbook/ch2/5.png)

<img src="/docs/assets/images/sysadmin-handbook/ch2/5.png" width="350" height="600">
-->

# Kubernetes Installation & Configuration

There are multiple articles that outline how to install Kubernetes, such as the following:

```
https://levelup.gitconnected.com/a-step-by-step-guide-to-setup-kubernetes-cluster-in-virtualbox-ubuntu-20-04-image-91d4510bbf1c
https://www.keitaro.com/2021/09/03/setting-up-a-kubernetes-on-premise-cluster-with-kubeadm/
https://medium.com/nerd-for-tech/using-cri-o-as-container-runtime-for-kubernetes-b8ddf8326d38
https://adamtheautomator.com/cri-o/
```

In particular, we used the crio container runtime engine to power our Kubernetes cluster.

`kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///var/run/crio.sock`

Make sure you run `sudo swapoff -a` on the control plane, as swap must be turned off for it to work properly.

We created a very simple flask application, uploaded it to Docker Hub, used docker login on the control plane, pulled the image from Docker Hub with `docker pull`, then defined a .yaml file for deploying the flask application, followed by a .yaml file for the service to expose the application, and then set up an NGINX Ingress Controller in front of it.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.34.1
          ports:
            - containerPort: 80

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flask-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: flask-service
                port:
                  number: 5000
```

If you get this error:
`Unexpected error searching IngressClass: ingressclasses.networking.k8s.io "nginx" is forbidden: User "system:serviceaccount:default:default" cannot get resource "ingressclasses" in API group "networking.k8s.io" at the cluster scope`

You should deploy the following on the K8s control plane:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: ingress-controller-role
rules:
- apiGroups: ["networking.k8s.io"]
  resources: ["ingressclasses"]
  verbs: ["get", "list"]

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ingress-controller-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: ingress-controller-role
subjects:
- kind: ServiceAccount
  name: default # Replace with the actual service account used by the Ingress Controller
  namespace: default
```

Additionally, you may need to use Helm if you get the following error when deploying NGINX:

```
"store.go:429 "Ignoring ingress because of error while validating ingress class" ingress="default/flask-ingress" error="ingress does not contain a valid IngressClass"

"store.go:429 "Ignoring ingress because of error while validating ingress class" ingress="default/flask-ingress" error="no object matching key \"nginx\" in local store"
```

`helm install <release_name> ingress-nginx/ingress-nginx --set controller.ingressClass.name=<desired_ingress_class_name>`

You may also need to delete validatingwebhookconfiguration as follows:

```
kubectl get validatingwebhookconfiguration
kubectl delete validatingwebhookconfiguration <name-of-resource-for-nginx-ingress-admission>
https://stackoverflow.com/questions/68576225/failed-calling-webhook-validate-nginx-ingress-kubernetes-io-error-while-apply
```

If you're running your cluster using VirtualBox, do the following to enable SSH:

```
1. VirtualBox -> File -> Tools -> Network Manager -> Host-only Networks tab
2. Set "Configure Adapter Automatically", and make sure DHCP Server is enabled.
3. Power off the VM.
4. VM -> Settings -> Network -> Adapter 2 tab -> Set it as Host-only Adapter
5. Boot up your VM again (once booted, make sure to disable swap again unless you've edited the configuration files that control it)
6. Run `sudo dhclient -1 enp0s8` (or whichever network interface is attached to the host-only adapter)
7. Now do `ip addr show`
8. You will see an IP assigned to the network interface (in this case enp0s8).
9. Connect to the control plane node from your local machine with `ssh username@ip`, using the IP you retrieved in the previous step.
10. You may also consider using the Remote_SSH VSCode Extension, if using VSCode.
```