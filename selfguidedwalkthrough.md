# Storage Gateway Self-Paced Setup Guide
## Deploying Storage Gateway on-premises brings an easy way to migrate storage workloads to the AWS cloud.  Hybrid storage deployment over Storage Gateway enables local access to cloud resources and scalability for both existing and new workloads in the datacenter.

### Choosing which Storage Gateway deployment strategy will depend upon the workload being deployed.

* File Gateway
  * Supporting existing file-based applications
  * Filesystem targets for backup solutions
* Volume Gateway
  * SAN-like storage with cloud recovery
  * iSCSI block devices with S3-backed storage
* Tape Gateway
  * Migrate tape-based backups to AWS
  * Support existing tape backup software

### Each Storage Gateway virtual appliance can operate in only one of the three available modes, so each of the above configurations will require a separate VM to run inside a customer's datacenter.  These deployments follow similar steps to one another, but have unique characteristics and steps to configure them.  These steps are outlined in the Storage Gateway user guide, and follow this general structure.

Log into AWS console
Navigate to **Storage Gateway** under **Services**
Choose File, Volume, or Tape gateway type
Select hypervisor host platform
Download virtual appliance for the hypervisor
Choose endpoint type
Use the IP address of the Storage Gateway appliance to activate the gateway with your AWS account
Configure local disks on the VM
Set up localized access to the appliance for users, servers, and/or backup software
Configure management and administration for gateway.

### Pre-requisites
On-Premises minimum VM configuration
* Hypervisors: ESXi, Hyper-V, KVM
* 4 vCPUs, 16GB RAM (reserved)
* 80GB storage for VM image and system data
* Additional storage:
  * File gateway (minimum): 150GiB cache
  * Cached Volume Gateway (minimum): 150GiB cache, 150GiB upload buffer
  * Stored Volume Gateway (minimum): Sufficient space for entire iSCSI volume, plus 150GiB+ upload buffer
  * VTL Gateway (minimum): 150GiB cache, 150GiB upload buffer
Stateful firewall ports (all configurations)
* HTTPS (TCP 443) (outbound to internet)
* HTTP (TCP 80) (inbound access from host you will use to configure Storage Gateway)
* DNS (UDP 53) (outbound to internet)
* SSH (TCP 22) (outbound to AWS)
* NTP (UDP 123) (outbound to internet)
* NetBIOS/LDAP (for Active Directory to/from local Domain Controllers)
Additional networking for file gateways (stateful)
* NFS/SMB (inbound from local network)
Additional networking for volume and tape gateways (stateful)
* iSCSI (in and out from local network)

## Setup process
Each of the following links will guide you through the process of initial setup for each gateway type.  Choose from one below, then follow the associated post-deployment steps laid out in the next group of links.
#### [File Gateway Setup](https://docs.aws.amazon.com/storagegateway/latest/userguide/create-file-gateway.html)
#### [Volume Gateway Setup](https://docs.aws.amazon.com/storagegateway/latest/userguide/create-volume-gateway-volume.html)
#### [Tape Gateway Setup](https://docs.aws.amazon.com/storagegateway/latest/userguide/create-tape-gateway.html)

### Post-deployment configuration
Depending upon the type of gateway you have configured, follow the appropriate post-deployment configuration steps to connect users, configure authentication, configure local storage, and create storage entities.
#### [File Gateway Management](https://docs.aws.amazon.com/storagegateway/latest/userguide/managing-gateway-file.html)
#### [Volume Gateway Management](https://docs.aws.amazon.com/storagegateway/latest/userguide/managing-volumes.html)
#### [Tape Gateway Management](https://docs.aws.amazon.com/storagegateway/latest/userguide/managing-gateway-vtl.html)

Finally, for all gateway types, set up CloudWatch against your gateway and associated AWS resources in order to closely monitor utilization and alerting.
#### [Storage Gateway Monitoring](https://docs.aws.amazon.com/storagegateway/latest/userguide/Main_monitoring-gateways-common.html)
