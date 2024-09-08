
# S3

Classes:

- Standard
	- No minimum storage duration charge
	- No data retrieval fee
- Standard-IA
	- minimum 30-day storage charge
	- Fast retrieval
	- Has data retrieval fee per GB
- One Zone-IA
	- Same as Standard-IA but stored only in 1 zone (99.95% availability)
	- Cheaper than standard-IA
- S3 Intelligent-Tiering
	- No data retrieval fee
	- For unpredictable access pattern data
	- Automatically moves data based on access pattern to different classes
- Glacier
	- suitable for archiving
	- For rarely accessed data with slow retrieval time
	- highly available 99.99%
	- minimum 90-day storage charge
	- high data retrieval fee
	- Vault Lock enables data integrity and access control (good for regulatory and compliance requirements)
		- For example write once-read many (could be set for any number of years)
	- Retrieval Options:
		- Expedited
			- within 1-5 minutes retrieval for less than 250 MB
			- Requires purchasing provisioned capacity
		- Standard
			- within 3-5 hours retrieval
		- Bulk
			- within 5-12 hours
			- cheapest
- Glacier Deep Archive
	- lowest cost
	- for 7-10 years storage
	- Good for digital preservation
	- 6 months minimum storage duration
	- Retrieval
		- Standard
			- Within 12 hours
		- Bulk
			- Within 48 hours
- 

##### S3 cross account copying with encryption

General flow is like this:
1. Create a IAM role in source account with permission to the destination bucket
2. Attach it to a user
3. Change object ownership of the destination account to Bucket Owner Preffered
4. Define a bucket policy in destination account that grant same access to the previous role
5. Set `--acl bucket-owner-full-control`  while copying to the destination

You are gonna need some extra KMS permissions if S3 is KMS encrypted

Check out this article:
https://medium.com/codebyte/cross-account-s3-object-copying-with-kms-encrypted-buckets-5ebabef8aa03

# Binding

Bind Mounts: Should be used only for temporary storage on EC2 or Fargate.

Docker volumes:
- Are only available for ECS using EC2 and are not available with Fargate.
- They could be used with volume drivers (plugins) to be integrated with EBS

EFS: Could be used with ECS

### Encryption
Server-Side Encryption in S3 is always AES256, whether you are using SSE-S3 or SSE-KMS.

In both cases, S3 uses a key to _transparently_ encrypt the object for storage and decrypt the object on request. The user accessing the object does not see the encrypted object in either case.

With SSE-S3, S3 owns and controls the keys, so permission to upload or download includes implicit permission for S3 to access the keys that it needs in order to access the object.

The level of encryption is the same whether you use SSE-S3 or SSE-KMS, but SSE-KMS imposes more stringent security constraints on accessing the objects, including mandatory use of HTTPS and Signature Version 4.


# EBS

- A gp2 and gp3 volume can range in size from 1 GiB to 16 TiB.
- EBS Volumes are AZ-locked, use a snapshot to move them to another region or AZ.

##### Encryption
- New volumes are not encrypted by default
- There's an account level setting (per region) in EC2 console to enable encryption for new volumes
- You can't change the encryption key, but you can get a snapshot, create volume from it and use a new key
- You can copy EBS snapshot to another account and use a KMS policy on target account to decrypt it from original key and encrypt it back with a new key in the target account.


### EFS
- You can't encrypt an unencrypted EFS file system
- You can create a new encrypted EFS and migrate the data using AWS FileSync


### FSX
There are 4 types available:
- NetApp ONTAP
	- Recommended for NetApp ONTAP or other NAS appliances
- OpenZFS
	- Recommended for ZFS or other linux based file servers
- Windows File Server
	- Recommended for MS Windows Server
- Lustre
	- Recommended for Lustre or other parallel file systems

They all provide latency of < 1ms and a very high throughput. Lustre provides the highest throughput and IOPS but is only available for Linux clients.

They are all available for EC2, ECS and EKS

Lustre provides lower availability than others due to the fact that it's single AZ only

##### FSX Lustre
Provides 2 deployment options: Scratch and persistent

Scratch: designed for temporary storage and short term processing. Data will not persist if file system fails.

Persistent: designed for long term storage and workloads.Highly available and data is automatically replicated within the signle AZ. 

It could be linked with S3 buckets, this way it can automatically sync data both ways as data are changed.

How to access it using EC2?
	Use open source Lustre client to mount the FSx and use it like normal file system
How to access in EKS?
	Use persistent storage volume backed by FSx using FSx for Lustre CSI driver

Can you use it for machine learning?
	Yes,  you can use it as input for SageMaker training jobs

Can you use it on-prem?
	Yes you can use Amazon Direct Connect or VPN

Does it support encryption?
	Yes, it always encrypt your data using KMS

# Storage Gateway

Storage Gateway supports four key hybrid cloud use cases – (1) move backups and archives to the cloud, (2) reduce on-premises storage with cloud-backed file shares, (3) provide on-premises applications low-latency access to data stored in AWS, and (4) data lake access for pre and post processing workflows.

### S3 File Gateway

With S3 File Gateway, your configured S3 buckets will be available as Network File System (NFS) mount points or Server Message Block (SMB) file shares.

### FSx File Gateway
Amazon FSx File Gateway optimizes on-premises access to Windows file shares on Amazon FSx, making it easy for users to access FSx for Windows File Server data with low latency and conserving shared bandwidth

### Tape Gateway
Tape Gateway is a cloud-based Virtual Tape Library (VTL). It presents your backup application with a VTL interface, consisting of a media changer and tape drives. You can create virtual tapes in your virtual tape library using the AWS Management Console. Your backup application can read data from or write data to virtual tapes by mounting them to virtual tape drives using the virtual media changer

### Volume Gateway
Volume Gateway provides an iSCSI target, which enables you to create block storage volumes and mount them as iSCSI devices from your on-premises or EC2 application servers. The Volume Gateway runs in either a cached or stored mode.

- In the cached mode, your primary data is written to S3, while retaining your frequently accessed data locally in a cache for low-latency access.
- In the stored mode, your primary data is stored locally and your entire dataset is available for low-latency access while asynchronously backed up to AWS.

