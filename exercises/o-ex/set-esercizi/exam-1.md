# CKAD Exercises

## Q. 1

Task  
SECTION: APPLICATION DESIGN AND BUILD

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

In the ckad-multi-containers namespace, create a pod named tres-containers-pod, which has 3 containers matching the below requirements:

The first container named primero runs busybox:1.28 image and has ORDER=FIRST environment variable.

The second container named segundo runs nginx:1.17 image and is exposed at port 8080.

The last container named tercero runs busybox:1.31.1 image and has ORDER=THIRD environment variable.

NOTE: All pod containers should be in the running state.

---

<details>
<summary>Soluzione</summary>

Use below YAML to create desired pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: primero
  name: tres-containers-pod
  namespace: ckad-multi-containers
spec:
  containers:
  - env:
    - name: ORDER
      value: FIRST
    image: busybox:1.28
    name: primero
    command:
    - /bin/sh
    - -c
    - sleep 3600;
    resources: {}
  - image: nginx:1.17
    name: segundo
    ports:
    - containerPort: 8080
    resources: {}
  - env:
    - name: ORDER
      value: THIRD
    image: busybox:1.31.1
    name: tercero
    command:
    - /bin/sh
    - -c
    - sleep 3600;
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</details>

---

Details

Is the primero container running?

Is the primero container use busybox:1.28 image?

Is the primero container environment variable well configured?

Does the segundo container running?

Does the segundo container use nginx:1.17 image?

Is the segundo container expose at port 8080?

Is the tercero container running?

Is the tercero container use busybox:1.31.1 image?

Is the tercero container environment variable well configured?

## Q. 2

Task  
SECTION: APPLICATION DESIGN AND BUILD

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Create a storage class with the name banana-sc-ckad08-str as per the properties given below:

- Provisioner should be kubernetes.io/no-provisioner,
- Volume binding mode should be WaitForFirstConsumer.
- Volume expansion should be enabled.

---

<details>
<summary>Soluzione</summary>

Use below YAML to create desired storage-class:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: banana-sc-ckad08-str
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

</details>

---

Details

Is banana-sc-ckad08-str storage class created?

## Q. 3

Task  
SECTION: APPLICATION DESIGN AND BUILD

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

In the ckad-job namespace, create a cronjob named simple-node-job to run every 30 minutes to list all the running processes inside a container that used node image (the command needs to be run in a shell).

In Unix-based operating systems, ps -eaf can be use to list all the running processes.

---

<details>
<summary>Soluzione</summary>

Create a YAML file with the content as below:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-node-job
  namespace: ckad-job
spec:
  schedule: "*/30 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: simple-node-job
            image: node
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - ps -eaf
          restartPolicy: OnFailure
```

Then use `kubectl apply -f file_name.yaml` to create the required object.

</details>

---

Details

Is cronjob simple-node-job created?

Is the container image node?

Does cronjob run ps -eaf command?

Does cronjob run every 30 minutes?

## Q. 4

Task  
SECTION: APPLICATION DESIGN AND BUILD

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

In the ckad-pod-design namespace, create a pod called ckad-ubuntu-qwfefefwe that runs a ubuntu image.

The pod's container should be named ubuntu-server; the container will sleep for 3600 seconds.

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ckad-ubuntu-qwfefefwe
  namespace: ckad-pod-design
spec:
  containers:
  - name: ubuntu-server
    image: ubuntu
    command:
    - sleep
    - "3600"
```

</details>

---

## Q. 5

Task  
SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

In this task, we have to create two identical environments that are running different versions of the application. The team decided to use the Blue/green deployment method to deploy a total of 10 application pods which can mitigate common risks such as downtime and rollback capability.

Also, we have to route traffic in such a way that 30% of the traffic is sent to the green-apd environment and the rest is sent to the blue-apd environment. All the development processes will happen on cluster 3 because it has enough resources for scalability and utility consumption.

Specification details for creating a blue-apd deployment are listed below: -

The name of the deployment is blue-apd.  
Use the label type-one: blue.  
Use the image kodekloud/webapp-color:v1.  
Add labels to the pod type-one: blue and version: v1.

Specification details for creating a green-apd deployment are listed below: -

The name of the deployment is green-apd.  
Use the label type-two: green.  
Use the image kodekloud/webapp-color:v2.  
Add labels to the pod type-two: green and version: v1.

