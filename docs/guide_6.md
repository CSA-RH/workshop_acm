# Observability

View system alerts, critical application metrics, and overall system health. Search, identify, and resolve issues that are impacting distributed workloads using an operational dashboard designed for Site Reliability Engineers (SREs). This is done via the integration of Grafana. Let's walk through the steps to integrate Grafana with ACM.

- You will need your AWS Keys.
- You will also need to create an AWS S3 bucket.
- SSH information to your bastion host.

## Create the S3 bucket
- Login to the bastion host.
- Run the following command to login to AWS: `aws configure`  and enter your AWS keys when prompted.  Default region: **us-east-2**
- Then run the following command to create the S3 bucket: `aws s3 mb s3://grafana-$GUID`
- Take note of the bucket name.


## Integrate Grafana into ACM
- Login to the bastion host host.
- Create a namespace to deploy the objects:

    ```sh
    oc create namespace open-cluster-management-observability
    ```

- Copy the pull secret into this new namespace by running the following TWO commands:

    ```sh
    DOCKER_CONFIG_JSON=`oc extract secret/pull-secret -n openshift-config --to=-`
    ```

    ```sh
    oc create secret generic multiclusterhub-operator-pull-secret -n open-cluster-management-observability --from-literal=.dockerconfigjson="$DOCKER_CONFIG_JSON" --type=kubernetes.io/dockerconfigjson
    ```

- In your current folder create a file called thanos-object-storage.yaml  and add the following text in the file. 
    - Please be sure to update the env variables with your S3 bucket name and AWS Keys:

    ```sh
    export AWS_ACCESS_KEY_ID=<YOUR_AWS_ACCESS_KEY_ID>
    export AWS_SECRET_ACCESS_KEY=<YOUR_AWS_SECRET_ACCESS_KEY>
    export YOUR_S3_BUCKET=grafana-${GUID}
    ```

    ```sh
    cat << EOF > thanos-object-storage.yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: thanos-object-storage
      namespace: open-cluster-management-observability
    type: Opaque
    stringData:
      thanos.yaml: |
        type: s3
        config:
          bucket: ${YOUR_S3_BUCKET}
          endpoint: s3.amazonaws.com
          insecure: false
          access_key: ${AWS_ACCESS_KEY_ID}
          secret_key: ${AWS_SECRET_ACCESS_KEY}
    EOF
    ```

- Create a secret for your object storage by running the following command:

    ```sh
    oc create -f thanos-object-storage.yaml
    ```

- Create the MultiClusterObservability custom resource for your managed clusters.  To do this create a  **YAML file named multiclusterobservability_cr.yaml**

    ```sh
    cat << EOF | oc apply -f -
    kind: MultiClusterObservability
    apiVersion: observability.open-cluster-management.io/v1beta2
    metadata:
      name: observability
    spec:
      observabilityAddonSpec: {}
      storageConfig:
        metricObjectStorage:
          key: thanos.yaml
          name: thanos-object-storage
    EOF
    ```
    - NOTE: There are multiple other optional fields to customize this resource. The following [reference](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.12/html/apis/apis#rhacm-docs_apis_multiclusterobservability_jsonmulticlusterobservability) have more details about this object.

- Log in to the ACM console, and navigate to the Clusters Screen
- Click on the Grafana link in the top right to view the metrics from the managed clusters. **Please note**: it will take a few minutes for the metrics to become visible on the dashboard.

![Alt text](../images/observability1.png?raw=true "observability1")

- You should shortly see something like the below

![Alt text](../images/observability2.png?raw=true "observability2")

Would you like to do more with Grafana? Please visit the following [link](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.12/html/observability/index) for more details.
