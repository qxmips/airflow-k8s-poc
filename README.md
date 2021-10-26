# Deploying Airflow on kubernetes using official helm chart

## Instructions

1. create dedicated ns for airflow deployment  

```
kubectl create namespace airflow
```

2. Add helm repo:

```
helm repo add airflow-stable https://airflow-helm.github.io/charts
```

3. Prepair values  for the chart:

generate  FERNET_KEY:

```
python3 -c "from cryptography.fernet import Fernet; FERNET_KEY = Fernet.generate_key().decode(); print(FERNET_KEY)"
```

4. create secret and configmap  putting appropriate values in  the files in the repo.

```
kubectl apply -f  secrets.yaml
kubectl apply -f configmap.yaml 

```

5. define connection ids  in " connections: " , for example added spark on k8s connection.

6. add extra packages that need to be installed to configure

7. set appropriate s3 bucket name in   config.AIRFLOW__LOGGING__REMOTE_BASE_LOG_FOLDER  = "s3://qxmips-airflow-logs"
and assign a role with access to s3 bucket that can be assumed by pod: 

```
serviceAccount:
  create: true
  name: "airflow"
  annotations: 
    eks.amazonaws.com/role-arn: "arn:aws:iam::123456789123:role/airflow20211025170151352700000001"
```

8. in  gitSync spesify a repo with DAGs, and  create secrets for git authentication:

```
gitSync:
    ## if the git-sync sidecar container is enabled
    ##
    enabled: true
    repo: "git@github.com:USERNAME/REPOSITORY.git"
    branch: "master"
    revision: "HEAD"
    syncWait: 60
    sshSecret: "airflow-ssh-git-secret"
```

9.  Deploy helm charm using values fileL

```
helm install airflow airflow-stable/airflow  -n airflow  --version "8.5.2"  --values ./values.yaml
```


## Deploying new DAGs

You can use git-sync sidecar  container to sync a repo with DAGs to airflow.  This is the  recommended way.Using this method complies with gitops approach. If you got  multiple repos you can create a repository that consists of submodules, for more refer https://github.com/airflow-helm/charts/issues/434

Other option would be to use peristant volume  either existing or created by the chart. that method is less prefered bacause it adds aditional efforts to deliver DAGs inside the PV.

## Persisting logs

For production, you should persist logs in a production deployment using object storage like s3 or PersistentVolumeClaim. Generally s3 or other object storage is prefered as it has higher availability and durability levels.
By default, logs are stored within the container's filesystem, therefore any restart of the pod will wipe your DAG logs.
See the instructions steps above  explains how to configure s3 for storing the logs.
