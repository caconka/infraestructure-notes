# Gcloud, kubectl and helm

## gcloud

### Credentials

1. `gcloud init`

2. `gcloud container clusters get-credentials <dev/prod> --zone europe-west1-b
--project <project-id>`
	- Cluster nuevo DEV:
		`gcloud beta container clusters get-credentials <project-id> --region
		europe-west1 --project <project-id>`

- Default credentials:
	`gcloud auth application-default login`

### Cryptokeys

- List cryptokeys:
	`gcloud kms keys list --project <project-cloud> --location europe --keyring
	<keyring-name>`

## helm

### Releases

- List:
	`helm list` \
		`helm ls --tiller-namespace <tiller-ns>`

	`helm ls -dr -m 1024` \
		`-d` lo ordena por fecha \
		`-r` reverse \
		`-m` para ponerle un máximo, sino puede no listarte todos

- Delete pod:
	`helm delete --purge <release-name>` \
	`helm delete --tiller-namespace <tiller-ns> --purge <release-name>`

### Downgrade version

- https://stackoverflow.com/questions/50701224/helm-incompatible-versions-between-client-and-server
	```
	$ brew unlink kubernetes-helm
	$ brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/78d64252f30a12b6f4b3ce29686ab5e262eea812/Formula/kubernetes-helm.rb
	$ brew switch kubernetes-helm 2.9.1
	```

### Encrypt

- Decrypt secret:
	`helm secrets --kube-context <context-name> --tiller-namespace <tiller-ns>
	dec secrets.yaml`

## kubectl

### Deploy

- Create deployment:
	`kubectl create deployment <deploy-name> --image=<app-image>` \
	<app-image> app image location (include the full repository url for images
	hosted outside Docker hub)

- List deployments:
	`kubectl get deployments`

- Expose the Pod to the public internet:
	`kubectl expose deployment <deployment-name> --type=LoadBalancer
	--port=8080 --type=LoadBalancer` \
	flag indicates that you want to expose your Service outside of the cluster.

- List services:
	`kubectl get services`

- Clean up:
	`kubectl delete service <deployment-name>` \
	`kubectl delete deployment <deployment-name>`

### Config

- View config:
	`kubectl config view`

- List contexts:
	`kubectl config get-contexts`

- Switch to specific context:
	`kubectl config use-context <context-name>`

### PODs

- Get pods:
	`kubectl --context <context-name> -n <namespace> get po`

- Get pods wide:
	`kubectl get po -o wide`

- Get pods filtering by status.phase Running
	`kubectl get pods --field-selector status.phase=Running`

- Logs pods:
	`kubectl -n <namespace> logs <pod-name> -c <container-name>` \
		`-f` forward (kubectl -n <namespace> logs -f <id-pod> -c <container-name>) \
		`-p` previous (kubectl -n <namespace> logs -f <id-pod> -c <container-name>)

- Get ip/services:
	`kubectl get svc`

- Get ingresses:
	`kubectl get ingresses`

- Get urls/host:
	`kubectl get vs`

- Get configmaps:
    `kubectl get configmaps <pod-config-name> -o yaml`

#### Interact with pod (exec)

- Connect pod:
	`kubectl exec -it <pod-name> -n <namespace> -- bash` \
	`kubectl exec -it <pod-name> -n <namespace> sh`

- Env vars:
    `kubectl exec <pod-name> env`

- Run commands in pod
	`kubectl exec <pod-name> <bash commands...>` \
	`kubectl exec <pod-name> ls /`

### Policies

- List policies
	`kubectl get policies`

- View policy
	`kubectl get policies <policy-name> -o yaml`

### Cronjobs

- Get cronjobs:
	`kubectl get cronjob`

- Lanzar un cronjob:
	`kubectl create job test --from=cronjob/<batch-name>`

- Update schedule:
	`kubectl patch cronjob <batch-name> -p '{"spec":{"schedule": "0 10 * * *"]'`

### Secrets

- Get secrets:
	`kubectl get secrets`

- Get secret:
	`kubectl get secrets <secret-name> -o yaml` \
	`kubectl describe secrets <secret-name>`

- Decode base64:
	`echo 'BASE64' | base64 --decode`

- Create secret
	- From file:
		`kubectl create secret -n <namespace> generic <secret-name> --dry-run
		--from-file=keyfile.json -o json > plain-secret.json`
	- From literal
		```
		kubectl create secret -n <namespace> generic <secretname> --dry-run \
		--from-literal=auth="Basic foo" --from-literal=secondKey="bar" -o json \
		> plain-secret.jsonk
		```
- Encrypt secret:
	`kubeseal -n <namespace> <plain-secret.json> secret.json`

- Upload secret:
	`kubectl create -f secret.json -n <namespace>`

### Redis
1. Connect pod:
	`kubectl exec -it <pod-id> -n <namespace> -- bash`

2. Run redis:
	`redis-cli -h redis`

- Connect cache prod number 2:
	`kubectl exec -ti
	$(kubectl get po -l run=redis-console -o=jsonpath="{range .items[*]}{.metadata.name}{end}")
	-- redis-cli -n 2 -h redis.<namespace>`

#### Comandos

- Select dbs:
	`select #db`

- List keys:
	`keys *`

- Remove a key:
	`del <nombre-key>`

- Remove all keys from selected database:
	`flushdb`

- Remove all keys from all databases:
	`flushall`

Si hay varios pods hay que ir probando uno a uno hasta encontrar uno que tenga
la propiedad `role:master`
``` bash
kubectl -n <namespace> exec -it <pod-name> sh

redis-cli -h redis
127.0.0.1:6379> info
...
Replication
role:master
...
```

### Deployment

- List ReplicaSets:
	`kubectl get rs`

- List deployments:
	`kubectl get deploy`

- Get deployment:
	`kubectl get deployment <deploy-name>`

- Describe deployment:
	`kubectl describe deployment <deploy-name>`

- View details of revision:
	`kubectl rollout history deployment <deploy-name> --revision=<rev-number>`

- Rollback:
	`kubectl rollout undo deployment <deploy-name>` \
	Especificando la version a la que queremos volver:
	`kubectl rollout undo deployment <deploy-name> --to-revision=<rev-number>`


## docker

### containers

- Stop / remove all dockers containers:
	`docker stop $(docker ps -a -q)` \
	`docker rm $(docker ps -a -q)`
