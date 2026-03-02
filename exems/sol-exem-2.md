
### Mock Exam-2
---
Create a deployment called my-webapp with image: nginx, label tier:frontend and 2 replicas. Expose the deployment as a NodePort service with name front-end-service , port: 80 and NodePort: 30083

---
- Solution

  ```
  controlplane ~ ✖ k create deploy my-webapp --image=nginx --replicas=2 --dry-run=client -o yaml > dep.yaml

  controlplane ~ ➜  vi dep.yaml

  controlplane ~ ➜  k apply -f dep.yaml

  controlplane ~ ✖ k expose deploy my-webapp --name=front-end-service --type=NodePort --port=80 --target-port=30083 --dry-run=client -o yaml > svc.yaml

  controlplane ~ ➜  vi svc.yaml

  controlplane ~ ➜  k apply -f svc.yaml
  service/front-end-service created

  ```
---
Add a taint to the node node01 of the cluster. Use the specification below:

key: app_type, value: alpha and effect: NoSchedule

Create a pod called alpha, image: redis with toleration to node01.

---
- Solution

  ```
  controlplane ~ ✖ k get node node01 -o yaml > node.yaml

  controlplane ~ ➜  vi node.yaml

  controlplane ~ ➜  k taint nodes node01 app_type=alpha:NoSchedule
  node/node01 tainted

  controlplane ~ ➜  k get node node01 -o yaml > node.yaml

  controlplane ~ ➜  k run alpha --image=redis --dry-run=client -o yaml > pod2.yaml

  controlplane ~ ➜  vi pod2.yaml

  controlplane ~ ➜  k apply -f  pod2.yaml

  controlplane ~ ➜  cal pod2.yaml
  -bash: cal: command not found

  controlplane ~ ✖ cat pod2.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      run: alpha
    name: alpha
  spec:
    tolerations:
    - key: "app_type"
      operator: "Equal"
      value: "alpha"
      effect: "NoSchedule"
    containers:
    - image: redis
      name: alpha
      resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
  status: {}
  
  controlplane ~ ➜


  
  ```
---
Apply a label app_type=beta to node controlplane. Create a new deployment called beta-apps with image: nginx and replicas: 3. Set Node Affinity to the deployment to place the PODs on controlplane only.

NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution

---
- Solution

  ```
  controlplane ~ ➜  k label nodes controlplane app_type=beta
  node/controlplane labeled
  
  controlplane ~ ✖ k create deploy beta-apps --image=nginx --replicas=3 --dry-run=client -o yaml > svc2.yaml
  
  controlplane ~ ✖ vi svc2.yaml
  
  controlplane ~ ➜  cat svc2.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      app: beta-apps
    name: beta-apps
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: beta-apps
    strategy: {}
    template:
      metadata:
        labels:
          app: beta-apps
      spec:
        containers:
        - image: nginx
          name: nginx
          resources: {}
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution: 
              nodeSelectorTerms:
                - matchExpressions:
                  - key: app_type
                    operator: In
                    values:
                    - beta
  status: {}
  
  
  controlplane ~ ➜  k apply -f svc2.yaml
  deployment.apps/beta-apps created
  ```
---

Create a new Ingress Resource for the service my-video-service to be made available at the URL: http://ckad-mock-exam-solution.com:30093/video.

To create an ingress resource, the following details are: -

annotation: nginx.ingress.kubernetes.io/rewrite-target: /

host: ckad-mock-exam-solution.com

path: /video

Once set up, the curl test of the URL from the nodes should be successful: HTTP 200

---
- Solution

  Create an ingress resource manifest file using the imperative command:-
  ```
  kubectl create ingress ingress --rule="ckad-mock-exam-solution.com/video*=my-video-service:8080" --dry-run=client -oyaml > ingress.yaml
  ```
  And then add the rewrite-target annotation.
  
  The final manifest file will look like this.
  ```
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
    name: ingress
  spec:
    rules:
    - host: ckad-mock-exam-solution.com
      http:
        paths:
        - backend:
            service:
              name: my-video-service
              port:
                number: 8080
          path: /video
          pathType: Prefix
  ```
  Now, run 
  ```
  kubectl create -f ingress.yaml
  ```
  to create an ingress resource.

---
We have deployed a new pod called pod-with-rprobe. This Pod has an initial delay before it is Ready. Update the newly created pod pod-with-rprobe with a readinessProbe using the given spec

httpGet path: /ready

httpGet port: 8080

