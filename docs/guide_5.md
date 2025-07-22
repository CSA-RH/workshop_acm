# Governance, Risk, and Compliance
## Creating Policies in ACM

At this point, you have completed the overview labs for Cluster Lifecycle and Application Lifecycle capabilities in RHACM. In the Cluster Lifecycle Lab, you learned how RHACM can help manage the lifecycles of your Kubernetes clusters, including both deploying new clusters and importing existing clusters. In that lab, you configured your RHACM instance to manage an OpenShift cluster.

In the Application Lifecycle Lab, you continued exploring RHACM functionality and learned how to deploy and configure an application. You used the cluster that you added in the first module as the target for deploying an application.

Now that you have a cluster and a deployed application, you need to make sure that they do not drift from their original configurations. This kind of drift is a serious problem, because it can happen from beginning and benevolent fixes and changes, as well as malicious activities that you might not notice but can cause significant problems. The solution that RHACM provides for this is the Governance, Risk, and Compliance, or GRC, functionality.
Review GRC Functionality

To begin, it is important to define exactly what GRC is. In RHACM, you build policies that are applied to managed clusters. These policies can do different things, which are described below, but they ultimately serve to govern the configurations of your clusters. This governance over your cluster configurations reduces risk and ensures compliance with standards defined by stakeholders, which can include security teams and operations teams

This table describes the two types of policy controllers available in RHACM along with the remediation mode they support:

| Policy Controller | Purpose | Enforce or Inform |
|----------|----------|----------|
| Configuration | Used to configure any Kubernetes resource across your clusters. Where these resources are created or configured is determined by the namespaces you include (or exclude) in the policy. | Both |
| Certificate | Used to detect certificates that are close to expiring. You can configure the certificate policy controller by updating the minimum duration parameter in your controller policy. When a certificate expires in less than the minimum duration, the policy becomes noncompliant. Certificates are identified from secrets in the included namespaces. | Inform |

This is a complex and emerging/evolving topic, and this course is only providing an overview. Please consult the [GRC product documentation](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.12/html/governance/index) for more details on any of these policy controllers.

### Create a Policy
We’ll go through a simple example, and create a policy in the default namespace.
For this, we’ll need a little setup:

- Navigate to Clusters and access the ClusterSets tab.

    ![Alt text](../images/policy1.png?raw=true "policy1")

- Click on the 3-dots button on the right, and then on Edit namespaces bindings and add the default namespace.

    ![Alt text](../images/policy2.png?raw=true "policy2")

- Navigate to the Governance screen and click create policy. 

    ![Alt text](../images/policy3.png?raw=true "policy3")

- Build a policy with the following information:
    - Name: policy-grc-cert
    - Namespace: default
    - Click NEXT.
        - Policy Templates: Click on Add policy template and select Certificate Management expiration.
    - Click NEXT.
    - On Placement, click on New placement.
        - Select global as the clusterSet.
    - Leave everything else as default and click NEXT.

        ![Alt text](../images/policy4.png?raw=true "policy4")

    - As you can see in the **Review** page, this policy will be applied to every OpenShift cluster in the **global ClusterSet**, and it will look for expired certificates in them. If there’s a certificate that is set to expire in less time than the **minimumDuration** specification, the policy will **inform** a not compliant status.

    - Click SUBMIT.

    - Please note that initially it will complain that there is an issue with the policy but shortly after should go green and get a checkmark.

        ![Alt text](../images/policy5.png?raw=true "policy5")

        ![Alt text](../images/policy6.png?raw=true "policy6")

You can find some examples of policies in the [Collection of policy examples for Open Cluster Management](https://github.com/open-cluster-management/policy-collection).


## Deploying Policies with Policy Generator

This Policy Generator definition will achieve 2 configuration policies:
- **openshift-gitops-installed**: The goal of the first one is to inform if the OpenShift GitOps operator is installed on managed clusters.
- **kubeadmin-removed**: The goal of this second policy is to inform if the kubeadmin user is removed from managed clusters.

Both policies are informative only and will only execute them manually to showcase how to resolve issues.

In order to deliver these policies we will need to leverage the RHACM Application engine and the GitOps Subscription model.

**Pre-requisite** 
- Before we start, on the local-cluster, make sure to add a label if it’s not there yet: `rhdp_usage=development`

    ![Alt text](../images/policy7.png?raw=true "policy7")

- Let’s just do one more step: In the bastion machine, run this command below to make sure our lab user has the right permissions to create policies with policy generator:

    ```sh
    cat << EOF | oc apply -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: open-cluster-management:subscription-admin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: open-cluster-management:subscription-admin
    subjects:
    - apiGroup: rbac.authorization.k8s.io
      kind: User
      name: kube:admin
    - apiGroup: rbac.authorization.k8s.io
      kind: User
      name: system:admin
    - apiGroup: rbac.authorization.k8s.io
      kind: User
      name: admin
    EOF
    ```

- Now, in the ACM UI, navigate to Applications.

- Click Create application, select Subscription. Enter the following information:
- Name: policy-generator
- Namespace: policy-generator
- Under repository types, select the GIT repository
- URL: https://github.com/levenhagen/demo-policygenerator.git
- Branch: main
- Under Select cluster for application deployment, Select global clusterSet and Deploy only to local cluster 
    - Label: name - equals any of - Value: local-cluster
- Verify all the information is correct. Click Create
- It will take a few minutes to deploy the application, Click on the Topology Tab to view and verify that all of the circles are green.

    ![Alt text](../images/policy8.png?raw=true "policy8")

Navigate to the Governance menu.
- Click on the Policies tab.
- Verify that you see 2 policies:
    - kubeadmin-removed
    - openshift-gitops-installed

    ![Alt text](../images/policy9_new.png?raw=true "policy9")

Notice that there is one violation for the policy kubeadmin-removed. This means there’s a kubeadmin password in this cluster and that OpenShift GitOps is installed. If you didn’t stall OpenShift GitOps manually through the Operator Hub, as stated in the previous section, the Governance would report a violation.

While the Policy can be enforced by clicking the three dots at the right of the policy and selecting Enforce, unless there is a Identitity Provider configured, this action would remove the kubeadmin user and leaving us withought console access to the OCP cluster.

https://github.com/CSA-RH/workshop_acm/blob/main/docs/guide_6.md