We have to create a service called route-apd-svc for these deployments. Details are here: -

The name of the service is route-apd-svc.  
Use the correct service type to access the application from outside the cluster and application should listen on port 8080.  
Use the selector label version: v1.

NOTE: - We do not need to increase replicas for the deployments, and all the resources should be created in the default namespace.

You can check the status of the application from the terminal by running the curl command with the following syntax:

```bash
curl http://cluster3-controlplane:NODE-PORT
```

You can SSH into the cluster3 using `ssh cluster3-controlplane` command.

---

<details>
<summary>Soluzione</summary>

Run the following command to change the context: -

```bash
kubectl config use-context cluster3
```

In this task, we will use the kubectl command. Here are the steps: -

Use the kubectl create command to create a deployment manifest file as follows: -

```bash
kubectl create deployment blue-apd --image=kodekloud/webapp-color:v1 --dry-run=client -o yaml > <FILE-NAME-1>.yaml
kubectl create deployment green-apd --image=kodekloud/webapp-color:v2 --dry-run=client -o yaml > <FILE-NAME-2>.yaml
kubectl create service nodeport route-apd-svc --tcp=8080:8080 --dry-run=client -oyaml > <FILE-NAME-3>.yaml
```

Open the file with any text editor such as vi or nano and make the changes as per given in the specifications. It should look like this: -

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    type-one: blue
  name: blue-apd
spec:
  replicas: 7
  selector:
    matchLabels:
      type-one: blue
      version: v1
  template:
    metadata:
      labels:
        version: v1
        type-one: blue
    spec:
      containers:
        - image: kodekloud/webapp-color:v1
          name: blue-apd
```

We will deploy a total of 10 application pods. Also, we have to route 70% traffic to blue-apd and 30% traffic to the green-apd deployment according to the task description.

Since the service distributes traffic to all pods equally, we have to set the replica count 7 to the blue-apd deployment so that the given service will send ~70% traffic to the deployment pods.

green-apd deployment should look like this: -

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    type-two: green
  name: green-apd
spec:
  replicas: 3
  selector:
    matchLabels:
      type-two: green
      version: v1
  template:
    metadata:
      labels:
        type-two: green
        version: v1
    spec:
      containers:
        - image: kodekloud/webapp-color:v2
          name: green-apd
```

route-apd-svc service should look like this: -

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: route-apd-svc
  name: route-apd-svc
spec:
  type: NodePort
  ports:
    - port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    version: v1
```

Now, create a deployment and service by using the kubectl create -f command: -

```bash
kubectl create -f <FILE-NAME-1>.yaml -f <FILE-NAME-2>.yaml -f <FILE-NAME-3>.yaml
```

</details>

---

Details

Is blue deployment configured correctly?

Is green deployment configured correctly?

Is service configured correctly?

## Q. 6

Task  
SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

On cluster3, in the dev-001 namespace, one of the interns deployed one web application called news-apd.

After successfully deploying on the worker node, we start getting alerts about the pod crashing.

We want you to inspect the dev-001 namespace and fix those issues.

---

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl get all -n dev-001
kubectl describe pod -n dev-001
kubectl logs -n dev-001 <POD-NAME>
kubectl edit deployment news-apd -n dev-001
kubectl rollout status deployment/news-apd -n dev-001
```

Inspect the deployment, identify the wrong image / command / port / env issue causing the crash, fix it, and wait for the rollout to complete.

</details>

---

## Q. 7

Task  
SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

On the student-node, a Helm chart repository is given under the /opt/ path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. The files have some issues. Fix those issues and deploy them with the following specifications: -

The release name should be webapp-color-apd.  
All the resources should be deployed on the frontend-apd namespace.  
The service type should be node port.  
Scale the deployment to 3.  
Application version should be 1.20.0.

NOTE: - Remember to make necessary changes in the values.yaml and Chart.yaml files according to the specifications, and, to fix the issues, inspect the template files.

---

<details>
<summary>Soluzione</summary>

Run the following command to change the context: -

```bash
kubectl config use-context cluster2
```

In this task, we will use the helm commands. Here are the steps: -

First, check the given namespace; if it doesn't exist, we must create it first; otherwise, it will give an error "namespaces not found" while installing the helm chart.

To check all the namespaces in the cluster2, we would have to run the following command: -

