**********************
Platform Installation
**********************

Prerequisites
=============

 * Kubernetes 1.14+ with RBAC enabled
 * Helm 2.14+

Install core components
=======================

Use Git to clone DigitalHub Platform repository

.. code-block:: shell
    :linenos:

    $ git clone https://github.com/scc-digitalhub/platformdocs.git

nginx-ingress
-------------

nginx-ingress is an Ingress controller that uses ConfigMap to store the nginx configuration.

Create an ingress controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: shell

   $ kubectl create namespace ingress

  $ helm install --namespace ingress --name my-release stable/nginx-ingress

During the installation, a public IP address is created for the ingress controller.

To get the public IP address use the following command:

.. code-block:: shell

   $ kubectl get service --namespace ingress

    NAME                                    TYPE           CLUSTER-IP     EXTERNAL-IP        PORT(S)                                     AGE
    my-release-ingress-controller           LoadBalancer   10.0.147.113   YOUR_EXTERNAL_IP   80:30058/TCP,443:32725/TCP                  26d
    my-release-ingress-controller-metrics   ClusterIP      10.0.48.136    <none>             9913/TCP                                    25d
    my-release-ingress-default-backend      ClusterIP      10.0.190.184   <none>             80/TCP                                      26d

cert-manager
------------

cert-manager is a Kubernetes addon to automate the management and issuance of TLS certificates from various issuing sources.

Installing with Helm
^^^^^^^^^^^^^^^^^^^^

Install the `CustomResourceDefinition` from jetstack repo.

.. code-block:: shell

    $ kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/v0.13.0/deploy/manifests/00-crds.yaml

Create cert-manager namespace.

.. code-block:: shell

    $ kubectl create namespace cert-manager

Add the jetstack helm repository

.. code-block:: shell

    $ helm repo add jetstack https://charts.jetstack.io

Update your local Helm chart repository cache.

.. code-block:: shell

    $ helm repo update

Install cert-manager with helm.

.. code-block:: shell

    $ helm install \
    --name cert-manager \
    --namespace cert-manager \
    --version v0.13.0 \
    jetstack/cert-manager


Check in cert-manager namespace if all pods are up & running

.. code-block:: shell

    $ kubectl get pods -n cert-manager
    NAME                                       READY   STATUS    RESTARTS   AGE
    cert-manager-587dc68fc4-kp8hk              1/1     Running   0          3h4m
    cert-manager-cainjector-67ff67fd45-vws89   1/1     Running   0          3h4m
    cert-manager-webhook-5c8cf6d9d4-8lv6p      1/1     Running   0          3h4m

Create `ClusterIssuer` definition.

.. code-block:: yaml

    $ cat <<EOF > clusterissuer-test.yaml
    apiVersion: cert-manager.io/v1alpha2
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-staging
    spec:
      acme:
        # You must replace this email address with your own.
        # Let's Encrypt will use this to contact you about expiring
        # certificates, and issues related to your account.
        email: user@example.com
        server: https://acme-staging-v02.api.letsencrypt.org/directory
        privateKeySecretRef:
          # Secret resource used to store the account's private key.
          name: example-issuer-account-key
        # Add a single challenge solver, HTTP01 using nginx
        solvers:
        - http01:
            ingress:
              class: nginx
    EOF

Install `ClusterIssuer`.

.. code-block:: shell

    $ kubectl apply -f clusterissuer-test.yaml

Install MySQL
-------------
Create one secret with init script e another with root credentials.

.. code-block:: shell

    $ kubectl create secret generic mysql-dbscripts --from-file=mysql/init-scripts/
    $ kubectl create secret generic mysql-db-ps --from-literal=rotps=rootpassword

Deploy MySQL container.

    $ kubectl apply -f mysql/

Install platform components
===========================

AAC
-----------

Configuration
^^^^^^^^^^^^^

Configure AAC using environment variables in aac/aac-configmap.yaml file.

See documentation for details: `https://digitalhub.readthedocs.io/en/latest/docs/service/aac.html`_

Installation
^^^^^^^^^^^^

.. code-block:: shell

    $ kubectl apply -f aac/

Org-Manager
-----------

##### Configuration

Configure Org-Manager using environment variables in org-manager/org-manager-configmap.yaml file.

See documentation for details: `https://digitalhub.readthedocs.io/en/latest/docs/service/orgman.html`_

Installation
^^^^^^^^^^^^
.. code-block:: shell

    $ kubectl apply -f org-manager/

WSO2 API Manager with APIM-Analytics
------------------------------------
1. APIM-Analytics
^^^^^^^^^^^^^^^^^
Configuration
^^^^^^^^^^^^^

Configure APIM-Analytics using environment variables in apim-analytics/apim-analytics-configmap.yml file.

See documentation for details: `https://digitalhub.readthedocs.io/en/latest/docs/service/apim.html`_

Installation
^^^^^^^^^^^^
.. code-block:: shell

    $ kubectl apply -f apim-analytics/

2. API-Manager
--------------
Configuration
^^^^^^^^^^^^^

Configure API-Manager using environment variables in api-manager/apim-configmap.yml file.

See documentation for details: `https://digitalhub.readthedocs.io/en/latest/docs/service/apim.html`_

Installation
^^^^^^^^^^^^
.. code-block:: shell

    $ kubectl apply -f api-manager/
