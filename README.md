# Slurm Cluster Project
- mission: troubleshoot cgroups settings and tunings
- quick swappable configs
- adapted from https://github.com/giovtorres/slurm-docker-cluster.git

## Adapted to support
- fedora 41
- podman
- support for cgroupsv2
    - using podman for systemd support
- slurm latest veresion (v24.11.1)
- deliberate separation of run steps for faster build with changes

## Build and Teardown
### Edit the configs
- `slurm.conf`
- `slurmdbd.conf`
- `cgroup.conf`

```sh
podman build --target slurmdbd --tag slurmdbd .
podman build --target slurmctld --tag slurmctld .
podman build --target slurmd --tag slurmd .
podman-compose up
podman-compose down --volumes
```

## Interact
- Check
```sh
podman exec -it c1 sinfo
```
- Allocate and run
```sh
podman exec -it c1 salloc -A nsls2 -p normal -n 1 --mem=1MB
salloc: Granted job allocation 1
salloc: Nodes c1 are ready for job

[root@c1 munge-0.5.13]# squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
                 1    normal interact     root  R       2:57      1 c1

[root@c1 munge-0.5.13]# scontrol show job 1
JobId=1 JobName=interactive
   UserId=root(0) GroupId=root(0) MCS_label=N/A
   Priority=1 Nice=0 Account=root QOS=normal
   JobState=RUNNING Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=0 Reboot=0 ExitCode=0:0
   RunTime=00:03:15 TimeLimit=5-00:00:00 TimeMin=N/A
   SubmitTime=2025-02-03T11:26:30 EligibleTime=2025-02-03T11:26:30
   AccrueTime=Unknown
   StartTime=2025-02-03T11:26:30 EndTime=2025-02-08T11:26:30 Deadline=N/A
   SuspendTime=None SecsPreSuspend=0 LastSchedEval=2025-02-03T11:26:30 Scheduler=Main
   Partition=normal AllocNode:Sid=c1:135
   ReqNodeList=(null) ExcNodeList=(null)
   NodeList=c1
   BatchHost=c1
   NumNodes=1 NumCPUs=1 NumTasks=1 CPUs/Task=1 ReqB:S:C:T=0:0:*:*
   ReqTRES=cpu=1,mem=1000M,node=1,billing=1
   AllocTRES=cpu=1,mem=1000M,node=1,billing=1
   Socks/Node=* NtasksPerN:B:S:C=0:0:*:* CoreSpec=*
   MinCPUsNode=1 MinMemoryNode=0 MinTmpDiskNode=0
   Features=(null) DelayBoot=00:00:00
   OverSubscribe=OK Contiguous=0 Licenses=(null) Network=(null)
   Command=/bin/bash
   WorkDir=/tmp/munge-0.5.13

[root@c1 munge-0.5.13]# srun hostname
c1

sbatch -t 2-00:00:00 --qos=long -n 30 --wrap="srun hostname"

sbatch -vvvvv <<EOF
#!/bin/bash
#SBATCH --output=/tmp/slurm_job_%j.log
#SBATCH --job-name=slurm_job
#SBATCH --ntasks=1
#SBATCH --partition=normal

hostname && sinfo
EOF
```