# VMWare section

## How to manually deploy a VM in VMWARE

1. Log into vSphere
2. Select Target Cluster
3. Right-Click -> Create new VM
4. Name it and give it a location
5. Chose resources: picj the host, cluster 
6. Storage selection: Pick the correct Guest OS  ( if not present we need an ISO in vsphere)
7. Customize Hardware: vCPU, memory, disk space, network adapter
8. Power on

## Ensuring  Resource Allocation

- keep a 4:1 ration for the CPU load to avoid problems with CPU in the host -> monitor host
- memory sizing: allocate just needed
- storage: use thin provisioning for flexibility
- network: allocate the vm in network where the connectivity needed is configured and nothing else except needed is alowed
- monitoring: if there is license: chech vrops for monitoring and performance

## Automate VM Deployment
- This will be done with ansible in a simple way