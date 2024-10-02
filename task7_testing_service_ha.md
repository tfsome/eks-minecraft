# Testing Minecraft server resilience

## Agenda

In this lab we will check how resilient are our solution.

## Perform resilience testing

We will use CloudWatch dashboard for monitoring test results.


### Delete 1 worker pod
- Start Minecraft and connect to your server. Play for a while.
- Select one of minecraft-worker pod and terminate it using `kubectl -n minecraft delete pod {pod-name}` command.
- Observe logs and metrics. Try to continue play in the same world. What happend? Does your progress saved?
### Delete master pod
- Terminate minecraft-master pod  using `kubectl -n minecraft delete pod {pod-name}` command. What happend? Does your progress saved?
- Observe logs and metrics. Try to continue play in the same world.

### Delete node with worker pod only
- Wait for a while and terminate a node(EC2 instance) which runs worker pod only(meaning that you have only 2 nodes in a node group and anti-afinity is configured for worker pods).
The following command could helps you to get pod-node assignement:
```
kubectl -n minecraft get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```
- Observe logs and metrics. Try to continue play in the same world. What happend? 

### Delete node with master and worker pod
- Wait for a while and terminate a node(EC2 instance) which runs both master and worker pods(meaning that you have only 2 nodes in a node group and anti-afinity is configured for worker pods).
The following command could helps you to get pod-node assignement:
```
kubectl -n minecraft get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```
- Observe logs and metrics. Try to continue play in the same world. What happend? 
- Are your solution highly available? What changes need to be done to the infrastructure?

## Definition of done

You can successfully connect and play to your Minecraft server with only one instance of minecraft-worker running. Cluster should run a new node and create pods after node removal to restore normal service with stored progress in Minecraft world.

## Clean-up

Do not forget to stop and delete your resources on the end of practice. You can use Tags to locate required resources.


