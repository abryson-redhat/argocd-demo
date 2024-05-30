# ArgoCD Deployments Tutorial


# Table of Contents
1. [Overview](#Overview)
2. [Architecture](#Architecture)
3. [Third Example](#third-example)
4. [Fourth Example](#fourth-examplehttpwwwfourthexamplecom)




## Overview
This project uses two applications `springboot-demo` and `springboot-postgres-demo` to demonstrate ArgoCD as a **C**ontinuous **D**elivery tool.  It is a manifest repository for the two projects.

ArgoCD synchronizes code changes real time with the runtime environment.

## Architecture
[ArgoCD](https://argo-cd.readthedocs.io/en/stable/) is a popular GitOps platform.  It is owned by Red Hat.  So, we use it a lot in GitOps engagements.

For purposes of this demonstration, ArgoCD is implemented via [Red Hat GitOps Operator](https://docs.openshift.com/gitops/1.12/installing_gitops/installing-openshift-gitops.html).  It's easy to install and configure.

This operator will be installed on CICD environment target clusters (eg. DEV cluster, QA cluster, etc).  Each cluster will have a demo namespace for the environment using this format "**\<app team name\>**-demo-**\<environment\>**".  For this example, appteam1 is the app team name and we will be demo'ing on the **dev** environment.

Within the namespace, we will have an *ArgoCD* custom resource instance.  The resource is responsible for all **C**ontinuous **D**elivery to this cluster for this team.

<br/>


### ArgoCD custom resources
ArgoCD has 2 controllers for managing application deployment events.  The first is the **Application** controller.  It manages events triggered by Application resources.  

The second is the **ApplicationSet** controller.  It manages events triggered by ApplicationSet resources. 

An <span style="color:blue">ApplicationSet</span> is simply a set of applications.  It implements a number of [generators](https://argocd-applicationset.readthedocs.io/en/stable/Generators/).  We are primarily interested in the [Git Generator](https://argocd-applicationset.readthedocs.io/en/stable/Generators-Git/).  Specifically, the Git Directory generator.

![ArgoCD decision tree](https://github.com/abryson-redhat/argocd-demo/blob/helm/images/argocd_execution_tree.png)

<br/>

### Deployments - Helm vs Kustomize

#### Continuous Delivery Sequencing




The <span style="color:blue">springboot-demo</span> application will use Helm for deployments.  Helm is a popular package management tool used by many Kubernetes shops.  

The <span style="color:blue">springboot-postgres-demo</span> application will leverage Kustomize.  Kustomize is a popular configuration management tool that can also be used to manage deployments.

The manifests for `helm` deployments will be kept in the *helm* branch of this **argocd-demo** project.  The manifests for `kustomize` will be kept in the *kustomize* branch.

<br/>

#### Helm based deployments
The manifests repository has the following structure:


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
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root-application-iac.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root-application.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/cluster-config\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/environments\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/dev\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;dev-namespace.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;dev-argocd.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/qa\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;qa-namespace.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;qa-argocd.yaml

<br/>

#### Kustomize based deployments

The manifests repository has the following structure:

---

./gitops\
&nbsp;&nbsp;            /manifests\
&nbsp;&nbsp;&nbsp;&nbsp;              <span style="background-color:light-grey">**/business unit name**<span>\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                 /integration\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                    /team\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:light-grey">**/app team name**</span>\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/apps\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/appset\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**bootstrap-appset.yaml**\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/**<app name 1>**\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/base\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-deployment.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-route.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-service.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-deployment.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;db-deployment.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;db-pvc.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;db-service.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kustomization.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/overlays\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/dev\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-deployment.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-route.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-service.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-deployment.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;db-deployment.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;db-pvc.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;db-service.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kustomization.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/qa\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-deployment.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-route.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-service.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app-deployment.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;db-deployment.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;db-pvc.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;db-service.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kustomization.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/**<app name 2>**\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;font-weight:700;font-size:20px">**...**</span>\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root-application-iac.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root-application.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/cluster-config\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/environments\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/dev\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;dev-namespace.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;dev-argocd.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/qa\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;qa-namespace.yaml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;qa-argocd.yaml







###### Helm ArgoCD manifests
At the top of the ArgoCD execution tree is a root application. It points to an ApplicationSet that generates Application instances.

Each instance is created for a given folder listed in the parent path.  

> **Root Application**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  finalizers:
  - resources-finalizer.argocd.argoproj.io
  name: appteam1-root-dev
  namespace: appteam1-demo-dev
  labels:
    demo/team.display.level: root-helm
spec:
  destination:
    namespace: appteam1-demo-dev
    server: https://kubernetes.default.svc
  project: default
  source:
    path: gitops/manifests/busunit1/integration/teams/appteam1/apps/appset
    repoURL: git@github.com:abryson-redhat/argocd-demo.git
    targetRevision: helm
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

> ApplicationSet
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: appteam1-apps-helm-dev
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
  - git:
      repoURL: https://github.com/abryson-redhat/argocd-demo.git
      revision: helm
      directories:
      - path: gitops/manifests/busunit1/integration/teams/appteam1/apps/*
      - exclude: true
        path: gitops/manifests/busunit1/integration/teams/appteam1/apps/appset
  template:
    metadata:
      name: '{{.path.basename}}'
    spec:
      project: "default"
      source:
        repoURL: https://github.com/abryson-redhat/argocd-demo.git
        targetRevision: helm
        path: '{{.path.path}}'
        helm:
          valueFiles:
          - 'configs/dev/values.yaml'
      destination:
        server: https://kubernetes.default.svc
        namespace: appteam1-demo-dev
      syncPolicy:
        syncOptions:
        - CreateNamespace=false
```


> Auto-generated application - **springboot-example**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  creationTimestamp: "2024-05-29T14:55:13Z"
  finalizers:
  - resources-finalizer.argocd.argoproj.io
  generation: 333
  name: springboot-demo
  namespace: appteam1-demo-dev
  ownerReferences:
  - apiVersion: argoproj.io/v1alpha1
    blockOwnerDeletion: true
    controller: true
    kind: ApplicationSet
    name: appteam1-apps-helm-dev
    uid: de7f96df-2d41-445f-a244-47919f2b9a81
  resourceVersion: "600471"
  uid: 1485dc1a-5f94-4608-83b7-6d2d927162a9
spec:
  destination:
    namespace: appteam1-demo-dev
    server: https://kubernetes.default.svc
  project: default
  source:
    helm:
      valueFiles:
      - configs/dev/values.yaml
    path: gitops/manifests/busunit1/integration/teams/appteam1/apps/springboot-demo
    repoURL: https://github.com/abryson-redhat/argocd-demo.git
    targetRevision: helm
  syncPolicy:
    syncOptions:
    - CreateNamespace=false
status:
  controllerNamespace: appteam1-demo-dev
  health:
    status: Healthy
  history:
  - deployStartedAt: "2024-05-29T14:55:41Z"
    deployedAt: "2024-05-29T14:55:41Z"
    id: 0
    revision: 4c59f8af1b70f5a02417b6c85209e28a6d792222
    source:
      helm:
        valueFiles:
        - configs/dev/values.yaml
      path: gitops/manifests/busunit1/integration/teams/appteam1/apps/springboot-demo
      repoURL: https://github.com/abryson-redhat/argocd-demo.git
      targetRevision: helm
  operationState:
    finishedAt: "2024-05-29T14:55:41Z"
    message: successfully synced (all tasks run)
    operation:
      initiatedBy:
        username: admin
      retry: {}
      sync:
        prune: true
        revision: 4c59f8af1b70f5a02417b6c85209e28a6d792222
        syncOptions:
        - CreateNamespace=false
        syncStrategy:
          hook: {}
    phase: Succeeded
    startedAt: "2024-05-29T14:55:41Z"
    syncResult:
      resources:
      - group: ""
        hookPhase: Running
        kind: ServiceAccount
        message: serviceaccount/springboot-demo created
        name: springboot-demo
        namespace: appteam1-demo-dev
        status: Synced
        syncPhase: Sync
        version: v1
      - group: ""
        hookPhase: Running
        kind: Service
        message: service/springboot-demo created
        name: springboot-demo
        namespace: appteam1-demo-dev
        status: Synced
        syncPhase: Sync
        version: v1
      - group: apps
        hookPhase: Running
        kind: Deployment
        message: deployment.apps/springboot-demo created
        name: springboot-demo
        namespace: appteam1-demo-dev
        status: Synced
        syncPhase: Sync
        version: v1
      - group: route.openshift.io
        hookPhase: Running
        kind: Route
        message: route.route.openshift.io/springboot-demo created
        name: springboot-demo
        namespace: appteam1-demo-dev
        status: Synced
        syncPhase: Sync
        version: v1
      revision: 4c59f8af1b70f5a02417b6c85209e28a6d792222
      source:
        helm:
          valueFiles:
          - configs/dev/values.yaml
        path: gitops/manifests/busunit1/integration/teams/appteam1/apps/springboot-demo
        repoURL: https://github.com/abryson-redhat/argocd-demo.git
        targetRevision: helm
  reconciledAt: "2024-05-30T16:55:15Z"
  resources:
  - health:
      status: Healthy
    kind: Service
    name: springboot-demo
    namespace: appteam1-demo-dev
    status: Synced
    version: v1
  - kind: ServiceAccount
    name: springboot-demo
    namespace: appteam1-demo-dev
    status: Synced
    version: v1
  - group: apps
    health:
      status: Healthy
    kind: Deployment
    name: springboot-demo
    namespace: appteam1-demo-dev
    status: Synced
    version: v1
  - group: route.openshift.io
    health:
      message: Route is healthy
      status: Healthy
    kind: Route
    name: springboot-demo
    namespace: appteam1-demo-dev
    status: Synced
    version: v1
  sourceType: Helm
  summary:
    images:
    - nexus-registry-nexus.apps.cluster-bcttf.dynamic.redhatworkshops.io/repository/smbc-demo/springboot-demo:latest
  sync:
    comparedTo:
      destination:
        namespace: appteam1-demo-dev
        server: https://kubernetes.default.svc
      source:
        helm:
          valueFiles:
          - configs/dev/values.yaml
        path: gitops/manifests/busunit1/integration/teams/appteam1/apps/springboot-demo
        repoURL: https://github.com/abryson-redhat/argocd-demo.git
        targetRevision: helm
    revision: 72de7430c679cd7e781ccc2eeee029b90cb2eda6
    status: Synced

```

## `springboot-demo` Project
This is a simple springboot based Rest Controller helloworld application. It has a `Containerfile` for createing an OCI image.  The image will be built using the CI Github Actions portion of this demo.  Which will not be addressed in this document. 






## `springboot-postgres-demo` Project
This is a springboot based web application with a PostgreSQL backend.  It is implemented as 2 separate deployments:  
- PostgreSQL back end implemented as a Kubernetes deployment.
- springboot-postgres-application web application.  The application uses JPA for persistence.

