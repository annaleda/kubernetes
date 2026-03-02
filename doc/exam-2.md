
### Mock Exam-2
---
Create a deployment called my-webapp with image: nginx, label tier:frontend and 2 replicas. Expose the deployment as a NodePort service with name front-end-service , port: 80 and NodePort: 30083

---

Add a taint to the node node01 of the cluster. Use the specification below:

key: app_type, value: alpha and effect: NoSchedule

Create a pod called alpha, image: redis with toleration to node01.

---

Apply a label app_type=beta to node controlplane. Create a new deployment called beta-apps with image: nginx and replicas: 3. Set Node Affinity to the deployment to place the PODs on controlplane only.

NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution

---


Create a new Ingress Resource for the service my-video-service to be made available at the URL: http://ckad-mock-exam-solution.com:30093/video.

To create an ingress resource, the following details are: -

annotation: nginx.ingress.kubernetes.io/rewrite-target: /

host: ckad-mock-exam-solution.com

path: /video

Once set up, the curl test of the URL from the nodes should be successful: HTTP 200

---

We have deployed a new pod called pod-with-rprobe. This Pod has an initial delay before it is Ready. Update the newly created pod pod-with-rprobe with a readinessProbe using the given spec

httpGet path: /ready

httpGet port: 8080


---

Create a new pod called nginx1401 in the default namespace with the image nginx. Add a livenessProbe to the container to restart it if the command ls /var/www/html/probe fails. This check should start after a delay of 10 seconds and run every 60 seconds.

You may delete and recreate the object. Ignore the warnings from the probe.

---
Create a job called whalesay with image busybox and command echo "cowsay I am going to ace CKAD!".

 - completions: 10
 - backoffLimit: 6
 - restartPolicy: Never

This simple job runs the popular cowsay game that was modifed by docker…

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


Create a PersistentVolume called custom-volume with size: 50MiB reclaim policy:retain, Access Modes: ReadWriteMany and hostPath: /opt/data

---