---
- Solution

  ```
  controlplane ~ ➜  k get pod pod-with-rprobe -o yaml > pod3.yaml

  controlplane ~ ➜  vi pod3.yaml
  
  controlplane ~ ➜  kubectl replace -f pod
  pod2.yaml  pod3.yaml  
  
  controlplane ~ ➜  kubectl replace -f pod3.yaml --force
  pod "pod-with-rprobe" deleted from default namespace
  pod/pod-with-rprobe replaced
  
  controlplane ~ ➜  cat pod3.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      cni.projectcalico.org/containerID: a8640ebe69768a0e674d2b216e9678913bc6e5f2ff1b5e5ad0e1c09c6cd6d65b
      cni.projectcalico.org/podIP: 172.17.0.10/32
      cni.projectcalico.org/podIPs: 172.17.0.10/32
    creationTimestamp: "2026-03-02T19:40:47Z"
    generation: 1
    labels:
      name: pod-with-rprobe
    name: pod-with-rprobe
    namespace: default
    resourceVersion: "7639"
    uid: bf4cfa72-bf2b-451d-a991-1867bf3418a0
  spec:
    containers:
    - env:
      - name: APP_START_DELAY
        value: "180"
      image: kodekloud/webapp-delayed-start
      imagePullPolicy: Always
      name: pod-with-rprobe
      ports:
      - containerPort: 8080
        protocol: TCP
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-rjlzc
        readOnly: true
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
            
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: controlplane
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
      operator: Exists
      tolerationSeconds: 300
    - effect: NoExecute
      key: node.kubernetes.io/unreachable
      operator: Exists
      tolerationSeconds: 300
    volumes:
    - name: kube-api-access-rjlzc
      projected:
        defaultMode: 420
        sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
            - key: ca.crt
              path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
            items:
            - fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
              path: namespace
  status:{}
  
  controlplane ~ ➜  
  ```
---

Create a new pod called nginx1401 in the default namespace with the image nginx. Add a livenessProbe to the container to restart it if the command ls /var/www/html/probe fails. This check should start after a delay of 10 seconds and run every 60 seconds.

You may delete and recreate the object. Ignore the warnings from the probe.

---
- Solution
  
  Solution manifest file to create a pod called nginx1401 as follows:-
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx1401
    namespace: default
  spec:
    containers:
      - name: nginx1401
        image: nginx
        livenessProbe:
          exec:
            command: ["ls /var/www/html/probe"]
          initialDelaySeconds: 10
          periodSeconds: 60
  ```
---
Create a job called whalesay with image busybox and command echo "cowsay I am going to ace CKAD!".

 - completions: 10
 - backoffLimit: 6
 - restartPolicy: Never

This simple job runs the popular cowsay game that was modifed by docker…

---
- Solution
  
  Solution manifest file to create a job called whalesay as follows:-
  ```
  apiVersion: batch/v1
  kind: Job
  metadata:
    name: whalesay
  spec:
    completions: 10
    backoffLimit: 6
    template:
      metadata:
        creationTimestamp: null
      spec:
        containers:
        - name: busybox-cowsay
          image: busybox
          command:
          - sh
          - -c
          - "echo 'cowsay I am going to ace CKAD!'"
        restartPolicy: Never
  ```
---
Create a pod called multi-pod with two containers.

- Container 1:
  - name: jupiter, image: nginx

- Container 2:
  - name: europa, image: busybox
  - command: sleep 4800

- Environment Variables:
  - Container 1:
  - type: planet
  - Container 2:
  - type: moon

---
- Solution
  
  Solution manifest file to create a multi pod containers called multi-pod as follows:-
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: null
    labels:
      run: multi-pod
    name: multi-pod
  spec:
    containers:
    - image: nginx
      name: jupiter
      env:
      - name: type
        value: planet
    - image: busybox
      name: europa
      command: ["/bin/sh","-c","sleep 4800"]
      env:
       - name: type
         value: moon
  ```
---

Create a PersistentVolume called custom-volume with size: 50MiB reclaim policy:retain, Access Modes: ReadWriteMany and hostPath: /opt/data

---
- Solution
  
  Solution manifest file to create a Persistent Volume called custom-volume as follows:-
  
  ---
  ```
  apiVersion: v1
  kind: PersistentVolume
  apiVersion: v1
  metadata:
    name: custom-volume
  spec:
    accessModes: ["ReadWriteMany"]
    capacity:
      storage: 50Mi
    persistentVolumeReclaimPolicy: Retain
    hostPath:
      path: /opt/data
  ```

---
