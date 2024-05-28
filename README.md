# ArgoCD Deployments Tutorial


# Table of Contents
1. [Overview](#Overview)
2. [Architecture](#Architecture)
3. [Third Example](#third-example)
4. [Fourth Example](#fourth-examplehttpwwwfourthexamplecom)




## Overview
This project uses two applications `springboot-demo` and `springboot-postgres-demo` to demonstrate 

## Architecture
[ArgoCD](https://argo-cd.readthedocs.io/en/stable/) is a popular GitOps platform.  It is owned by Red Hat.  So, we use it a lot in GitOps engagements.

For purposes of this demonstration, ArgoCD is implemented via [Red Hat GitOps Operator](https://docs.openshift.com/gitops/1.12/installing_gitops/installing-openshift-gitops.html).  It's easy to install and configure.

This operator will be installed on CICD environment target clusters (eg. DEV cluster, QA cluster, etc).  Each cluster will have a demo namespace for the environment using this format "**\<app team name\>**-demo-**\<environment\>**".  For this example, appteam1 is the app team name and we will be demo'ing on the **dev** environment.

Within the namespace, we will have an *ArgoCD* custom resource instance.  That resource is responsible for all Continuous Delivery to this cluster for this team.

</br>

<img src="https://github.com/abryson-redhat/argocd-demo/blob/helm/images/argocd_execution_tree.jpg" alt="ArgoCD decision tree" style="float: left; margin-right: 10px;"/>

</br>

### Deployments - Helm vs Kustomize
The <span style="color:blue">springboot-demo</span> application will use Helm for deployments.  Helm is a popular package management tool used by many Kubernetes shops.  

The <span style="color:blue">springboot-postgres-demo</span> application will leverage Kustomize.  Kustomize is a popular configuration management tool that can also be used to manage deployments.

The manifests for `helm` deployments will be kept in the *helm* branch of this **argocd-demo** project.  The manifests for `kustomize` will be kept in the *kustomize* branch.

##### The manifests repository has the following structure:</large>

###### Helm
---


./gitops\
&nbsp;&nbsp;            /manifests\
&nbsp;&nbsp;&nbsp;&nbsp;              <span style="background-color:light-grey">**/business unit name**<span>\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                 /integration\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                    /team\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:light-grey">**/app team name**</span>\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/apps\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Chart.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/appset\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**bootstrap-appset.yaml**\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/configs\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/dev\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/qa\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/**<app name 1>**\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/**<app name 2>**\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/templates\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;values.yaml\
###### Kustomize
---

###### Application Set details


At the top of the ArgoCD execution tree is a
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: appteam1-apps-dev
spec:
  generators:
  - list:
      elements:
      - cluster: in-cluster
        url: https://kubernetes.default.svc
        environment: dev
        app: python-demo
        repoPath: "gitops/manifests/busunit1/integration/teams/appteam1/apps/python-demo"
        repoURL: "git@github.com:abryson-redhat/argocd-demo.git"
        ref: argocd-demo
        valueFile: "configs/dev/values.yaml"
        namespace: appteam1-demo-dev
      - cluster: in-cluster
        url: https://kubernetes.default.svc
        environment: dev
        app: springboot-demo
        repoPath: "gitops/manifests/busunit1/integration/teams/appteam1/apps/springboot-demo"
        repoURL: "git@github.com:abryson-redhat/argocd-demo.git"
        ref: argocd-demo
        valueFile: "configs/dev/values.yaml"
        namespace: appteam1-demo-dev
```



## `springboot-demo` Project
This is a simple springboot based Rest Controller helloworld application. It has a `Containerfile` for createing an OCI image.  The image will be built using the CI Github Actions portion of this demo.  Which will not be addressed in this document. 






## `springboot-postgres-demo` Project
This is a springboot based web application with a PostgreSQL backend.  It is implemented as 2 separate deployments:  
- PostgreSQL back end implemented as a Kubernetes deployment.
- springboot-postgres-application web application.  The application uses JPA for persistence.

