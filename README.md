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
```sh
podman exec -it c1 bash
```