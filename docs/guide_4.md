# Application Lifecycle

In the previous lab, you explored the Cluster Lifecycle functionality in RHACM. This allowed you to register an OpenShift cluster to RHACM, which you can now use to deploy applications.

Application Lifecycle functionality in RHACM provides the processes that are used to manage application resources on your managed clusters. This allows you to define a single or multi-cluster application using Kubernetes specifications, but with additional automation of the deployment and lifecycle management of resources to individual clusters. An application designed to run on a single cluster is straightforward and something you ought to be familiar with from working with OpenShift fundamentals. A multi-cluster application allows you to orchestrate the deployment of these same resources to multiple clusters, based on a set of rules you define for which clusters run the application components.

## Deploying Argo CD on the Hub Cluster

**Prerequisites**:  

- On the local cluster make sure add a label if it’s not there yet:  `rhdp_usage=development`

    ![Alt text](../images/application1.png?raw=true "application1")

- On the new cluster you provisioned via ACM double check you added the label: `environment=prod`

    ![Alt text](../images/application2.png?raw=true "application2")

**Deploy ArgoCD Operator in HUB Cluster**

- In RHACM, navigate to the MultiCluster Switcher and select Local-Cluster  to see the OCP UI.

    ![Alt text](../images/application3.png?raw=true "application3")

- Navigate to Operators → OperatorHub and search or OpenShift GItops 

    ![Alt text](../images/application4.png?raw=true "application4")

- Click on the Operator and Click Install, leave all the defaults and click Install, you might see errors those are normal, just wait for the installation to complete, might take up to 7 min to deploy.

    ![Alt text](../images/application5.png?raw=true "application5")

- Once the installation is complete, you might need to refresh the web console.

    ![Alt text](../images/application6.png?raw=true "application6")

- At the very top, click on Refresh web console. Then, you’re now able to see the OpenShift GitOps operator route by click at the button on the top-right corner of the web console.

    ![Alt text](../images/application7.png?raw=true "application7")

- Go to the Bastion host again, login with the credentials provided, for your user and run the following command:

    ```sh
    cat << EOF | oc apply -f -
    apiVersion: user.openshift.io/v1
    kind: Group
    metadata:
      name: cluster-admins
    users:
    - admin
    EOF
    ```

    This step is adding the right ArgoCD permissions to the lab user so we can later log in.

- Log in via OpenShift button using your Openshift admin credential, you might need to allow permissions, click Allow. Once logged in ArgoCD should look like this:

    ![Alt text](../images/application8.png?raw=true "application8")

- Navigate back to RHACM by clicking on the cluster selector to All Clusters then click on Applications. If ArgoCD was deployed successfully you should see under create application a Argo CD Application set for Pull and one for Push Model 

    ![Alt text](../images/application9.png?raw=true "application9")

## Deploying An Application Using Argo CD

Argo CD is a CNCF-graduated project that utilizes a GitOps approach for continuously managing and deploying applications on Kubernetes clusters. On the other hand RHACM, which is based off of the CNCF Sandbox project Open Cluster Management, focuses on managing a fleet of Kubernetes clusters at scale.

By utilizing RHACM users can now enable the optional Argo CD pull model architecture which offers flexibility that may be better suited for certain scenarios. One of the main use cases for the optional pull model is to address network scenarios where the centralized cluster is unable to reach out to remote clusters, while the remote clusters can communicate with the centralized cluster. In such scenarios, the push model would not be easily feasible.

Argo CD currently utilizes a push model architecture where the workload is pushed from a centralized cluster to remote clusters, requiring a connection from the centralized cluster to the remote destinations.

In the pull model, the Argo CD Application CR is distributed from the centralized cluster to the remote clusters. Each remote cluster independently reconciles and deploys the application using the received CR. Subsequently, the application status is reported back from the remote clusters to the centralized cluster, resulting in a user experience (UX) that is similar to the push model.

Another advantage of the pull model is decentralized control, where each cluster has its own copy of the configuration and is responsible for pulling updates independently. The hub-managed architecture using Argo CD and the pull model can reduce the need for a centralized system to manage the configurations of all target clusters, making the system more scalable and easier to manage. However, note that the hub cluster itself still represents a potential single point of failure, which you should address through redundancy or other means.
Additionally, the pull model provides allows clusters to pull updates on their own schedule.

For this exercise we will use the Push Model.

- In RHACM, navigate to Applications 
- and click Create application → Argo CD ApplicationSet Push Model 
- Name: rocketchat
- Argo Server: Click add ArgoServer 

    ![Alt text](../images/application10.png?raw=true "application10")

- Name: argodemo 
- and select the openshift-gitops Namespace. 
- Click ADD

    ![Alt text](../images/application11.png?raw=true "application11")

- Once added should show up now on the drop down as openshift-gitops. Click NEXT
    - Click on the Git Repository and enter the following:
    - URL: https://github.com/levenhagen/rocketchat-acm.git
    - Revision: main
    - Path: rocketchat
    - Remote namespace: default

    ![Alt text](../images/application12.png?raw=true "application12")

- Click NEXT till you are at 4 Placement
- Select default cluster set 
- Expand the Label expressions and set the following:
    - Label: environment
    - Operator: equals any of
    - Values: prod
- Click Next to review and Submit

    ![Alt text](../images/application13.png?raw=true "application13")

- Click on the Topology tab and watch as the application is being deployed. The Application deployment is completed once all of the Topology view is Green. Please note that this might take about 4 minutes to fully complete

    ![Alt text](../images/application14.png?raw=true "application14")

- Once complete, click on the ApplicationSet and Launch to the Argo Editor

    ![Alt text](../images/application15.png?raw=true "application15")

- Once ArgoCD opens click Log in VIA OpenShift once logged in you can see your Application fully deployed in Argo.

    ![Alt text](../images/application16.png?raw=true "application16")

Feel free to explore around ArgoCD
You have now completed the overview of the Application Lifecycle functionality in RHACM.

You successfully deployed an application to a target cluster using RHACM. This approach leveraged a Git repository which housed all of the manifests that defined your application. RHACM was able to take those manifests and use them as deployables, which were then deployed to the target clusters.

## Continue to the next chapter
https://github.com/CSA-RH/workshop_acm/blob/main/docs/guide_5.md
