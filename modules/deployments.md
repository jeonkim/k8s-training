## Deployments

A deployment is a supervisor for pods and replica sets, giving you fine-grained control over how and when a new pod version is rolled out as well as rolled back to a previous state.

### Exercise 1: Create a deployment

1. Save the following file as `deployment.yaml`.
    ```console
    cat > deployment.yaml <<EOF
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: simpleservice
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: simpleservice
      template:
        metadata:
          labels:
            app: simpleservice
        spec:
          containers:
          - name: simpleservice
            image: mhausenblas/simpleservice:0.5.0
            ports:
            - containerPort: 9876
            env:
            - name: SIMPLE_SERVICE_VERSION
              value: "0.9"
    EOF
    ```

1. Create deployment.
    ```console
    kubectl create -f deployment.yaml
    ```

1. Check deployment, replica set and pods, created by the previous command.
    ```console
    kubectl get deploy
    kubectl get rs
    kubectl get pods
    ```
    Copy pod IP address

1. SSH to any kubernetes node and query app info.
    ```shell
    PODIP=$(kubectl describe $(kubectl get pod -l app=simpleservice -o name) |awk '/IP/ {print $2;exit}')
    gcloud compute ssh $(gcloud compute instances list|awk '/node/ {print $1;exit}') -- "curl $PODIP:9876/info;echo"
    ```
    Make sure that simpleservice returns version `0.9`

    Output:
    ```yaml
    {"host": "100.96.2.5:9876", "version": "0.9", "from": "10.138.0.4"}
    Connection to 35.233.191.94 closed.
    ```

1. Update `deployment.yaml` and set `SIMPLE_SERVICE_VERSION` to `1.0`.

1. Apply changes.
    ```
    kubectl apply -f deployment.yaml
    ```
1. Run
    ```
    kubectl get pods
    ```
    What we now see is the rollout of two new pods with the updated version 1.0 as well as the two old pods with version 0.9 being terminated.
    ```
    NAME                                 READY     STATUS        RESTARTS   AGE
    simpleservice-5f5fb45496-6gk2j   1/1       Terminating   0          17m
    simpleservice-5f5fb45496-v5czm   1/1       Terminating   0          17m
    simpleservice-84f7f575d8-pvwcq   1/1       Running       0          15s
    simpleservice-84f7f575d8-qt254   1/1       Running       0          17s
    ```

1. Make sure that new replica set has been created.
    ```
    kubectl get rs
    ```

1. Once again get pod ip, ssh to one of the nodes and send a request to simpleservice.
    ```
    PODIP=$(kubectl describe $(kubectl get pod -l app=simpleservice -o name) |awk '/IP/ {print $2;exit}')
    gcloud compute ssh $(gcloud compute instances list|awk '/node/ {print $1;exit}') -- "curl $PODIP:9876/info;echo"
    ```
    Make sure that version now is "1.0"

1. Check deployment history.
    ```
    kubectl rollout history deploy/simpleservice
    ```

1. If there are problems in the deployment Kubernetes will automatically roll back to the previous version, however you can also explicitly roll back to a specific revision, as in our case to revision 1 (the original pod version).
    ```
    kubectl rollout undo deploy/simpleservice --to-revision=1
    ```
    At this point in time we're back at where we started, with two new pods serving again version 0.9.

1. One question we get a lot is if we can set custom messages for deployment change. Here is a way to do that.

    This is format:
    ```shell
    kubectl patch RESOURCE RESOURCE_NAME  --patch '{"metadata": {"annotations": {"my-annotation-key": "my-annotation-value"}}}'
    ```

    Let's patch our simpleservice:
    ```console
    kubectl rollout history deploy/simpleservice
    ```

    Output before patch:
    ```console
    REVISION  CHANGE-CAUSE
    2         <none>
    3         <none>
    ```

    Patch typically would be done in CI, but could be done manually like this:
    ```shell
    kubectl patch deployment simpleservice --patch '{"metadata": {"annotations": {"kubernetes.io/change-cause": "Rollback v1.0 had issues"}}}'
    ```

    Get history once again:
    ```console
    kubectl rollout history deploy/simpleservice
    ```

    Output after patch:
    ```console
    REVISION  CHANGE-CAUSE
    2         <none>
    3         Rollback v1.0 had issues
    ```

### Exercise 2 (Optional): Observe how kubernetes restarts containers

1. Use the simpleservice deployment
1. Exec into the container, find and kill web server process
1. Observe whether kubernetes tries to redeploy container

### Clean-up

1. Delete the deployment.
    ```
    kubectl delete deployment simpleservice
    ```
