# Deploying Choreo Connect on Kubernetes With WSO2 API Manager as a Control Plane - Helm Artifacts

Let's deploy an API on Choreo Connect, which running on Kubernetes, with WSO2 API Manager as the Control Plane. You can select one of the options to install Choreo Connect Helm Chart in the middle of the documentation.

- [Option 1: Install Chart from WSO2 Helm Chart Repository](#option-1-install-chart-from-wso2-helm-chart-repository)
- [Option 2: Install Chart from Source](#option-2-install-chart-from-source)

## Before you begin

-   WSO2 product Docker images used for the Kubernetes deployment.

    WSO2 product Docker images available at [DockerHub](https://hub.docker.com/u/wso2/) (General Availability (GA)
    versions of WSO2 products with no [WSO2 Updates](https://wso2.com/updates).

    For a production grade deployment of the desired WSO2 product-version, it is highly recommended to use the relevant
    Docker image which packages WSO2 Updates, available at [WSO2 Private Docker Registry](https://docker.wso2.com/). In order
    to use these images, you need an active [WSO2 Subscription](https://wso2.com/subscription).
    
-   Install [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), [Helm](https://helm.sh/docs/intro/install/), and [Kubernetes client](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
    
-   Set up a [Kubernetes cluster](https://kubernetes.io/docs/setup/#learning-environment).
    - Minimum CPU : 3vCPU
    - Minimum Memory : 2GB
    
-   Install [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/). 

    !!! Note
        Helm resources for WSO2 product deployment patterns are compatible with the [`nginx-0.30.0`](https://github.com/kubernetes/ingress-nginx/releases/tag/nginx-0.30.0) release.


## Deploy API Manager as Control Plane

Following are some sample instructions to deploy API Manager for **quick start purpose**.

!!! Tip
    Please follow the document [Deploying API-M on Kubernetes using Helm Resources]({{base_path}}/install-and-setup/install/deploying-api-manager-with-kubernetes-or-openshift-resources/) which describes deploying API Manager. The following guide describes deploying API Manager for quick start purpose.

1.  Execute the command that is relevant to your Helm version.

    !!! Tip
        If you do not have sufficient resources you can adjust them setting following values when installing the chart.
        ```bash
        --set wso2.deployment.am.resources.requests.memory=2Gi \
        --set wso2.deployment.am.resources.requests.cpu=1000m \
        --set wso2.deployment.am.resources.limits.memory=2Gi \
        --set wso2.deployment.am.resources.limits.cpu=1000m
        ```

    -   Using **Helm v2**

        ```bash
        helm install --name apim-as-cp wso2/am-single-node --version 4.0.0-1 --namespace apim \
            --set wso2.deployment.am.ingress.gateway.hostname=gw.wso2.com \
            --set-file wso2.deployment.am.config."deployment\.toml"=https://raw.githubusercontent.com/wso2/kubernetes-microgateway/v1.0.0.1/resources/controlplane-deployment.toml
        ```

    -   Using **Helm v3**

        ```bash
        helm install apim-as-cp wso2/am-single-node --version 4.0.0-1 --namespace apim --create-namespace \
            --set wso2.deployment.am.ingress.gateway.hostname=gw.wso2.com \
            --set-file wso2.deployment.am.config."deployment\.toml"=https://raw.githubusercontent.com/wso2/kubernetes-microgateway/v1.0.0.1/resources/controlplane-deployment.toml
        ```

2.  Choreo Connect is used as the gateway of API Manager, hence we need to delete the gateway Ingress resource of gateway component of the WSO2 API Manager.

    ```bash
    kubectl delete ing -n apim wso2am-single-node-am-gateway-ingress
    ```

## Option 1: Install Chart from WSO2 Helm Chart Repository

### Step 1 - Add the WSO2 Helm chart repository

Add the [WSO2 Helm chart repository](https://hub.helm.sh/charts/wso2) by executing the following command.

```bash
helm repo add wso2 https://helm.wso2.com && helm repo update
```

### Step 2 - Install Chart

Execute the following command to install the Helm Cart by selecting the helm version you installed.

-   Using **Helm v2**

    ```bash tab='Format'
    helm install --name <RELEASE_NAME> wso2/choreo-connect --version 1.0.0-1 --namespace <NAMESPACE> \
        --set wso2.deployment.mode=APIM_AS_CP \
        --set wso2.apim.controlPlane.hostName=am.wso2.com \
        --set wso2.apim.controlPlane.serviceName=wso2am-single-node-am-service.apim
    ```

    ```bash tab='Sample'
    helm install --name my-release wso2/choreo-connect --version 1.0.0-1 --namespace cc \
        --set wso2.deployment.mode=APIM_AS_CP \
        --set wso2.apim.controlPlane.hostName=am.wso2.com \
        --set wso2.apim.controlPlane.serviceName=wso2am-single-node-am-service.apim
    ```

-   Using **Helm v3**

    ```bash tab='Format'
    helm install <RELEASE_NAME> wso2/choreo-connect --version 1.0.0-1 --namespace <NAMESPACE> --create-namespace \
        --set wso2.deployment.mode=APIM_AS_CP \
        --set wso2.apim.controlPlane.hostName=am.wso2.com \
        --set wso2.apim.controlPlane.serviceName=wso2am-single-node-am-service.apim
    ```

    ```bash tab='Sample'
    helm install my-release wso2/choreo-connect --version 1.0.0-1 --namespace cc --create-namespace \
        --set wso2.deployment.mode=APIM_AS_CP \
        --set wso2.apim.controlPlane.hostName=am.wso2.com \
        --set wso2.apim.controlPlane.serviceName=wso2am-single-node-am-service.apim
    ```

The above steps will deploy the Choreo Connect using WSO2 product Docker images available at DockerHub.

If you are using WSO2 product Docker images available from WSO2 Private Docker Registry, please provide your **WSO2 Subscription** credentials via input values (using `--set` argument).

Please see the following example.

```bash tab='Format'
helm install --name <RELEASE_NAME> wso2/choreo-connect --version 1.0.0-1 --namespace <NAMESPACE> \
  --set wso2.subscription.username=<SUBSCRIPTION_USERNAME> \
  --set wso2.subscription.password=<SUBSCRIPTION_PASSWORD>
```

<br/>
Skip the following section "Option 2: Install Chart from Source" since you have already installed Choreo Connect and jump to [Access the Choreo Connect deployment](#access-the-choreo-connect-deployment) for deploying APIs.

## Option 2: Install Chart from Source

### Step 1 - Get the Helm resources

Check out the Helm Resources for the Choreo Connect Git repository.

1.  Open a terminal and navigate to the location where you want to save the local copy.
2.  Clone the Choreo Connect Git repository with Helm resources:

    ```bash
    git clone https://github.com/wso2/kubernetes-microgateway.git
    git checkout tags/v1.0.0.1
    ```

This creates a local copy of [wso2/kubernetes-microgateway](https://github.com/wso2/kubernetes-microgateway), which includes all the Helm Resources for Choreo Connect.

Let's refer to the root folder of the local copy as `<KUBERNETES_HOME>`.

### Step 2 - Update the deployment configurations

Follow the steps given below to configure how your Choreo Connect deployment should be set up.

1.  Open the `values.yaml` file in the `<KUBERNETES_HOME>/helm/choreo-connect` directory of your local copy.

    !!! Info
        Before you do any changes, go through the [default configurations](https://github.com/wso2/kubernetes-microgateway/tree/v1.0.0.1/helm/choreo-connect) in this file.

2.  Use the following guidelines to update the deployment configurations:

    -   **Updating the WSO2 subscription details**

        You can update the username and password in the following section. If you don't have an active WSO2 subscription, leave these parameters empty.

        ```yaml
        wso2:
            subscription:
                username: "<username>"
                password: "<password>"
        ```

        Alternatively, you can skip this step and pass your subscription details at the time of deploying (see the next step for details).

    -   **Updating Choreo Connect Deployment Mode**

        ```yaml
        wso2:
            deployment:
                mode: "APIM_AS_CP"
        ```

    -   **Updating Choreo Connect control plane configurations**

        ```yaml
        wso2:
            apim:
                controlPlane:
                    hostName: "<controlplane host name>"
                    serviceName: "<controlplane kubernetes service name>"
        ```

    -   You can update [other configurations](https://github.com/wso2/kubernetes-microgateway/tree/v1.0.0.1/helm/choreo-connect/README.md) as required.

3.  Save the `values.yaml` file.

### Step 3 - Deploy Choreo Connect

Once you have set up your Helm resources locally, follow the instructions given below to set up the deployment.

1.  Open a terminal and navigate to the `<KUBERNETES_HOME>/helm/choreo-connect` directory.
2.  Execute the command that is relevant to your Helm version.

    !!! Tip
        Be sure to replace `NAMESPACE` with the Kubernetes namespace in which your resources are deployed.

    -   Using **Helm v2**

        ```bash
        helm install --name <RELEASE_NAME> wso2/choreo-connect --version 1.0.0-1 --namespace <NAMESPACE>
        ```

    -   Using **Helm v3**

        ```bash
        helm install <RELEASE_NAME> wso2/choreo-connect --version 1.0.0-1 --namespace <NAMESPACE> --create-namespace
        ```

#### Update configurations during deployment

If required, you can set any of the deployment configurations at the time of running the deployment (instead of
specifying them in the `values.yaml` file). See the examples given below.

-   Setting the subscription username and password.

    ```bash
    --set wso2.subscription.username=<SUBSCRIPTION_USERNAME>
    --set wso2.subscription.username=<SUBSCRIPTION_USERNAME>
    ```

-   Setting the Choreo Connect deployment mode. 

    ```bash
    --set wso2.deployment.mode=APIM_AS_CP
    ```

-   Use the Choreo Connect control plane configurations.

    ```bash
    --set wso2.apim.controlPlane.hostName=am.wso2.com
    --set wso2.apim.controlPlane.serviceName=wso2am-single-node-am-service.apim
    ```

## Access the Choreo Connect deployment

Follow the steps given below.

1.  Get the external IP (`EXTERNAL-IP`) of the Ingress resources by listing down the Kubernetes Ingresses.

    -   **API Manager - Control Plane**

        ```bash
        kubectl get ing -n apim
        ```

        Example:

        ```bash
        NAME                                       HOSTS                      ADDRESS        PORTS     AGE
        wso2am-pattern-3-am-cp-ingress             am.wso2.com                <EXTERNAL-IP>  80, 443   3m
        ```

    -   **Choreo Connect**

        ```bash
        kubectl get ing -n <NAMESPACE>
        ```

        Example:

        ```bash
        NAME                                       HOSTS                      ADDRESS        PORTS     AGE
        <RELEASE_NAME>-choreo-connect-router       gw.wso2.com                <EXTERNAL-IP>  80, 443   3m
        ```

2.  Add the above hosts in the `/etc/hosts` file as follows:

    ```bash
    <EXTERNAL-IP>   am.wso2.com
    <EXTERNAL-IP>   gw.wso2.com
    ```

## Deploy Sample API from API Manager

 - Publisher Portal:  [https://am.wso2.com/publisher/](https://am.wso2.com/publisher/)
 - Developer Portal:  [https://am.wso2.com/devportal/](https://am.wso2.com/devportal/)

Follow the instructions in [create and publish an API via API Manager]({{base_path}}/deploy-and-publish/deploy-on-gateway/choreo-connect/getting-started/quick-start-guide-docker-with-apim/#step-3-create-and-publish-an-api-from-api-manager).
