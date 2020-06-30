
### PGDown 

User receives an email alert with below error message.  Reasons for Alerts are listed below,  in most cases the errors are temporary and CAAS related and heal by itself. 
- Network errors between the PODS and CAAS Master Nodes
- CAAS Node restarts
- PostgreSQL related issues.
#### Alert Message Example


| Labels/Label Values |
| -------------------------------------------- |
alertname = PrimaryDown
alert_value = 1
primary = true
service = postgresql
severity = critical
severity_num = 300
Annotations
summary = postgres_exporter(s) running on is unable to communicate with the configured database
Source

#### Alert Diagnosis

-  Assumptions
   -  Have access to Openshift Console
   -  PGO client is configured for Postgres cluster
   -  OC client is configured.
   -  Familiar with basic Postgres and Openshift commands

1. To identify if the Postgres Pod is not in Running Status. Perform folowing 

   1. Using Oc client login into openshift.  e.g. ``` oc login https://api.caas.ford.com:443 --token=xYacLpap_FW9rkhjBLQoJmfBY_y-nYmxV6Uvkw-wHEY ```
   1. Set project context e.g. ``` oc project metrics-pgsql-prod-prod ```
   1. Execute ``` oc get pods ``` it will display similar information below. 
      - In below information if pods status is anything other than running, we need to investigate further or give it few minutes to recover by itself. 
  
     ```
    Pod Information ....
    NAME                                                READY     STATUS      RESTARTS   AGE
    backrest-backup-metricsprod-rvldh                   0/1       Completed   0          3d
    metricsprod-backrest-incr-backup-schedule-fcr9v     0/1       Completed   0          19h
    metricsprod-backrest-shared-repo-6f48d88bff-fmz9f   1/1       Running     0          32d
    metricsprod-csrx-77b7869d48-8t65s                   3/3       Running     1          29d
    ```
     ```diff
    - metricsprod-qsok-768dcbfd-66m9g                     3/3       Unknown     0          25d 
   ```
    ```
    metricsprod-qsok-768dcbfd-f6pqc                     3/3       Running     0          35m
    monitoring-8447b6db9c-fbsgr                         3/3       Running     4          6d
    pgadmin4-7484f6684f-zmj4k                           1/1       Running     0          93d
    postgres-operator-7d9f44dff4-j6jg9                  3/3       Running     0          25d
    Checking cluster metricsprod 
    Checking pod metricsprod-csrx-77b7869d48-8t65s 
    Checking pod metricsprod-qsok-768dcbfd-66m9g 
    Checking pod metricsprod-qsok-768dcbfd-f6pqc 
    Checking pod metricsprod-csrx-77b7869d48-8t65s 
    Checking pod metricsprod-qsok-768dcbfd-66m9g 
   ```
      - If the status column shows "ErrImagePull" please go to Alert correction and follow the Pod delete instructions. 
   
    4.  Execute pgo client environment e.g. ``` source .pgo/prod/metrics-pgsql-prod-prod/pgop4.0.1/pgopenv ```
   1. List  Postgres Cluster e.g. pgo show cluster metricsprod,  it will displays similar information as below.
        - In below inforamtion if primary pod , displays number of ready containers not same as the expected containers than their may be an issue with the pod. 
   ```
    $ pgo show cluster metricsprod
        cluster : metricsprod (crunchy-postgres:rhel7-11.4-2.4.1)
        pod : metricsprod-csrx-77b7869d48-8t65s (Running) on worker2.caas.ford.com - (3/3) (replica) 
        pvc : metricsprod-csrx
        pod : metricsprod-qsok-768dcbfd-f6pqc (Running) on worker29.caas.ford.com (3/3) (primary)
        pvc : metricsprod-qsok
        resources : CPU Limit=1600m Memory Limit=4Gi, CPU Request=800m Memory Request=2Gi
        storage : Primary=100Gi Replica=100Gi
        deployment : metricsprod-backrest-shared-repo
        deployment : metricsprod-csrx
        deployment : metricsprod-gost
        deployment : metricsprod-qsok
        deployment : metricsprod-wzca
        service : metricsprod - ClusterIP (19.2.38.22)
        service : metricsprod-replica - ClusterIP (19.2.47.189)
        replica : metricsprod-dgwj
        replica : metricsprod-eavz
        replica : metricsprod-gjtw
        replica : metricsprod-gost
        replica : metricsprod-lfan
        replica : metricsprod-obvz
        labels : autofail=true crunchy-pgbadger=true name=metricsprod pgo-backrest=true        ```
   ```
#### Alert Correction   

1.  If the error continues to persist after checking on the pods by running commands in alert diagnosis ``` oc get pods ``` and ``` pgo show cluster --all ``` try steps below
    - Delete pod using oc command, exctract pod name from ``` oc get pods ``` issue ``` oc delete pod <pod name> ``` 
        -   e.g.  ``` oc delete pod metricsprod-qsok-768dcbfd-66m9g ```
    - Delete pod via Openshift Console.  Below are the steps
      1.  In Openshift Console ⇒ Application ⇒ Pods ⇒< Error Pod Name>⇒ Actions ⇒ Delete
        -   e.g. In Openshift Console ⇒ Application ⇒ Pods ⇒metricsprod-qsok-768dcbfd-66m9g⇒ Actions ⇒ Delete
 1.          
####  Alert validation  
1. Execute commands in alert diagnosis ``` oc get pods ``` and ``` pgo show cluster --all ``` , check if pod is in running


#### Notes
