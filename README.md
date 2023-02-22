# Launch Ansible jobs upon ztp-sno label

This repo includes the files to monitor for ZTP workflow progress in the `managedcluster` custom resource (CR).

1. After a cluster (deployed by Telco ZTP) receives the `ztp-done` label, we are now in a good spot to trigger any 
external provisioning using Automation Controller (or Tower).

2. This work reads the Automation Controller (or Tower) info from a secret (sample below), and launches a job template 
right after the `ztp-done` is applied to a managed cluster.


## To install

First fill in the [aap-controller-token.yaml](01-aap-controller-token.yaml) with the proper info to access your 
Automation Controller / Tower instance, as well as the job template that you want to trigger (see sample below).

```yaml
---
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: aap-controller-token
  namespace: ztp-aap-integration
  labels:
    cluster.open-cluster-management.io/type: ans
    cluster.open-cluster-management.io/credentials: ""
stringData:
  host: https://automation-aap.apps.zt-sno1.inbound.vz.bos2.lab     # <- variable provided by admin
  token: aSTRwLrS9URWIGXqxQ1hstuDpZkKey                             # <- variable provided by admin
  job_template: ztp-day2-automation-template                        # <- variable provided by admin
```

Then, to install all the required objects you may use the provided [kustomization.yaml](kustomization.yaml) file.

```shell
-> oc get all
NAME                                 READY   STATUS      RESTARTS   AGE
pod/ztp-aap-cronjob-27951325-fwcpq   0/1     Completed   0          5m19s
pod/ztp-aap-cronjob-27951330-6tcxt   0/1     Completed   0          19s

NAME                            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/ztp-aap-cronjob   */5 * * * *   False     0        19s             7m10s

NAME                                 COMPLETIONS   DURATION   AGE
job.batch/ztp-aap-cronjob-27951325   1/1           10s        5m19s
job.batch/ztp-aap-cronjob-27951330   1/1           9s         19s
```

The logs should look similar to this:

