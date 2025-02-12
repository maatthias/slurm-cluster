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
podman exec -it saturn4 bash
```
- Allocate and run
```sh
[root@saturn4 ~]# salloc -M nsls2 --partition=jupyter-staff-chx --job-name=spawner-jupyterhub --ntasks=1 --time=7-1 --mem=0 --cpus-per-task=1
salloc: Granted job allocation 2
salloc: Nodes saturn4 are ready for job

[root@saturn4 slurmstepd.scope]# scontrol show job 2
JobId=2 JobName=spawner-jupyterhub
   UserId=root(0) GroupId=root(0) MCS_label=N/A
   Priority=313940 Nice=0 Account=root QOS=normal
   JobState=RUNNING Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=0 Reboot=0 ExitCode=0:0
   RunTime=00:00:34 TimeLimit=7-01:00:00 TimeMin=N/A
   SubmitTime=2025-02-12T17:51:23 EligibleTime=2025-02-12T17:51:23
   AccrueTime=Unknown
   StartTime=2025-02-12T17:51:23 EndTime=2025-02-19T18:51:23 Deadline=N/A
   SuspendTime=None SecsPreSuspend=0 LastSchedEval=2025-02-12T17:51:23 Scheduler=Main
   Partition=jupyter-staff-chx AllocNode:Sid=saturn4:351
   ReqNodeList=(null) ExcNodeList=(null)
   NodeList=saturn4
   BatchHost=saturn4
   NumNodes=1 NumCPUs=1 NumTasks=1 CPUs/Task=1 ReqB:S:C:T=0:0:*:*
   ReqTRES=cpu=1,mem=1000M,node=1,billing=1
   AllocTRES=cpu=1,node=1,billing=1
   Socks/Node=* NtasksPerN:B:S:C=0:0:*:* CoreSpec=*
   MinCPUsNode=1 MinMemoryNode=0 MinTmpDiskNode=0
   Features=(null) DelayBoot=00:00:00
   OverSubscribe=YES Contiguous=0 Licenses=(null) Network=(null)
   Command=/bin/bash
   WorkDir=/root
   TresPerTask=cpu=1
   

sbatch -t 2-00:00:00 --qos=long -n 30 --wrap="srun hostname"

sbatch -vvvvv <<EOF
#!/bin/bash
#SBATCH --output=/tmp/slurm_job_%j.log
#SBATCH --job-name=slurm_job
#SBATCH --ntasks=1
#SBATCH --partition=jupyter-staff-chx
#SBATCH --time=7-1
#SBATCH --mem=0
#SBATCH --cpus-per-task=1

hostname && sinfo
EOF
```