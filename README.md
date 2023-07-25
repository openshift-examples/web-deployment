# GitOps artefacts to deploy https://examples.openshift.pub


## ArgoCD Instance configuration

Important part is `resourceExclusions`


```yaml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: gitops
spec:
  applicationSet:
    resources:
      limits:
        cpu: "2"
        memory: 1Gi
      requests:
        cpu: 250m
        memory: 512Mi
    webhookServer:
      ingress:
        enabled: false
      route:
        enabled: false
  controller:
    processors: {}
    resources:
      limits:
        cpu: "2"
        memory: 2Gi
      requests:
        cpu: 250m
        memory: 1Gi
    sharding: {}
  grafana:
    enabled: false
    ingress:
      enabled: false
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 250m
        memory: 128Mi
    route:
      enabled: false
  ha:
    enabled: false
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 250m
        memory: 128Mi
  initialSSHKnownHosts: {}
  monitoring:
    enabled: false
  notifications:
    enabled: false
  prometheus:
    enabled: false
    ingress:
      enabled: false
    route:
      enabled: false
  rbac:
    policy: |
      g, system:cluster-admins, role:admin
      g, cluster-admins, role:admin
    scopes: '[groups]'
  redis:
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 250m
        memory: 128Mi
  repo:
    resources:
      limits:
        cpu: "1"
        memory: 1Gi
      requests:
        cpu: 250m
        memory: 256Mi
  resourceExclusions: |
    - apiGroups:
      - monitoring.coreos.com
      - network.openshift.io
      clusters:
      - 'https://api.rh-us-east-1.openshift.com'
      kinds:
      - '*'
    - apiGroups:
      - '*'
      clusters:
      - 'https://api.rh-us-east-1.openshift.com'
      kinds:
      - PodTemplate
    - apiGroups:
      - 'apps'
      clusters:
      - 'https://api.rh-us-east-1.openshift.com'
      kinds:
      - ControllerRevision

    - apiGroups:
      - tekton.dev
      clusters:
      - '*'
      kinds:
      - TaskRun
      - PipelineRun
  server:
    autoscale:
      enabled: false
    grpc:
      ingress:
        enabled: false
    ingress:
      enabled: false
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 125m
        memory: 128Mi
    route:
      enabled: true
    service:
      type: ""
  sso:
    dex:
      openShiftOAuth: true
      resources:
        limits:
          cpu: 500m
          memory: 256Mi
        requests:
          cpu: 250m
          memory: 128Mi
    provider: dex
  tls:
    ca: {}
```
Source: >https://github.com/rbo/gitops-cluster/blob/main/apps/onlogic/examples-openshift-pub-cicd/ArgoCD/gitops.yaml>


### Add to argocd instance

#### Create service account for argocd & fetch TOKEN

```bash
oc login https://api.rh-us-east-1.openshift.com:443 --token=$TOKEN

oc create sa argocd

oc create rolebinding argocd  -n production \
    --clusterrole=admin \
    --serviceaccount="production:argocd"

oc create rolebinding argocd  -n openshiftanwender \
    --clusterrole=admin \
    --serviceaccount="production:argocd"

TOKEN=$(oc sa get-token argocd)
```

##### Added access to argocd instance

```bash
export KUBECONFIG=/tmp/foo

oc login https://api.rh-us-east-1.openshift.com:443 --token=$TOKEN

oc config rename-context \
    openshiftanwender/api-rh-us-east-1-openshift-com:443/system:serviceaccount:production:argocd \
    employee-ocp-production

argocd cluster add employee-ocp-production \
    --system-namespace production \
    --namespace production \
    --service-account argocd

```
