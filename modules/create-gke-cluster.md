Create a Kubernetes Cluster
===========================

Module objectives
-----------------

- create a cluster using `gcloud` tool
- configure `kubectl` tool

---

In this module, you will use Google Kubernetes Engine (GKE) managed service to deploy a Kubernetes cluster.

---

Theory
------

Google Kubernetes Engine (GKE) does containerized application management at scale. One may deploy a wide variety of applications to Kubernetes cluster and operate this cluster seamlessly with high availability.

One may scale both cluster and applications to meet increased demand and move applications freely between on-premise and cloud.

Kubernetes cluster consists of two types of nodes. Master nodes coordinate container placement and store cluster state. Worker nodes actually run the application containers.

---

1. Ensure all apis we may use are enabled, make sure you are running on cloud shell.

    ```shell
    gcloud services enable \
      container.googleapis.com \
      compute.googleapis.com \
      containerregistry.googleapis.com \
      cloudbuild.googleapis.com \
      sourcerepo.googleapis.com \
      monitoring.googleapis.com \
      logging.googleapis.com \
      stackdriver.googleapis.com
    ```


1. Create a cluster running two `n1-standard-2` worker nodes

    ```shell
    gcloud container clusters create jenkins-cd \
    --num-nodes 2 \
    --machine-type n1-standard-2 \
    --cluster-version 1.11.8-gke.6 \
    --labels=project=jenkins-workshop \
    --image-type COS \
    --enable-autorepair \
    --no-enable-basic-auth \
    --no-issue-client-certificate \
    --enable-ip-alias \
    --metadata disable-legacy-endpoints=true \
    --scopes "https://www.googleapis.com/auth/projecthosting,cloud-platform,compute-rw,storage-rw,service-control,service-management"
    ```

    Output:
    ```console
    kubeconfig entry generated for jenkins-cd.
    NAME        LOCATION    MASTER_VERSION  MASTER_IP       MACHINE_TYPE   NODE_VERSION  NUM_NODES  STATUS
    jenkins-cd  us-west1-c  1.11.8-gke.6    35.235.101.300  n1-standard-2  1.11.8-gke.6  2          RUNNING
    ```

1. Get credentials for the cluster

    ```console
    gcloud container clusters get-credentials jenkins-cd
    ```

    Output:
    ```console
    Fetching cluster endpoint and auth data.
    kubeconfig entry generated for jenkins-cd.
    ```

1. Verify that you can connect to the cluster and list the nodes

    ```shell
    kubectl get nodes
    ```
    This command should display all cluster nodes. In GCP console open 'Compute Engine' -> 'VM instances' to verify that each node has a corresponding VM.


Module summary
--------------

You created a GKE Kubernetes cluster, configured `kubectl` CLI and deployed Helm.

In the next module, you will deploy Jenkins.

Optional Exercises
-------------------

### Resize node pool and then create new node pool

1. The common operation when manging k8s clusters is to change the capacity of the cluster.
   Try to add one node using command `gcloud beta container clusters resize`. You may always get help with the `--help` flag.

1. Sometimes you need to migrate your workloads to a different node pool. In this exercise don't forget to match the scope of the current node pool, otherwise   you will not have some needed permissions later. Create a new node pool, and then migrate the workload to it (At this point only what is in kube-system), then delete the default node pool.

  - create a new node pool. For simplicity, it should be identical to the default node pool you have created before when provisioning a cluster (use `gcloud container node-pools` command)
  - drain the nodes to trigger eviction of the running pods (use `kubectl drain` command)
  - delete the default node pool when all the pods are running (use `gcloud container node-pools` command)
  - we also found some link between node pool having `--enable-autoupgrade` turned on is needed to use network policies, at least for them to be enabled post creation of cluster.