```bash
kubectl get ns
```

If the given namespace doesn't exist, then run the following command: -

```bash
kubectl create ns frontend-apd
```

Now, on the student-node go to the /opt/ directory. We have given the helm chart directory webapp-color-apd that contains templates, values files, and the chart file etc.

Update the values according to the given specifications as follows: -

a.) Update the value of the appVersion to 1.20.0 in the Chart.yaml file.  
b.) Update the value of the replicaCount to 3 in the values.yaml file.  
c.) Update the value of the type to NodePort in the values.yaml file.

Now, we will use the helm lint command to check the Helm chart because it can identify errors such as missing or misconfigured values, invalid YAML syntax, and deprecated APIs etc.

```bash
cd /opt/
helm lint ./webapp-color-apd/
```

If there is no misconfiguration, we will see similar output.

But in our case, there are some issues with the given templates.

Deployment apiVersion needs to be correctly written. It should be `apiVersion: apps/v1`.

In the service YAML, there is a typo in the template variable `{{ .Values.service.name }}` because of that, it's not able to reference the value of the name field defined in the values.yaml file for the Kubernetes service that is being created or updated.

Now run the following command to install the helm chart in the frontend-apd namespace: -

```bash
helm install webapp-color-apd -n frontend-apd ./webapp-color-apd
helm ls -n frontend-apd
```

</details>

---

Details

Is the release name set?

Are the resources deployed on given namespace?

Is the service type defined?

Is the deployment scaled?

Is the application version set?

Are resources running?

## Q. 8

Task  
SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

We have deployed a pod pod22-ckad-svcn in the default namespace. Create a service svc22-ckad-svcn that will expose the pod at port 6335.

Note: Use the imperative command for the above scenario.

---

<details>
<summary>Soluzione</summary>

```bash
kubectl expose pod pod22-ckad-svcn --name=svc22-ckad-svcn --port=6335
```

</details>

---

## Q. 9

Task  
SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

You are requested to create a network policy named deny-all-svcn that denies all incoming and outgoing traffic to ckad12-svcn namespace.

Note: The namespace ckad12-svcn doesn't exist. Create the namespace before creating the Policy.

---

<details>
<summary>Soluzione</summary>

```bash
kubectl create namespace ckad12-svcn
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-svcn
  namespace: ckad12-svcn
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF
```

</details>

---

## Q. 10

Task  
SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

We have an external webserver running on student-node which is exposed at port 9999.

We have also created a service called external-webserver-ckad01-svcn that can connect to our local webserver from within the cluster3 but, at the moment, it is not working as expected.

Fix the issue so that other pods within cluster3 can use external-webserver-ckad01-svcn service to access the webserver.

---

<details>
<summary>Soluzione</summary>

Let's check if the webserver is working or not:

```bash
curl student-node:9999
```

Now we will check if service is correctly defined:

```bash
kubectl describe svc external-webserver-ckad01-svcn
```

As we can see there is no endpoints specified for the service, hence we won't be able to get any output. Since we can not destroy any k8s object, let's create the endpoint manually for this service as shown below:

```bash
export IP_ADDR=$(ifconfig eth0 | grep 'inet ' | awk '{print $2}')

kubectl apply -f - <<EOF
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-webserver-ckad01-svcn
  labels:
    kubernetes.io/service-name: external-webserver-ckad01-svcn
addressType: IPv4
ports:
  - protocol: TCP
    port: 9999
endpoints:
  - addresses:
      - $IP_ADDR
EOF
```

Finally check if the curl test works now:

```bash
kubectl --context cluster3 run --rm -i test-curl-pod --image=curlimages/curl --restart=Never -- curl -m 2 external-webserver-ckad01-svcn
```

</details>

---

Details

Does the external-webserver-ckad01-svcn service have the correct IP addresses listed as endpoints?

Is correct port specified ?

Can other pods use external-webserver-ckad01-svcn service to access webserver?

## Q. 11

Task  
SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Deploy a pod with name webapp-svcn using the kodekloud/webapp-color image with the label tier=msg.

Now, Create a service webapp-service-svcn to expose the pod webapp-svcn application within the cluster on port 6379.

---

<details>
<summary>Soluzione</summary>