```shell
-> oc logs pod/ztp-aap-cronjob-27951325-fwcpq
# 2023-02-22T15:25:02Z Sync started
# 2023-02-22T15:25:02Z Sync 110740051	/tmp/launch_aap_jobs zt-sno1 ""

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0{"job":1655,"ignored_fields":{},"id":1655,"type":"job","url":"/api/v2/jobs/1655/","related":{"created_by":"/api/v2/users/1/","modified_by":"/api/v2/users/1/","labels":"/api/v2/jobs/1655/labels/","inventory":"/api/v2/inventories/2/","project":"/api/v2/projects/8/","organization":"/api/v2/organizations/1/","credentials":"/api/v2/jobs/1655/credentials/","unified_job_template":"/api/v2/job_templates/10/","stdout":"/api/v2/jobs/1655/stdout/","job_events":"/api/v2/jobs/1655/job_events/","job_host_summaries":"/api/v2/jobs/1655/job_host_summaries/","activity_stream":"/api/v2/jobs/1655/activity_stream/","notifications":"/api/v2/jobs/1655/notifications/","create_schedule":"/api/v2/jobs/1655/create_schedule/","job_template":"/api/v2/job_templates/10/","cancel":"/api/v2/jobs/1655/cancel/","relaunch":"/api/v2/jobs/1655/relaunch/"},"summary_fields":{"organization":{"id":1,"name":"Default","description":""},"inventory":{"id":2,"name":"hub-acm-inventory","description":"","has_active_failures":false,"total_hosts":4,"hosts_with_active_failures":0,"total_groups":8,"has_inventory_sources":true,"total_inventory_sources":1,"inventory_sources_with_failures":0,"organization_id":1,"kind":""},"project":{"id":8,"name":"aap-for-ran-ztp-project","description":"","status":"successful","scm_type":"git","allow_override":false},"job_template":{"id":10,"name":"ztp-day2-automation-template","description":""},"unified_job_template":{"id":10,"name":"ztp-day2-automation-template","description":"","unified_job_type":"job"},"created_by":{"id":1,"username":"admin","first_name":"","last_name":""},"modified_by":{"id":1,"username":"admin","first_name":"","last_name":""},"user_capabilities":{"delete":true,"start":true},"labels":{"count":0,"results":[]},"credentials":[{"id":3,"name":"hub-acm-kubeconfig","description":"","kind":null,"cloud":true}]},"created":"2023-02-22T15:25:04.710949Z","modified":"2023-02-22T15:25:04.729764Z","name":"ztp-day2-automation-template","description":"","job_type":"run","inventory":2,"project":8,"playbook":"playbooks/ztp-day2-automation-labeled.yml","scm_branch":"","forks":0,"limit":"","verbosity":0,"extra_vars":"{\"state\": \"present\", \"namespace\": \"ztp-day2-automation\", \"git_protocol\": \"http\", \"git_repo_url\": \"jumphost.inbound.vz.bos2.lab:3000/kni/aap-for-ztp-bos2.git\", \"git_scm_version\": \"master\", \"git_user\": \"---\", \"git_password\": \"---\", \"vault_id\": \"ztp\", \"vault_password\": \"---\", \"target_clusters\": [\"zt-sno1\"]}","job_tags":"","force_handlers":false,"skip_tags":"","start_at_task":"","timeout":0,"use_fact_cache":false,"organization":1,"unified_job_template":10,"launch_type":"manual","status":"pending","execution_environment":null,"failed":false,"started":null,"finished":null,"canceled_on":null,"elapsed":0.0,"job_args":"","job_cwd":"","job_env":{},"job_explanation":"","execution_node":"","controller_node":"","result_traceback":"","event_processing_finished":false,"launched_by":{"id":1,"name":"admin","type":"user","url":"/api/v2/users/1/"},"work_unit_id":null,"job_template":10,"passwords_needed_to_start":[],"allow_simultaneous":true,"artifacts":{},"scm_revision":"","instance_group":null,"diff_mode":false,"job_slice_number":0,"job_slice_count":1,"webhook_service":"",100  3346  100  3298  100    48   9161    133 --:--:-- --:--:-- --:--:--  9294
Launching job template called ztp-day2-automation-template in Automation Controller/Tower for zt-sno1


# 2023-02-22T15:25:04Z Sync 110740200	/tmp/launch_aap_jobs zt-sno2 ""

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0{"job":1658,"ignored_fields":{},"id":1658,"type":"job","url":"/api/v2/jobs/1658/","related":{"created_by":"/api/v2/users/1/","modified_by":"/api/v2/users/1/","labels":"/api/v2/jobs/1658/labels/","inventory":"/api/v2/inventories/2/","project":"/api/v2/projects/8/","organization":"/api/v2/organizations/1/","credentials":"/api/v2/jobs/1658/credentials/","unified_job_template":"/api/v2/job_templates/10/","stdout":"/api/v2/jobs/1658/stdout/","job_events":"/api/v2/jobs/1658/job_events/","job_host_summaries":"/api/v2/jobs/1658/job_host_summaries/","activity_stream":"/api/v2/jobs/1658/activity_stream/","notifications":"/api/v2/jobs/1658/notifications/","create_schedule":"/api/v2/jobs/1658/create_schedule/","job_template":"/api/v2/job_templates/10/","cancel":"/api/v2/jobs/1658/cancel/","relaunch":"/api/v2/jobs/1658/relaunch/"},"summary_fields":{"organization":{"id":1,"name":"Default","description":""},"inventory":{"id":2,"name":"hub-acm-inventory","description":"","has_active_failures":false,"total_hosts":4,"hosts_with_active_failures":0,"total_groups":8,"has_inventory_sources":true,"total_inventory_sources":1,"inventory_sources_with_failures":0,"organization_id":1,"kind":""},"project":{"id":8,"name":"aap-for-ran-ztp-project","description":"","status":"running","scm_type":"git","allow_override":false},"job_template":{"id":10,"name":"ztp-day2-automation-template","description":""},"unified_job_template":{"id":10,"name":"ztp-day2-automation-template","description":"","unified_job_type":"job"},"created_by":{"id":1,"username":"admin","first_name":"","last_name":""},"modified_by":{"id":1,"username":"admin","first_name":"","last_name":""},"user_capabilities":{"delete":true,"start":true},"labels":{"count":0,"results":[]},"credentials":[{"id":3,"name":"hub-acm-kubeconfig","description":"","kind":null,"cloud":true}]},"created":"2023-02-22T15:25:06.591033Z","modified":"2023-02-22T15:25:06.612917Z","name":"ztp-day2-automation-template","description":"","job_type":"run","inventory":2,"project":8,"playbook":"playbooks/ztp-day2-automation-labeled.yml","scm_branch":"","forks":0,"limit":"","verbosity":0,"extra_vars":"{\"state\": \"present\", \"namespace\": \"ztp-day2-automation\", \"git_protocol\": \"http\", \"git_repo_url\": \"jumphost.inbound.vz.bos2.lab:3000/kni/aap-for-ztp-bos2.git\", \"git_scm_version\": \"master\", \"git_user\": \"---\", \"git_password\": \"---\", \"vault_id\": \"ztp\", \"vault_password\": \"---\", \"target_clusters\": [\"zt-sno2\"]}","job_tags":"","force_handlers":false,"skip_tags":"","start_at_task":"","timeout":0,"use_fact_cache":false,"organization":1,"unified_job_template":10,"launch_type":"manual","status":"pending","execution_environment":null,"failed":false,"started":null,"finished":null,"canceled_on":null,"elapsed":0.0,"job_args":"","job_cwd":"","job_env":{},"job_explanation":"","execution_node":"","controller_node":"","result_traceback":"","event_processing_finished":false,"launched_by":{"id":1,"name":"admin","type":"user","url":"/api/v2/users/1/"},"work_unit_id":null,"job_template":10,"passwords_needed_to_start":[],"allow_simultaneous":true,"artifacts":{},"scm_revision":"","instance_group":null,"diff_mode":false,"job_slice_number":0,"job_slice_count":1,"webhook_service":"","we100  3343  100  3295  100    48   9360    136 --:--:-- --:--:-- --:--:--  9497
Launching job template called ztp-day2-automation-template in Automation Controller/Tower for zt-sno2

Shutting down after 3s ...
```

## Considerations

- The resource we are using to monitor the `ztp-done` label in the `managerclusters` is a [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/).

- This has been configured to run every five minutes (i.e. `*/5 * * * *`), and the pod jobs it creates will be removed 
after 25 minutes (i.e. `ttlSecondsAfterFinished: 1500`).