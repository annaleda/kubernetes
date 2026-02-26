### Pod Security

Update the pod ubuntu-sleeper to run as Root user and with the SYS_TIME capability.

Note:
Set the capabilities within the container, not at the pod level, as setting them at the pod level is not supported.
  - Pod Name: ubuntu-sleeper
  - Image Name: ubuntu
  - SecurityContext: Capability SYS_TIME
---

Is run as a root user?

kubectl exec multi-pod -- id

---
Now update the pod to also make use of the NET_ADMIN capability.

Note:
Ensure that you set the capabilities at the container level rather than at the pod level, as configuring them at the pod level is not supported.
  - Pod Name: ubuntu-sleeper
  - Image Name: ubuntu
  - SecurityContext: Capability SYS_TIME
  - SecurityContext: Capability NET_ADMIN

------