```bash
kubectl run webapp-svcn --image=kodekloud/webapp-color --labels=tier=msg
kubectl expose pod webapp-svcn --name=webapp-service-svcn --port=6379
```

</details>

---

## Q. 12

Task  
SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

For this scenario, create a Service called ckad12-service that routes traffic to an external IP address.

Please note that service should listen on port 53 and be of type ExternalName. Use the external IP address 8.8.8.8

Create the service in the default namespace.

---

<details>
<summary>Soluzione</summary>

Create the service using the following manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ckad12-service
spec:
  type: ExternalName
  externalName: 8.8.8.8
  ports:
    - name: http
      port: 53
      targetPort: 53
```

</details>

---

Details

Is service ckad12-service created?

Is service of type - "ExternalName"?

Is service configured with externalName - 8.8.8.8?

## Q. 13

Task  
SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Create a pod named ckad17-qos-aecs-3 in namespace ckad17-nqoss-aecs with image nginx and container name ckad17-qos-ctr-3-aecs.

Define other fields such that the Pod is configured to use the Quality of Service (QoS) class of Burstable.

Also retrieve the name and QoS class of each Pod in the namespace ckad17-nqoss-aecs in the below format and save the output to a file named qos_status_aecs in the /root directory.

Format:

```text
NAME    QOS
pod-1   qos_class
pod-2   qos_class
```

---

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ckad17-qos-aecs-3
  namespace: ckad17-nqoss-aecs
spec:
  containers:
  - name: ckad17-qos-ctr-3-aecs
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
EOF

kubectl --namespace=ckad17-nqoss-aecs get pod --output=custom-columns="NAME:.metadata.name,QOS:.status.qosClass"
kubectl --namespace=ckad17-nqoss-aecs get pod --output=custom-columns="NAME:.metadata.name,QOS:.status.qosClass" > /root/qos_status_aecs
```

</details>

---

Details

Is the pod created with the "QoS" class of 'Burstable'?

Is the QoS class of each pod retrieved in the format mentioned?

## Q. 14

Task  
SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Create a custom resource my-anime of kind Anime with the below specifications:

Name of Anime: Death Note  
Episode Count: 37

TIP: You may find the respective CRD with anime substring in it.

---

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl get crd | grep -i anime
kubectl get crd animes.animes.k8s.io -o json | jq .spec.versions[].schema.openAPIV3Schema.properties.spec.properties
kubectl api-resources | grep anime

cat << YAML | kubectl apply -f -
apiVersion: animes.k8s.io/v1alpha1
kind: Anime
metadata:
  name: my-anime
spec:
  animeName: "Death Note"
  episodeCount: 37
YAML
```

</details>

---

Details

Are correct specifications used for custom resource?

## Q. 15

Task  
SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Create a ConfigMap named ckad04-config-multi-env-files-aecs in the default namespace from the environment(env) files provided at /root/ckad04-multi-cm directory.

---

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
kubectl create configmap ckad04-config-multi-env-files-aecs \
  --from-env-file=/root/ckad04-multi-cm/file1.properties \
  --from-env-file=/root/ckad04-multi-cm/file2.properties
```

</details>

---

Details

Is ConfigMap created with proper configuration ?

## Q. 16

Task  
SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

We have already deployed the required pods and services in the namespace ckad01-db-sec.

Create a new secret named ckad01-db-scrt-aecs with the data given below.

Secret Name: ckad01-db-scrt-aecs

Secret 1: DB_Host=sql01  
Secret 2: DB_User=root  
Secret 3: DB_Password=password123

Configure ckad01-mysql-server to load environment variables from the newly created secret, where the keys from the secret should become the environment variable name in the Pod.

---

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3

kubectl create secret generic ckad01-db-scrt-aecs \
  --namespace=ckad01-db-sec \
  --from-literal=DB_Host=sql01 \
  --from-literal=DB_User=root \
  --from-literal=DB_Password=password123

kubectl get -n ckad01-db-sec pod ckad01-mysql-server -o yaml > webapp-pod-sec-cfg.yaml
```

Edit `webapp-pod-sec-cfg.yaml` so that it includes:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: ckad01-mysql-server
  name: ckad01-mysql-server
  namespace: ckad01-db-sec
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: ckad01-db-scrt-aecs
```

Then recreate the pod:

```bash
kubectl replace -f webapp-pod-sec-cfg.yaml --force
kubectl exec -n ckad01-db-sec ckad01-mysql-server -- printenv | egrep -w 'DB_Password=password123|DB_User=root|DB_Host=sql01'
```

</details>

---

Details

Is the secret created with correct values ?

Is the pod configured to use secret as env vars?

## Q. 17

Task  
SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Create a ResourceQuota called ckad16-rqc in the namespace ckad16-rqc-ns and enforce a limit of one ResourceQuota for the namespace.

---

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl create namespace ckad16-rqc-ns

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ckad16-rqc
  namespace: ckad16-rqc-ns
spec:
  hard:
    resourcequotas: "1"
EOF
```

</details>

---

Details

Is resource quota created?

## Q. 18

Task  
SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Using the pod template on student-node at /root/ckad08-dotfile-aecs.yaml , create a pod ckad18-secret-pod in the namespace ckad18-secret with the specifications as defined below:

Define a volume section named secret-volume that is backed by a Kubernetes Secret named ckad18-secret-aecs.

Mount the secret-volume volume to the container's /etc/secret-volume directory in read-only mode, so that the container can access the secrets stored in the ckad18-secret-aecs secret.

---

<details>
<summary>Soluzione</summary>

```bash
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ckad18-secret-pod
  namespace: ckad18-secret
spec:
  restartPolicy: Never
  volumes:
  - name: secret-volume
    secret:
      secretName: ckad18-secret-aecs
  containers:
  - name: ckad08-top-scrt-ctr-aecs
    image: registry.k8s.io/busybox
    command:
    - ls
    - "-al"
    - "/etc/secret-volume"
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
EOF
```

</details>

---

Details

Is "read-only" volume created using "ckad18-secret-aecs" ?

Is 'readonly' volumeMount used with correct path ?

## Q. 19

Task  
SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Update the newly created pod simple-webapp-aom with a readinessProbe using the given specifications.

Configure an HTTP readiness probe with:

path value set to /ready

port number to access container is 8080

initialDelaySeconds set to 15 (to allow app startup time)

Note: You need to recreate the pod to add the readiness probe configuration.

---

<details>
<summary>Soluzione</summary>

Use the following YAML file and create a file - for example, simple-webapp-aom.yaml:

```bash
cat <<EOF > simple-webapp-aom.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-aom
  labels:
    name: simple-webapp-aom
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-delayed-start
    ports:
    - containerPort: 8080
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 5
EOF
```

To recreate the pod, run the command:

```bash
kubectl replace -f simple-webapp-aom.yaml --force
```

</details>

---

Details

Is pod named simple-webapp-aom created?

Image Name: kodekloud/webapp-delayed-start

Readiness Probe: httpGet

Http Probe: /ready

Http Port: 8080

## Q. 20

Task  
SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Pod manifest file is already given under the /root/ directory called ckad-pod-busybox.yaml.

There is error with manifest file correct the file and create resource.

---

<details>
<summary>Soluzione</summary>

You will see following error

```bash
kubectl create -f ckad-pod-busybox.yaml
Error from server (BadRequest): error when creating "ckad-pod-busybox.yaml": Pod in version "v1" cannot be handled as a Pod.
```

Use the following yaml file and create resource

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ckad-pod-busybox
spec:
  containers:
    - command:
        - sleep
        - "3600"
      image: busybox
      name: pods-simple-container
```

</details>

---

Details

Is the pod ckad-pod-busybox running?

Is correct apiVersion used?

Is image busybox?

## Q. 21

Task  
SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

Create a new pod with image redis and name ckad-probe and configure the pod with livenessProbe with command ls and set initialDelaySeconds to 5 .

TIP: - Make use of the imperative command to create the above pod.

---

<details>
<summary>Soluzione</summary>

Using imperative command

```bash
kubectl run ckad-probe --image=redis  --dry-run=client -o yaml > ckad-probe.yaml
```

Use the following YAML file update yaml with livenessProbe

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: ckad-probe
spec:
  containers:
    - image: redis
      imagePullPolicy: IfNotPresent
      name: redis
      resources: {}
      livenessProbe:
        exec:
          command:
            - ls
        initialDelaySeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

To recreate the pod, run the command:

```bash
kubectl create -f ckad-probe.yaml
```

</details>

---

Details

Podname: ckad-probe

redis

livenessProbe

Command set to ls
