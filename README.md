# Host your own Hadoop Cluster on Amazon Web Services #

**(AKA the problem that we solved: cluster got corrupted and we need to re-deploy a new one, also we couldn’t restart the cluster for the fear of corrupt the cluster)**

We are going to deploy a machine with Ambari Server exposed to the internet and as a single access point to the hadoop cluster made of 4 node (or more)

1. Prerequisite
  1.1. Your Linux distro package management.
  1.2. An understanding of what Ambari and Hadoop are
  1.3. AWS EC2 AMI.
  1.4. AWS CloudFormation.
  1.5. AWS Route53 and your own domain which will include  a HostedZoneId
  1.6. Mount externals drive on linux.
2. Create Ambari Server AMI.
  2.1. [Download Ambari Server](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.4.0/bk_ambari-installation/content/download_the_ambari_repo.html "Download the Ambari Repository")
  2.2. [Install Ambari Server](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.4.0/bk_ambari-installation/content/install-ambari-server.html "Install the Ambari Server")
  2.3. [Setup Ambari](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.4.0/bk_ambari-installation/content/set_up_the_ambari_server.html "Set Up the Ambari Server")
3. Create Ambari Agent AMI
  3.1. [Download and install Ambari Agent](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.4.0/administering-ambari/content/amb_installing_ambari_agents_manually.html "Installing Ambari agents manually") **NOTE:** You don’t need to configure it, we’ll do that later on
4. Deploy CloudFormation template
  4.1. Make sure you are in the region that you want to deploy the cluster
  4.2. Choose a different name than hadoop, since we are using this name as tag for several resources to identifying them as part of this template, please choose something more meaningful to what your ultimate goal is.
  4.3. Select an EC2 instance type for the hadoop server machine and one for the hadoop cluster machines [more info](https://aws.amazon.com/ec2/instance-types/ "Amazon EC2 Instance Types")
  4.4. Check for log in the EC2 console
5. Mount external drives

    ```sh
    lsblk
    namenode="nvme1n1"
    datanode="nvme0n1"
    fdisk /dev/${namenode}

    fdisk /dev/${datanode}

    mkfs.ext4 /dev/${namenode}p1
    mkfs.ext4 /dev/${datanode}p1
    mount /dev/${namenode}p1 /opt/external-drives/namenode
    mount /dev/${datanode}p1 /opt/external-drives/datanode1
    echo "/dev/${namenode}p1 /opt/external-drives/namenode ext4 defaults 0 0" >>/etc/fstab
    echo "/dev/${datanode}p1 /opt/external-drives/datanode1 ext4 defaults 0 0" >>/etc/fstab
    cat /etc/fstab

    mount | grep /opt/external-drives/
    ```

6. Deploy your own Ambari Blueprint **(Optional)**

    ```sh
    curl -H "X-Requested-By: ambari" -X POST -u admin:admin -d "@./blueprint.json" "http://127.0.0.1:8080/api/v1/blueprints/${ClusterName}_blueprint"

    curl -H "X-Requested-By: ambari" -X POST -u admin:admin -d "@./ambari.map2.json" "http://127.0.0.1:8080/api/v1/clusters/${ClusterName}"
    ```

7. What's Next
  7.1. Mount more external drives
  7.2. Make ambari server to be accessible only through a VPN or VPC
  7.3. Remove access to internet from Hadoop cluster and work with private repositories for software updates
  7.4. Add more Hadoop Workers
  7.5. Implement AWS::AutoScaling::LaunchConfiguration and AWS::AutoScaling::AutoScalingGroup instead of single EC2 instances
