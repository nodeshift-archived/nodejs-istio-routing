# Istio Routing Mission for Node.js

## Purpose

Showcase Routing in Istio with a Node.js application

## Prerequisites

* Docker installed and running
* OpenShift and Istio environment up and running (See https://github.com/openshift-istio/istio-docs/blob/master/user-journey.adoc for details)

## Launcher Flow Setup

If the Booster is installed through the Launcher and the Continuous Delivery flow, no additional steps are necessary.

Skip to the _Use Cases_ section.

## Local Source to Image Build (S2I)

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

oc new-app --template=nodejs-istio-routing-client-service -p SOURCE_REPOSITORY_URL=https://github.com/bucharest-gold/nodejs-istio-routing -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=routing-client
oc new-app --template=nodejs-istio-routing-service-a-service -p SOURCE_REPOSITORY_URL=https://github.com/bucharest-gold/nodejs-istio-routing -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=routing-service-a
oc new-app --template=nodejs-istio-routing-service-b-service -p SOURCE_REPOSITORY_URL=https://github.com/bucharest-gold/nodejs-istio-routing -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=routing-service-b
```

## Use Cases

### Default Service load balancing

1. Retrieve the URL for the Istio Ingress route, with the below command, and open it in a web browser.
    ```
    echo http://$(oc get route istio-ingress -o jsonpath='{.spec.host}{"\n"}' -n istio-system)/
    ```
2. The user will be presented with the web page of the Booster
3. Click the "Invoke" button. You should see a message in the result box indicating which service instance was called.
4. Click "Invoke" several more times.
Notice that there is an even 50% split between service a and b.

### Transfer load between services

1. Modify the load balancing such that all requests go to service a:
    ```
    oc apply -f rules/load-balancing-rule.yaml
    ```
2. Clicking on "Invoke" in the UI you will see that all requests are now being sent to service a.
