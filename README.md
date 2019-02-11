# Istio Routing Mission for Node.js

[![Greenkeeper badge](https://badges.greenkeeper.io/nodeshift-starters/nodejs-istio-routing.svg)](https://greenkeeper.io/)

## Purpose

Showcase Routing in Istio with a Node.js application

## Prerequisites

* Openshift 3.10 cluster with Istio. For local development, download the latest release from [Maistra](https://github.com/Maistra/origin/releases) and run:

```
# Set oc to be the Maistra one
oc cluster up --enable="*,istio"
oc login -u system:admin
oc apply -f https://raw.githubusercontent.com/Maistra/openshift-ansible/maistra-0.6/istio/cr-minimal.yaml -n istio-operator
oc get pods -n istio-system -w
```
Wait until the `openshift-ansible-istio-installer-job-xxxx` job has completed. It can take several minutes. The OpenShift console is available on https://127.0.0.1:8443.

* Create a new project/namespace on the cluster. This is where your application will be deployed.

```
oc login -u system:admin
oc adm policy add-cluster-role-to-user admin developer --as=system:admin
oc adm policy add-scc-to-user anyuid -z default -n myproject
oc adm policy add-scc-to-user privileged -z default -n myproject
oc login -u developer -p developer
oc new-project <whatever valid project name you want> # not required
```

### Build and Deploy the Application

#### Launcher Flow Setup

If the Booster is installed through the Launcher and the Continuous Delivery flow, no additional steps are necessary.

Skip to the _Use Cases_ section.

#### With Source to Image build (S2I)

### Prepare the Namespace

Create a new namespace/project:
```
oc new-project <whatever valid project name you want>
```

### Build and Deploy the Application

#### With Source to Image build (S2I)

Run the following commands to apply and execute the OpenShift templates that will configure and deploy the applications:
```bash
find . | grep openshiftio | grep application | xargs -n 1 oc apply -f

oc new-app --template=nodejs-istio-routing-client-service -p SOURCE_REPOSITORY_URL=https://github.com/nodeshift-starters/nodejs-istio-routing -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=routing-client
oc new-app --template=nodejs-istio-routing-service-a-service -p SOURCE_REPOSITORY_URL=https://github.com/nodeshift-starters/nodejs-istio-routing -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=routing-service-a
oc new-app --template=nodejs-istio-routing-service-b-service -p SOURCE_REPOSITORY_URL=https://github.com/nodeshift-starters/nodejs-istio-routing -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=routing-service-b
```



Any steps issuing `oc` commands require the user to have run `oc login` first and switched to the appropriate project with `oc project <project name>`.

### Default Service load balancing

1. Create a Gateway and Virtual Service in Istio so that we can access the service within the Mesh:
    ```
    oc apply -f istio-config/gateway.yaml
    ```
2. Retrieve the URL for the Istio Ingress Gateway route, with the below command, and open it in a web browser.
    ```
    echo http://$(oc get route istio-ingressgateway -o jsonpath='{.spec.host}{"\n"}' -n istio-system)/nodejs-istio-routing
    ```
3. The user will be presented with the web page of the Booster
4. Click the "Invoke" button. You should see a message in the result box indicating which service instance was called.
5. Click "Invoke" several more times.
Notice that there is an even 50% split between service a and b.

### Transfer load between services

1. Modify the load balancing such that all requests go to service a:
    ```
    oc apply -f istio-config/load-balancing-rule.yaml
    ```
2. Clicking on "Invoke" in the UI you will see that all requests are now being sent to service a.
