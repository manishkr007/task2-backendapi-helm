## Deploy backend API on a Kubernetes cluster using Helm

### Prerequisites :-

1. Docker Desktop
2. Kubernetes CLI (kubectl)
3. Helm
4. System Requirements (Minimum 4GB RAM, 10GB disk space)
5. Access Requirements
   - Administrative privileges (for Docker Desktop installation)
   - Internet connection (for pulling container images)

### To run the application :-

Execute below commands and then visit http://127.0.0.1:8080 :  
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=backend-api,app.kubernetes.io/instance=backend-api" -o jsonpath="{.items[0].metadata.name}")  
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")  
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT  

To get the message printed by Cron Job, execute below commands :  
  kubectl get pods --sort-by=.metadata.creationTimestamp | grep cronjob | tail -n 1  
  kubectl logs backend-api-cronjob-29130258-vr7ft  
  Or,  
  kubectl logs $(kubectl get pods --sort-by=.metadata.creationTimestamp | grep cronjob | tail -n 1 | awk '{print $1}')  

### Assumptions :-

1. Autoscaling considered to deploy on at least 3 Pod replicas to 5 replicas.
2. Scaling condition kept on CPU and RAM utilization.
3. Limits and request set on resource allocation.
4. CronJob uses busybox image.
5. For CronJob, Pods of only 3 most recent successful and 3 most recent failure are kept.

### Common Issues and Resolutions :-

1. Error: INSTALLATION FAILED: Kubernetes cluster unreachable: Get "https://127.0.0.1:62344/version": dial tcp 127.0.0.1:62344: connectex: No connection could be made because the target machine actively refused it.  
  Reason: Kubernetes cluster not running on machine.  
  Resolution: Run the kubernetes cluster using docker desktop.  

2. Error: There are multiple pods for cronJob with Completed status.  
  Reason: cronJob ran successfully but there is no configuration to regularly remove those pods for each execution.  
  Resolution: Tested with frequency of 1 min instead of 30 mins.  