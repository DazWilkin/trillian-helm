## Trillian Helm (Experiment) ##

**Description**
: Prototype Helm Chart for Google [Trillian](github.com/google/trillian)

Extends the [Kubernetes Deployment](https://github.com/google/trillian/tree/master/examples/deployment/kubernetes) included in Trillian repo.

Chart assumes:
* A cluster comprising arbitrary Node Pool
* etcd Operator is provisioned to the cluster
* Cloud Spanner is provisioned to the GCP Project
* Secret exists representing a GCP service account

### Aside: Helm installation ###
The following got me a working Helm (tiller) installation on a Kubernetes Engine (1.10.x) cluster w/ RBAC:
```
kubectl create serviceaccount tiller \
--namespace=kube-system

kubectl create clusterrolebinding tiller \
--clusterrole cluster-admin \
--serviceaccount=kube-system:tiller

helm init --service-account=tiller
```

### Install Chart ###

Chart assumes bulk of `/.create.sh` has been run:
* Kubernetes cluster created and auth'd to Helm user
* Cloud Spanner service enabled, instance created and DB provisioned
* GCP Service Account created, IAM policy revised and...
* Kubernetes Secret created for the GCP Service Account
* etc Operator deployed and Trillian (`trillian-etcd-cluster`) created
* Helm is installed and Tiller is deployed to the cluster

Clone this github repo but remain in the parent directory.

You may dry-run the deployment to verify that it does what you expect:

```
helm install ./trillian-helm \
--debug \
--dry-run \
--set \
gcp.project.id=${PROJECT_NAME},\
registry.project=${PROJECT_NAME}
```

Or you may apply the Chart to the cluster:

```
helm install ./trillian-helm \
--debug \
--dry-run \
--set \
gcp.project.id=${PROJECT_NAME},\
registry.project=${PROJECT_NAME}
```
**NB** `gcp.project.id` should be assigned to the Project ID of the Google Cloud Platform (GCP) project containing the Kubernetes cluster and the Cloud Spanner instance and database.

**NB** `registry.project` should be assigned to the Project ID of the GCP project containing the Google Container Registry containing Trillian's Docker images. This *may* have the same value as `gcp.project.id`

**NB** If your Docker images are elsewhere, you may specify the service using `registry.host` (e.g. `docker.io`) and leave `registry.project` unassigned to reference dockerhub.

**NB** `${PROJECT_NAME}` back-references the environment variable used by Trillian's Kubernetes deployment (bash) scripts as defined in `config.sh`. 

You should expect to see an enumeration of the Kubernetes resources created by the Chart that resembles (!) this:
```
NAME:   intentional-fish
LAST DEPLOYED: Sun Jun  3 12:00:00 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME           DATA  AGE
deploy-config  8     0s

==> v1beta1/Deployment
NAME                 DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
trillian-log-server  4        4        4           0          0s
trillian-log-signer  2        2        2           0          0s

==> v1/Pod(related)
NAME                                  READY  STATUS             RESTARTS  AGE
trillian-log-server-6c64fd84f4-24m54  0/2    ContainerCreating  0         0s
trillian-log-server-6c64fd84f4-8bbc4  0/2    ContainerCreating  0         0s
trillian-log-server-6c64fd84f4-9976f  0/2    ContainerCreating  0         0s
trillian-log-server-6c64fd84f4-vjlsn  0/2    ContainerCreating  0         0s
trillian-log-signer-557854dc86-2ngq4  0/2    ContainerCreating  0         0s
trillian-log-signer-557854dc86-4rk47  0/2    Pending            0         0
```

### List Chart ###
Either of the following will give you a quick summary of the state of the cluster and the Chart:
```
kubectl get all
```
or:
```
helm list

NAME            	REVISION	UPDATED                 	STATUS  	CHART         	NAMESPACE
intentional-fish	1       	Sun Jun  3 12:00:00 2018	DEPLOYED	trillian-0.0.1	default 
```

### Delete Chart ###
If you wish to delete the Chart (which you should do before redeploying) grab the `NAME` from the result of the list command:
```
helm delete intentional-fish
```
This will be useful [Optional output as JSON and YAML](https://github.com/seaneagan/helm/commit/b215e72eade80554bb6b084ee4631151913e3c62)

### Differences ###

* Chart uses CPM|RAM constraints (not names) to deploy to Node Pools

### Future Development ###

* Support for multiple Node Pools