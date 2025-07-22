# Lifecycle OpenShift Clusters
At a high level Cluster Lifecycle management is about creating, upgrading, and destroying and importing clusters in a multi cloud environment.

## Create Credential 
In order to create a new OpenShift cluster in the AWS cloud we will need these keys to create a Provider Connection. On the left bar, select Credentials and then select Add Credential.

**NOTE: The credential has been already created and setup on your behalf, so you dont have to take any action.**

For your reference the following is the required configuration:

- Open the OpenShift Hub cluster console 
- in the left tab navidate to Credentials.
- press  button *Add Credential*

![Alt text](../images/credentials.png?raw=true "credentials")

You will need to provide the connection details:
- Credential Type: Choose Amazon Web Services and then, Amazon Web Services again
- Credential Name:  aws
- Namespace: open-cluster-management
- Base DNS Domain:  Domain of the cluster
- Access Key ID:  AWS Key Id
- Secret Access Key ID: AWS Access Key
- Leave empty the Proxies or additional CAs
- Red Hat OpenShift pull secret. 
- SSH private and public keys(optional). Note that the key pair needs to be in the PEM format.
- Review and create.

Please refer to [Creating a cloud connection for Amazon Web Services](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.8/html/clusters/cluster_mce_overview#creating-a-credential-for-amazon-web-services) and this [link](https://docs.openshift.com/container-platform/4.13/installing/installing_aws/installing-aws-default.html#ssh-agent-using_installing-aws-default) for more information.

![Alt text](../images/credentials1.png?raw=true "credentials1")

## Create a new OpenShift cluster in AWS

- From the menu select Infrastructure → Clusters
- Click Create Cluster
- Select Amazon Web services
- Select Standalone
- Select the Infrastructure provider credential called AWS
    - Name cluster: cluster1
    - Leave the Cluster set empty
    - Select a Release Image, choose whatever OCP version is newest
    - Add a label of `environment=prod`. 
    - Click Next
- Change the region to: `us-west-1 or us-west-2`

    ![Alt text](../images/region.png?raw=true "region")

    `NOTE: The deployment might fail, if so please retry at different availability zones, as the amount of elastic IP addresses are limited.`

- Leave everything else as is(default) and then click NEXT on the other screens or select 7 - Review and create on the menu and then click CREATE

    ![Alt text](../images/clustercreate1.png?raw=true "clustercreate1")

In about 45 minutes this new cluster will be ready to go!  

![Alt text](../images/clustercreate11.png?raw=true "clustercreate11")

## Creating a Single Node Cluster (SNO) in AWS

In this exercise we will show you how to create a single node cluster:

- From the menu select Infrastructure → Clusters
- Click Create Cluster
- Select Amazon Web services
- Select Standalone
- Select the Infrastructure provider credential called AWS
    - Name cluster: cluster2-sno
    - Leave the Cluster set empty
    - Select a Release Image, choose whatever OCP version is newest
    - Add a label of `environment=qa`. 
    - Click Next
- Change the region to: `us-west-1 or us-west-2`

    `NOTE: The deployment might fail, if so please retry at different availability zones, as the amount of elastic IP addresses are limited.`

- Click on Worker Pool 1 and change the Node count to 0

    ![Alt text](../images/clustercreate21.png?raw=true "clustercreate21")

    - Proceed with everything else as is and while at final step, **before** proceeding turn **YAML: ON**.

        ![Alt text](../images/clustercreate22.png?raw=true "clustercreate22")

    - Click on “install-config” in the YAML window pane and change the master replica number to 1 (will likely be 3).  Double check that the worker replica is 0.

        ![Alt text](../images/clustercreate23.png?raw=true "clustercreate23")

    - Click on CREATE.

## Continue to the next chapter
https://github.com/CSA-RH/workshop_acm/blob/main/docs/guide_4.md
