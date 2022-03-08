### liveness
```
never create pods directly. Instead, you create other types of resources, such as ReplicationControllers or Deployments, 
which then create and manage the actual pods.

But if the whole node fails, the pods on the node are lost and will not be replaced with new ones, 
unless those pods are managed by the previously mentioned ReplicationControllers or similar

liveness probe periodically with initialDelay, no auth makes suer that the container is up and responding to request 
within timeout limit.
When liveness probe fails then the pod restarted by k8s.
Kubernetes keeps your containers running by restarting them if they crash or if their liveness probes fail. 
This job is performed by the Kubelet on the node hosting the pod.
the Kubernetes Control Plane components running on the master(s) have no part in this process.

But if the node itself crashes, it’s the Control Plane that must create replacements for all the pods that 
went down with the node. It doesn’t do that for pods that you create directly.
Those pods aren’t managed by anything except by the Kubelet, 
but because the Kubelet runs on the node itself, it can’t do anything if the node fails.


```
### repication controller & set
```
pod which are not created by rc won't be restarted once died.
rc uses level selector to detect the pod count distributed across the nodes.
if the level value of pod is modified then the rc recretes new pod. old pod goes orphan

instead rc we can use rs. rs does samething what rc do.
rc does equity based label selctor with pod.
rs do set based like matchLabels or matchExpression or both.

rc & rs create new pods randomly in one of the node.

```
### daemon set pod
```
this will run a pod in every node. daemonset can use node-label-selector to decide on which node it has
to create the pod.
whenever a new node is created daemonset will attach a pod.
if pod is deleted manually daemonset will recreate it in the same node.

daemonset doesn't maintain replica count like rc and rs.

 remember that nodes can be made unschedulable, preventing pods from being deployed to them. 
 A DaemonSet will deploy pods even to such nodes, because the unschedulable attribute is only used by 
 the Scheduler, whereas pods managed by a DaemonSet bypass the Scheduler completely. This is usually desirable, 
 because DaemonSets are meant to run system services, which usually need to run even on unschedulable nodes.
 
```
### k8s job: pods that do one task and die
```
Kubernetes includes support for this through the Job resource, which is similar to the other k8s resources, 
but it allows you to run a pod whose container isn’t restarted when the process running inside 
finishes successfully. Once it does, the pod is considered complete.

In the event of a node failure, the pods on that node that are managed by a Job 
will be rescheduled to other nodes the way ReplicaSet pods are. 
In the event of a failure of the process itself (when the process returns an error exit code),
the Job can be configured to either restart the container or not.

In a pod’s specification, you can specify what Kubernetes should do when the processes running 
in the container finish. 
This is done through the restartPolicy pod spec property, which defaults to Always. 
Job pods can’t use the default policy, because they’re not meant to run indefinitely. 
Therefore, you need to explicitly set the restart policy to either OnFailure or Never. 
This setting is what prevents the container from being restarted when it finishes 
(not the fact that the pod is being managed by a Job resource).

The reason the pod isn’t deleted (but status set as COMPLETETD)
when it completes is to allow you to examine its logs

```
### multipod job
```
Jobs may be configured to create more than one pod instance and run them in parallel or sequentially. 
This is done by setting the completions and the parallelism properties in the Job spec.


apiVersion: batch/v1
kind: Job
metadata:
  name: multi-completion-batch-job
spec:
  completions: 5                                1
  template:
    <template is the same as in listing 4.11>
1 Setting completions to 5 makes this Job run five pods sequentially.
This Job will run five pods one after the other. It initially creates one pod, and when the pod’s
container finishes, 
it creates the second pod, and so on, until five pods complete successfully. If one of the 
pods fails, the Job creates a new pod, so the Job may create more than five pods overall.


Running Job pods in parallel: multi-completion-parallel-batch-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: multi-completion-batch-job
spec:
  completions: 5                      1
  parallelism: 2                      2
  template:
    <same as in listing 4.11>
1 This job must ensure five pods complete successfully.
2 Up to two pods can run in parallel.

By setting parallelism to 2, the Job creates two pods and runs them in parallel:
As soon as one of them finishes, the Job will run the next pod, until five pods finish successfully.


You can even change a Job’s parallelism property while the Job is running. 
This is similar to scaling a ReplicaSet or ReplicationController, 
and can be done with the kubectl scale command:

$ kubectl scale job multi-completion-batch-job --replicas 3
job "multi-completion-batch-job" scaled

```
### pod time limit with job
```
A pod’s time can be limited by setting the activeDeadlineSeconds property in the pod spec. 
If the pod runs longer than that, the system will try to terminate it and will mark the Job as failed.

You can configure how many times a Job can be retried before it is marked as failed by 
specifying the spec.backoffLimit field in the Job manifest. 
If you don’t explicitly specify it, it defaults to 6.

```

### k8s jobs also support cron jobs
```
apiVersion: batch/v1beta1
kind: CronJob
spec:
  schedule: "0,15,30,45 * * * *"
  startingDeadlineSeconds: 15  
  
```  
 
 
