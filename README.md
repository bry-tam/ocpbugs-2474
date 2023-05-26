## Disclaimer 
_Please, kindly reminder that automated workarounds are out of scope of coverage of Red Hat Support Services[1]._
_This script is delivered as courtesy support through a TAM engagement in an strategic Red Hat Partner._
_Customers and partners are welcome to review and enhance this script at its own at the discretion, but must be aware that it is out of support scope._
[1] https://access.redhat.com/support/offerings/production/soc 

## Background

The following notes describes a workaround for [OCPBUGS-2474](https://issues.redhat.com/browse/OCPBUGS-2474). During our investigation, we discovered a kubelet issue, after the pod workers delete pod syncTerminated failed, reported in the kubernetes upstream as well https://github.com/kubernetes/kubernetes/issues/113091 and downstream https://github.com/openshift/kubernetes/pull/1399, wich cause kubelet not being able to start a new pod with the same namespace, affecting static pods of cluster operators static pods like kube-apiserver, etcd and openshift-kube-scheduler-guard.   

A workaround to resolve this will be to restart the kubelet service on the affected Master nodes. 

Manual workarounds may be problematic for customers and partners managing hundreds of clusters in the field. 
This deployment is proposed to be used, in such scenarios, as temporary fix while we wait for definitive kubelet fixes through PRs kubernetes upstream https://github.com/kubernetes/kubernetes/issues/113091 and openshift downstream https://github.com/openshift/kubernetes/pull/1399.

The provided script automates detection and remediation of the aforementioned issues.
The script will create a Deployment on the master nodes with the rights to:

* list cluster operators
* list nodes
* list and delete pods
* exec inside pods

The pod rh-support-ocpbugs-2474, will run the detection/remediation script every 5 minutes.
When the script find `MissingStaticPodControllerDegraded` message reported by by Cluster Operators using static PODs, it will review the pods and define the affected node that must have kubelet restarted. 

The function find_error_msg, in the cleanup.sh script, could be enhanced to detect additional error messages as well.
Customers and partners are welcome to review and enhance this script at its own at the discretion. 

This is currently deployed as single replica, as far as it wasn't still reproduced in current TAM labs. But could be eventually be also deployed with additional replicas to avoid corner cases, with the caveat of additional load on the cluster. 


## Verifying the pod logs

### Logs when there is nothing to do

Check the pod logs:
~~~
$ for p in $(oc get pods -n rh-support-ocpbugs-2474 -l app=rh-support-ocpbugs-2474 -o name); do echo === $p === ; oc -n rh-support-ocpbugs-2474 logs $p --tail=5; done
=== pod/rh-support-ocpbugs-2474-c656669dc-mlc4t ===
++ TZ=Z
++ date +%FT%TZ
+ echo '2022-11-02T12:32:40Z /scripts/cleanup.sh: Cluster healthy'
2022-11-02T12:32:40Z /scripts/cleanup.sh: Cluster healthy
+ exit 0

~~~

### Logs for kubelet restart action

When the script find `MissingStaticPodControllerDegraded` message reported by any Cluster Operator, it will review the pods and define the affected node that must have kubelet restarted: 
~~~
## Not reproduced yet in TAM labs, asked QA team assigned Bug https://issues.redhat.com/browse/OCPBUGS-2474 to evaluate it. 
~~~

