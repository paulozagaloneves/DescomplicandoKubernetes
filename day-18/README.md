# Day 18 - Descomplicando RBAC nos Kubernetes

## Índice

- [Day 18 - Descomplicando RBAC nos Kubernetes](#day-18---descomplicando-rbac-nos-kubernetes)
  - [Índice](#índice)
  - [O que é RBAC](#o-que-é-rbac)
  - [Componentes principais](#componentes-principais)
    - [Role](#role)
    - [RoleBinding](#rolebinding)
    - [ClusterRole](#clusterrole)
    - [ClusterRoleBinding](#clusterrolebinding)
  - [Resumo visual](#resumo-visual)
  - [Criando e assinando um certificado no kubernetes](#criando-e-assinando-um-certificado-no-kubernetes)
    - [Chave privada](#chave-privada)
    - [CSR](#csr)
    - [Certificate Signing Request](#certificate-signing-request)
    - [Approve Certificate](#approve-certificate)
  - [Role Binding](#role-binding)
    - [Api Resources](#api-resources)
      - [Namespaced](#namespaced)
      - [Verbs](#verbs)
    - [Role Binding](#role-binding-1)
      - [Role](#role-1)
      - [Criando Role com kubectl](#criando-role-com-kubectl)
      - [Role Biding](#role-biding)
      - [Criando RoleBinding com kubectl](#criando-rolebinding-com-kubectl)
  - [Configurando User e contexto kubectl config](#configurando-user-e-contexto-kubectl-config)
    - [Criando Contexto](#criando-contexto)
    - [Trocar de Contexto](#trocar-de-contexto)
    - [Testando User Aula](#testando-user-aula)
    - [Listando permissões](#listando-permissões)
  - [Alterando permissões](#alterando-permissões)
  - [ClusterRole e ClusterRoleBinding](#clusterrole-e-clusterrolebinding)
    - [Criar certificado](#criar-certificado)
    - [Criar Certificate Request](#criar-certificate-request)
    - [Certificate Signing Request](#certificate-signing-request-1)
    - [Aprovar certificado](#aprovar-certificado)
    - [Obter certificado](#obter-certificado)
    - [ClusterRole](#clusterrole-1)
    - [ClusterRole Binding](#clusterrole-binding)
    - [Configurando config](#configurando-config)
      - [1. Credentials](#1-credentials)
      - [2. Context](#2-context)
      - [3. Trocar de contexto](#3-trocar-de-contexto)
  - [ServiceAccount com Token](#serviceaccount-com-token)
    - [Para que serve](#para-que-serve)
    - [Token do ServiceAccount](#token-do-serviceaccount)
    - [Meu ServiceAccount](#meu-serviceaccount)
      - [Criando serviceAccount com kubectl](#criando-serviceaccount-com-kubectl)
    - [ServiceAccount Secret](#serviceaccount-secret)
      - [Criando ServiceAccount Token com kubectl](#criando-serviceaccount-token-com-kubectl)
    - [Obter Token](#obter-token)
    - [ServiceAccount Role](#serviceaccount-role)
    - [ServiceAccount RoleBinding](#serviceaccount-rolebinding)
    - [Testando](#testando)
  - [Validando permissoes](#validando-permissoes)

## O que é RBAC

RBAC (Role-Based Access Control) é um mecanismo de controle de acesso do Kubernetes que determina quais usuários, grupos ou service accounts podem executar quais ações em quais recursos. Ele funciona através de políticas que definem permissões granulares no cluster.

## Componentes principais

### Role

Uma Role define um conjunto de permissões dentro de um namespace específico. Ela especifica quais verbos (ações) podem ser executados em quais recursos.

**Características:**
- Escopo: **Namespace** (limitada a um namespace específico)
- Define permissões para recursos como pods, deployments, services, etc.
- Exemplo: permissão para listar e visualizar pods apenas

### RoleBinding

Um RoleBinding conecta uma Role a usuários, grupos ou service accounts dentro de um namespace. Ele atribui as permissões definidas na Role aos sujeitos.

**Características:**
- Escopo: **Namespace**
- Estabelece a relação entre Role e usuários/grupos/service accounts
- Múltiplos sujeitos podem ser referenciados em um único RoleBinding

### ClusterRole

Uma ClusterRole é semelhante a uma Role, porém com escopo de cluster inteiro. Define permissões que podem ser aplicadas em todos os namespaces ou para recursos globais.

**Características:**
- Escopo: **Cluster inteiro**
- Pode conceder acesso a recursos de qualquer namespace
- Aceita recursos que não pertencem a namespaces (nodes, persistentvolumes, etc.)
- Exemplo: permissão de administrador do cluster

### ClusterRoleBinding

Um ClusterRoleBinding conecta uma ClusterRole a usuários, grupos ou service accounts em nível de cluster. Concede permissões em todo o cluster.

**Características:**
- Escopo: **Cluster inteiro**
- Atribui permissões de ClusterRole a nível global
- Possibilita acesso a todos os namespaces e recursos globais

## Resumo visual

| Componente             | Escopo    | Uso                                     |
| ---------------------- | --------- | --------------------------------------- |
| **Role**               | Namespace | Permissões limitadas a um namespace     |
| **RoleBinding**        | Namespace | Concede Role a sujeitos em um namespace |
| **ClusterRole**        | Cluster   | Permissões em todo o cluster            |
| **ClusterRoleBinding** | Cluster   | Concede ClusterRole globalmente         |

## Criando e assinando um certificado no kubernetes

### Chave privada

```bash
$ openssl genrsa -out developer.key 2048
$ ls -l
total 8
-rw------- 1 paulo paulo 1708 mar 28 07:34 developer.key
-rw-rw-r-- 1 paulo paulo 2843 mar 28 07:35 README.md
$ cat developer.key                                     
-----BEGIN PRIVATE KEY-----

-----END PRIVATE KEY-----
```

### CSR

```bash
$ openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"
$ ls -l
total 12
-rw-rw-r-- 1 paulo paulo  891 mar 28 07:39 developer.csr
-rw------- 1 paulo paulo 1708 mar 28 07:34 developer.key
-rw-rw-r-- 1 paulo paulo 3260 mar 28 07:39 README.md
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ cat developer.csr 
-----BEGIN CERTIFICATE REQUEST-----
MIICWTCCAUECAQAwFDESMBAGA1UEAwwJZGV2ZWxvcGVyMIIBIjANBgkqhkiG9w0B
AQEFAAOCAQ8AMIIBCgKCAQEAsSyGfT91ThrxzGghMKEBu+gn6z+x9av+1u6E2XGA
kgrLqVBwI/sIwPlcmisnMp24ctlLflsMddfRtVPKuZS7fECdXX37TqLGaKWV6uis
...
4I2CRIdLsak14znNe5hzSeuYbHsLJXrHJiSJX3yuOZZF8roPcieLhQGNimSQRjsi
UnRKMZ2fZHB8FayIe/VIZxvAiqRssl4O3B47AVq0oAbPUteoyQgAEE1xIfmmf/Tq
NCAkTgH7UVnL9jCAbFFJliCUjd9q3o/+AP+o4gQ=
-----END CERTIFICATE REQUEST-----
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ cat developer.csr | base64 | tr -d '\n'
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dUQ0NBVUVDQVFBd0ZERVNNQkFHQTFVRUF3d0paR1YyWld4dmNHVnlNSUlCSWpBTkJna3Foa2lHOXcwQgpBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFzU3l...eUllL1ZJWnh2QWlxUnNzbDRPM0I0N0FWcTBvQWJQVXRlb3lRZ0FFRTF4SWZtbWYvVHEKTkNBa1RnSDdVVm5MOWpDQWJGRkpsaUNVamQ5cTNvLytBUCtvNGdRPQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K%  
```

### Certificate Signing Request

**File:** developer.yaml

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: developer
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVN...LS0tLS0K
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 31536000
  usages:
    - client auth
```

```bash
$ kubectl apply -f developer.yaml                   
certificatesigningrequest.certificates.k8s.io/developer created
$
$
$ kubectl get csr     
NAME        AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
developer   84s   kubernetes.io/kube-apiserver-client   kubernetes-admin   365d                Pending
$
$
$ kubectl describe csr developer                                
Name:         developer
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"certificates.k8s.io/v1","kind":"CertificateSigningRequest","metadata":{"annotations":{},"name":"developer"},"spec":{"expirationSeconds":31536000,.."request":"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dUQ0NBVUVDQVFBd0Z
...
WJGRkpsaUNVamQ5cTNvLytBUCtvNGdRPQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K","signerName":"kubernetes.io/kube-apiserver-client","usages":["client auth"]}}

CreationTimestamp:   Sat, 28 Mar 2026 07:50:08 +0000
Requesting User:     kubernetes-admin
Signer:              kubernetes.io/kube-apiserver-client
Requested Duration:  365d
Status:              Pending
Subject:
         Common Name:    developer
         Serial Number:  
Events:  <none>

```

### Approve Certificate

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl certificate approve developer 
certificatesigningrequest.certificates.k8s.io/developer approved
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl get csr                      
NAME        AGE     SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
developer   9m12s   kubernetes.io/kube-apiserver-client   kubernetes-admin   365d                Approved,Issued
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$
```

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ ls   
developer.csr  developer.key  developer.yaml  README.md
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl get csr developer -o jsonpath='{.status.certificate}'
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMrVENDQWVHZ0F3SUJBZ0lRUWVlbHhvd0RmUndNSXNnSWNYV2pDekFOQmdrcWhraUc5dzBCQVFzRkFEQVYKTVJNd0VRWURWUVFERXdwcmRXSmxjbTVsZEdWek1CNFhEVE
...
xSNjlJUkNlMkJLdDZ4aHJNY2RmRUhVTkdtU3lGMDc2bkZiTURRaklCREpjWQphOGlhTHlWQ1ZpL3VBY2ducGFESVRCNGNpZE4vUThPc2ZlNjBocXA1R0NJMnpDSjNpT3ZYeE45ODJreUdRTEJSCmRSODE1U0RlaHo1ZXErWTIyOVJ1bjQ0UmdSMWxjc1FKWEVuVDJnS2J3cEd1anpPblNDR0VVNzhyZzhDVAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==% 
$
$
$
aulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl get csr developer -o jsonpath='{.status.certificate}' | base64 --decode > developer.crt
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ cat developer.crt                      
-----BEGIN CERTIFICATE-----
MIIC+TCCAeGgAwIBAgIQQeelxowDfRwMIsgIcXWjCzANBgkqhkiG9w0BAQsFADAV
...
a8iaLyVCVi/uAcgnpaDITB4cidN/Q8Osfe60hqp5GCI2zCJ3iOvXxN982kyGQLBR
dR815SDehz5eq+Y229Run44RgR1lcsQJXEnT2gKbwpGujzOnSCGEU78rg8CT
-----END CERTIFICATE-----
```

## Role Binding

### Api Resources

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl api-resources                                                                          
NAME                                SHORTNAMES                          APIVERSION                          NAMESPACED   KIND
bindings                                                                v1                                  true         Binding
componentstatuses                   cs                                  v1                                  false        ComponentStatus
configmaps                          cm                                  v1                                  true         ConfigMap
endpoints                           ep                                  v1                                  true         Endpoints
events                              ev                                  v1                                  true         Event
limitranges                         limits                              v1                                  true         LimitRange
namespaces                          ns                                  v1                                  false        Namespace
nodes                               no                                  v1                                  false        Node
persistentvolumeclaims              pvc                                 v1                                  true         PersistentVolumeClaim
persistentvolumes                   pv                                  v1                                  false        PersistentVolume
pods                                po                                  v1                                  true         Pod
podtemplates                                                            v1                                  true         PodTemplate
replicationcontrollers              rc                                  v1                                  true         ReplicationController
resourcequotas                      quota                               v1                                  true         ResourceQuota
secrets                                                                 v1                                  true         Secret
serviceaccounts                     sa                                  v1                                  true         ServiceAccount
services                            svc                                 v1                                  true         Service
challenges                                                              acme.cert-manager.io/v1             true         Challenge
orders                                                                  acme.cert-manager.io/v1             true         Order
mutatingwebhookconfigurations                                           admissionregistration.k8s.io/v1     false        MutatingWebhookConfiguration
validatingadmissionpolicies                                             admissionregistration.k8s.io/v1     false        ValidatingAdmissionPolicy
validatingadmissionpolicybindings                                       admissionregistration.k8s.io/v1     false        ValidatingAdmissionPolicyBinding
validatingwebhookconfigurations                                         admissionregistration.k8s.io/v1     false        ValidatingWebhookConfiguration
customresourcedefinitions           crd,crds                            apiextensions.k8s.io/v1             false        CustomResourceDefinition
apiservices                                                             apiregistration.k8s.io/v1           false        APIService
controllerrevisions                                                     apps/v1                             true         ControllerRevision
daemonsets                          ds                                  apps/v1                             true         DaemonSet
deployments                         deploy                              apps/v1                             true         Deployment
replicasets                         rs                                  apps/v1                             true         ReplicaSet
statefulsets                        sts                                 apps/v1                             true         StatefulSet
selfsubjectreviews                                                      authentication.k8s.io/v1            false        SelfSubjectReview
tokenreviews                                                            authentication.k8s.io/v1            false        TokenReview
localsubjectaccessreviews                                               authorization.k8s.io/v1             true         LocalSubjectAccessReview
selfsubjectaccessreviews                                                authorization.k8s.io/v1             false        SelfSubjectAccessReview
selfsubjectrulesreviews                                                 authorization.k8s.io/v1             false        SelfSubjectRulesReview
subjectaccessreviews                                                    authorization.k8s.io/v1             false        SubjectAccessReview
horizontalpodautoscalers            hpa                                 autoscaling/v2                      true         HorizontalPodAutoscaler
cronjobs                            cj                                  batch/v1                            true         CronJob
jobs                                                                    batch/v1                            true         Job
certificaterequests                 cr,crs                              cert-manager.io/v1                  true         CertificateRequest
certificates                        cert,certs                          cert-manager.io/v1                  true         Certificate
clusterissuers                      ciss                                cert-manager.io/v1                  false        ClusterIssuer
issuers                             iss                                 cert-manager.io/v1                  true         Issuer
certificatesigningrequests          csr                                 certificates.k8s.io/v1              false        CertificateSigningRequest
ciliumcidrgroups                    ccg                                 cilium.io/v2                        false        CiliumCIDRGroup
ciliumclusterwidenetworkpolicies    ccnp                                cilium.io/v2                        false        CiliumClusterwideNetworkPolicy
ciliumendpoints                     cep,ciliumep                        cilium.io/v2                        true         CiliumEndpoint
ciliumidentities                    ciliumid                            cilium.io/v2                        false        CiliumIdentity
ciliuml2announcementpolicies        l2announcement                      cilium.io/v2alpha1                  false        CiliumL2AnnouncementPolicy
ciliumloadbalancerippools           ippools,ippool,lbippool,lbippools   cilium.io/v2                        false        CiliumLoadBalancerIPPool
ciliumnetworkpolicies               cnp,ciliumnp                        cilium.io/v2                        true         CiliumNetworkPolicy
ciliumnodeconfigs                                                       cilium.io/v2                        true         CiliumNodeConfig
ciliumnodes                         cn,ciliumn                          cilium.io/v2                        false        CiliumNode
ciliumpodippools                    cpip                                cilium.io/v2alpha1                  false        CiliumPodIPPool
leases                                                                  coordination.k8s.io/v1              true         Lease
endpointslices                                                          discovery.k8s.io/v1                 true         EndpointSlice
events                              ev                                  events.k8s.io/v1                    true         Event
flowschemas                                                             flowcontrol.apiserver.k8s.io/v1     false        FlowSchema
prioritylevelconfigurations                                             flowcontrol.apiserver.k8s.io/v1     false        PriorityLevelConfiguration
backendtlspolicies                  btlspolicy                          gateway.networking.k8s.io/v1        true         BackendTLSPolicy
gatewayclasses                      gc                                  gateway.networking.k8s.io/v1        false        GatewayClass
gateways                            gtw                                 gateway.networking.k8s.io/v1        true         Gateway
grpcroutes                                                              gateway.networking.k8s.io/v1        true         GRPCRoute
httproutes                                                              gateway.networking.k8s.io/v1        true         HTTPRoute
referencegrants                     refgrant                            gateway.networking.k8s.io/v1beta1   true         ReferenceGrant
accesscontrolpolicies                                                   hub.traefik.io/v1alpha1             false        AccessControlPolicy
aiservices                                                              hub.traefik.io/v1alpha1             true         AIService
apiauths                                                                hub.traefik.io/v1alpha1             true         APIAuth
apibundles                                                              hub.traefik.io/v1alpha1             true         APIBundle
apicatalogitems                                                         hub.traefik.io/v1alpha1             true         APICatalogItem
apiplans                                                                hub.traefik.io/v1alpha1             true         APIPlan
apiportalauths                                                          hub.traefik.io/v1alpha1             true         APIPortalAuth
apiportals                                                              hub.traefik.io/v1alpha1             true         APIPortal
apiratelimits                                                           hub.traefik.io/v1alpha1             true         APIRateLimit
apis                                                                    hub.traefik.io/v1alpha1             true         API
apiversions                                                             hub.traefik.io/v1alpha1             true         APIVersion
managedapplications                                                     hub.traefik.io/v1alpha1             true         ManagedApplication
managedsubscriptions                                                    hub.traefik.io/v1alpha1             true         ManagedSubscription
cleanuppolicies                     cleanpol                            kyverno.io/v2                       true         CleanupPolicy
clustercleanuppolicies              ccleanpol                           kyverno.io/v2                       false        ClusterCleanupPolicy
clusterpolicies                     cpol                                kyverno.io/v1                       false        ClusterPolicy
globalcontextentries                gctxentry                           kyverno.io/v2                       false        GlobalContextEntry
policies                            pol                                 kyverno.io/v1                       true         Policy
policyexceptions                    polex                               kyverno.io/v2                       true         PolicyException
updaterequests                      ur                                  kyverno.io/v2                       true         UpdateRequest
bfdprofiles                                                             metallb.io/v1beta1                  true         BFDProfile
bgpadvertisements                                                       metallb.io/v1beta1                  true         BGPAdvertisement
bgppeers                                                                metallb.io/v1beta2                  true         BGPPeer
communities                                                             metallb.io/v1beta1                  true         Community
configurationstates                                                     metallb.io/v1beta1                  true         ConfigurationState
ipaddresspools                                                          metallb.io/v1beta1                  true         IPAddressPool
l2advertisements                                                        metallb.io/v1beta1                  true         L2Advertisement
servicebgpstatuses                                                      metallb.io/v1beta1                  true         ServiceBGPStatus
servicel2statuses                                                       metallb.io/v1beta1                  true         ServiceL2Status
nodes                                                                   metrics.k8s.io/v1beta1              false        NodeMetrics
pods                                                                    metrics.k8s.io/v1beta1              true         PodMetrics
alertmanagerconfigs                 amcfg                               monitoring.coreos.com/v1alpha1      true         AlertmanagerConfig
alertmanagers                       am                                  monitoring.coreos.com/v1            true         Alertmanager
podmonitors                         pmon                                monitoring.coreos.com/v1            true         PodMonitor
probes                              prb                                 monitoring.coreos.com/v1            true         Probe
prometheusagents                    promagent                           monitoring.coreos.com/v1alpha1      true         PrometheusAgent
prometheuses                        prom                                monitoring.coreos.com/v1            true         Prometheus
prometheusrules                     promrule                            monitoring.coreos.com/v1            true         PrometheusRule
scrapeconfigs                       scfg                                monitoring.coreos.com/v1alpha1      true         ScrapeConfig
servicemonitors                     smon                                monitoring.coreos.com/v1            true         ServiceMonitor
thanosrulers                        ruler                               monitoring.coreos.com/v1            true         ThanosRuler
ingressclasses                                                          networking.k8s.io/v1                false        IngressClass
ingresses                           ing                                 networking.k8s.io/v1                true         Ingress
ipaddresses                         ip                                  networking.k8s.io/v1                false        IPAddress
networkpolicies                     netpol                              networking.k8s.io/v1                true         NetworkPolicy
servicecidrs                                                            networking.k8s.io/v1                false        ServiceCIDR
runtimeclasses                                                          node.k8s.io/v1                      false        RuntimeClass
deletingpolicies                    dpol                                policies.kyverno.io/v1              false        DeletingPolicy
generatingpolicies                  gpol                                policies.kyverno.io/v1              false        GeneratingPolicy
imagevalidatingpolicies             ivpol                               policies.kyverno.io/v1              false        ImageValidatingPolicy
mutatingpolicies                    mpol                                policies.kyverno.io/v1              false        MutatingPolicy
namespaceddeletingpolicies          ndpol                               policies.kyverno.io/v1              true         NamespacedDeletingPolicy
namespacedgeneratingpolicies        ngpol                               policies.kyverno.io/v1              true         NamespacedGeneratingPolicy
namespacedimagevalidatingpolicies   nivpol                              policies.kyverno.io/v1              true         NamespacedImageValidatingPolicy
namespacedmutatingpolicies          nmpol                               policies.kyverno.io/v1              true         NamespacedMutatingPolicy
namespacedvalidatingpolicies        nvpol                               policies.kyverno.io/v1              true         NamespacedValidatingPolicy
policyexceptions                                                        policies.kyverno.io/v1              true         PolicyException
validatingpolicies                  vpol                                policies.kyverno.io/v1              false        ValidatingPolicy
poddisruptionbudgets                pdb                                 policy/v1                           true         PodDisruptionBudget
clusterrolebindings                                                     rbac.authorization.k8s.io/v1        false        ClusterRoleBinding
clusterroles                                                            rbac.authorization.k8s.io/v1        false        ClusterRole
rolebindings                                                            rbac.authorization.k8s.io/v1        true         RoleBinding
roles                                                                   rbac.authorization.k8s.io/v1        true         Role
clusterephemeralreports             cephr                               reports.kyverno.io/v1               false        ClusterEphemeralReport
ephemeralreports                    ephr                                reports.kyverno.io/v1               true         EphemeralReport
deviceclasses                                                           resource.k8s.io/v1                  false        DeviceClass
resourceclaims                                                          resource.k8s.io/v1                  true         ResourceClaim
resourceclaimtemplates                                                  resource.k8s.io/v1                  true         ResourceClaimTemplate
resourceslices                                                          resource.k8s.io/v1                  false        ResourceSlice
priorityclasses                     pc                                  scheduling.k8s.io/v1                false        PriorityClass
csidrivers                                                              storage.k8s.io/v1                   false        CSIDriver
csinodes                                                                storage.k8s.io/v1                   false        CSINode
csistoragecapacities                                                    storage.k8s.io/v1                   true         CSIStorageCapacity
storageclasses                      sc                                  storage.k8s.io/v1                   false        StorageClass
volumeattachments                                                       storage.k8s.io/v1                   false        VolumeAttachment
volumeattributesclasses             vac                                 storage.k8s.io/v1                   false        VolumeAttributesClass
ingressroutes                                                           traefik.io/v1alpha1                 true         IngressRoute
ingressroutetcps                                                        traefik.io/v1alpha1                 true         IngressRouteTCP
ingressrouteudps                                                        traefik.io/v1alpha1                 true         IngressRouteUDP
middlewares                                                             traefik.io/v1alpha1                 true         Middleware
middlewaretcps                                                          traefik.io/v1alpha1                 true         MiddlewareTCP
serverstransports                                                       traefik.io/v1alpha1                 true         ServersTransport
serverstransporttcps                                                    traefik.io/v1alpha1                 true         ServersTransportTCP
tlsoptions                                                              traefik.io/v1alpha1                 true         TLSOption
tlsstores                                                               traefik.io/v1alpha1                 true         TLSStore
traefikservices                                                         traefik.io/v1alpha1                 true         TraefikService
clusterpolicyreports                cpolr                               wgpolicyk8s.io/v1alpha2             false        ClusterPolicyReport
policyreports                       polr                                wgpolicyk8s.io/v1alpha2             true         PolicyReport
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ 
```

```bash
$ kubectl api-resources --namespaced=false
NAME                                SHORTNAMES                          APIVERSION                        NAMESPACED   KIND
componentstatuses                   cs                                  v1                                false        ComponentStatus
namespaces                          ns                                  v1                                false        Namespace
nodes                               no                                  v1                                false        Node
persistentvolumes                   pv                                  v1                                false        PersistentVolume
mutatingwebhookconfigurations                                           admissionregistration.k8s.io/v1   false        MutatingWebhookConfiguration
validatingadmissionpolicies                                             admissionregistration.k8s.io/v1   false        ValidatingAdmissionPolicy
validatingadmissionpolicybindings                                       admissionregistration.k8s.io/v1   false        ValidatingAdmissionPolicyBinding
validatingwebhookconfigurations                                         admissionregistration.k8s.io/v1   false        ValidatingWebhookConfiguration
customresourcedefinitions           crd,crds                            apiextensions.k8s.io/v1           false        CustomResourceDefinition
apiservices                                                             apiregistration.k8s.io/v1         false        APIService
selfsubjectreviews                                                      authentication.k8s.io/v1          false        SelfSubjectReview
tokenreviews                                                            authentication.k8s.io/v1          false        TokenReview
selfsubjectaccessreviews                                                authorization.k8s.io/v1           false        SelfSubjectAccessReview
selfsubjectrulesreviews                                                 authorization.k8s.io/v1           false        SelfSubjectRulesReview
subjectaccessreviews                                                    authorization.k8s.io/v1           false        SubjectAccessReview
clusterissuers                      ciss                                cert-manager.io/v1                false        ClusterIssuer
certificatesigningrequests          csr                                 certificates.k8s.io/v1            false        CertificateSigningRequest
ciliumcidrgroups                    ccg                                 cilium.io/v2                      false        CiliumCIDRGroup
ciliumclusterwidenetworkpolicies    ccnp                                cilium.io/v2                      false        CiliumClusterwideNetworkPolicy
ciliumidentities                    ciliumid                            cilium.io/v2                      false        CiliumIdentity
ciliuml2announcementpolicies        l2announcement                      cilium.io/v2alpha1                false        CiliumL2AnnouncementPolicy
ciliumloadbalancerippools           ippools,ippool,lbippool,lbippools   cilium.io/v2                      false        CiliumLoadBalancerIPPool
ciliumnodes                         cn,ciliumn                          cilium.io/v2                      false        CiliumNode
ciliumpodippools                    cpip                                cilium.io/v2alpha1                false        CiliumPodIPPool
flowschemas                                                             flowcontrol.apiserver.k8s.io/v1   false        FlowSchema
prioritylevelconfigurations                                             flowcontrol.apiserver.k8s.io/v1   false        PriorityLevelConfiguration
gatewayclasses                      gc                                  gateway.networking.k8s.io/v1      false        GatewayClass
accesscontrolpolicies                                                   hub.traefik.io/v1alpha1           false        AccessControlPolicy
clustercleanuppolicies              ccleanpol                           kyverno.io/v2                     false        ClusterCleanupPolicy
clusterpolicies                     cpol                                kyverno.io/v1                     false        ClusterPolicy
globalcontextentries                gctxentry                           kyverno.io/v2                     false        GlobalContextEntry
nodes                                                                   metrics.k8s.io/v1beta1            false        NodeMetrics
ingressclasses                                                          networking.k8s.io/v1              false        IngressClass
ipaddresses                         ip                                  networking.k8s.io/v1              false        IPAddress
servicecidrs                                                            networking.k8s.io/v1              false        ServiceCIDR
runtimeclasses                                                          node.k8s.io/v1                    false        RuntimeClass
deletingpolicies                    dpol                                policies.kyverno.io/v1            false        DeletingPolicy
generatingpolicies                  gpol                                policies.kyverno.io/v1            false        GeneratingPolicy
imagevalidatingpolicies             ivpol                               policies.kyverno.io/v1            false        ImageValidatingPolicy
mutatingpolicies                    mpol                                policies.kyverno.io/v1            false        MutatingPolicy
validatingpolicies                  vpol                                policies.kyverno.io/v1            false        ValidatingPolicy
clusterrolebindings                                                     rbac.authorization.k8s.io/v1      false        ClusterRoleBinding
clusterroles                                                            rbac.authorization.k8s.io/v1      false        ClusterRole
clusterephemeralreports             cephr                               reports.kyverno.io/v1             false        ClusterEphemeralReport
deviceclasses                                                           resource.k8s.io/v1                false        DeviceClass
resourceslices                                                          resource.k8s.io/v1                false        ResourceSlice
priorityclasses                     pc                                  scheduling.k8s.io/v1              false        PriorityClass
csidrivers                                                              storage.k8s.io/v1                 false        CSIDriver
csinodes                                                                storage.k8s.io/v1                 false        CSINode
storageclasses                      sc                                  storage.k8s.io/v1                 false        StorageClass
volumeattachments                                                       storage.k8s.io/v1                 false        VolumeAttachment
volumeattributesclasses             vac                                 storage.k8s.io/v1                 false        VolumeAttributesClass
clusterpolicyreports                cpolr                               wgpolicyk8s.io/v1alpha2           false        ClusterPolicyReport
```

#### Namespaced

```bash
$ kubectl api-resources --namespaced=false | grep role
clusterrolebindings                                                     rbac.authorization.k8s.io/v1      false        ClusterRoleBinding
clusterroles                                                            rbac.authorization.k8s.io/v1      false        ClusterRole
```

```bash
$ kubectl api-resources --namespaced=true | grep role     
rolebindings                                       rbac.authorization.k8s.io/v1        true         RoleBinding
roles                                              rbac.authorization.k8s.io/v1        true         Role
```

#### Verbs

```bash
$ kubectl api-resources -o wide | grep pods          
pods                                po                                  v1                                  true         Pod                                create,delete,deletecollection,get,list,patch,update,watch   all
pods                                                                    metrics.k8s.io/v1beta1              true         PodMetrics                         get,list                                                     
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ 
```

### Role Binding

#### Role

**File:** role-developer.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "create"]
```

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl create namespace dev             
namespace/dev created
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl apply -f role-developer.yaml          
role.rbac.authorization.k8s.io/developer created

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl get role -n dev                                  
NAME        CREATED AT
developer   2026-03-28T08:25:51Z
```

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl get role -n dev -o yaml
apiVersion: v1
items:
- apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"rbac.authorization.k8s.io/v1","kind":"Role","metadata":{"annotations":{},"name":"developer","namespace":"dev"},"rules":[{"apiGroups":[""],"resources":["pods"],"verbs":["get","list","create"]}]}
    creationTimestamp: "2026-03-28T08:25:51Z"
    name: developer
    namespace: dev
    resourceVersion: "25528653"
    uid: 14e85a2e-0ba2-4b6b-8ed3-f30c4d279900
  rules:
  - apiGroups:
    - ""
    resources:
    - pods
    verbs:
    - get
    - list
    - create
kind: List
metadata:
  resourceVersion: ""
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl describe role developer -n dev                                                                                                                                  1 ↵
Name:         developer
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get list create]
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ 
```

#### Criando Role com kubectl

```bash
$ kubectl create role pod-reader --verb=get,list --resource=pods -n dev
```

#### Role Biding

**File:** rolebinding-developer.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer
  namespace: dev
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl apply -f rolebinding-developer.yaml 
rolebinding.rbac.authorization.k8s.io/developer created
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl get rolebindings.rbac.authorization.k8s.io -n dev
NAME        ROLE             AGE
developer   Role/developer   22s
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl get rolebindings.rbac.authorization.k8s.io -n dev -o yaml
apiVersion: v1
items:
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"rbac.authorization.k8s.io/v1","kind":"RoleBinding","metadata":{"annotations":{},"name":"developer","namespace":"dev"},"roleRef":{"apiGroup":"rbac.authorization.k8s.io","kind":"Role","name":"developer"},"subjects":[{"apiGroup":"rbac.authorization.k8s.io","kind":"User","name":"developer"}]}
    creationTimestamp: "2026-03-28T08:34:29Z"
    name: developer
    namespace: dev
    resourceVersion: "25531288"
    uid: c15a4d7b-9858-4ac9-a3ff-c9439899ec4b
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: developer
  subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: developer
kind: List
metadata:
  resourceVersion: ""
```

#### Criando RoleBinding com kubectl

```bash
$ kubectl create rolebinding rb-pod-reader --role=pod-reader --user=joao -n dev
```

## Configurando User e contexto kubectl config

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl config set-credentials developer --client-certificate developer.crt --client-key developer.key --embed-certs 
User "developer" set.
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ 
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:giropops)
╰─$ kubectl config get-users 
NAME
developer
k8s_40
k8s_60
kind-girus
```

### Criando Contexto

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl config set-context developer-aula --cluster kubernetes --namespace dev --user developer 
Context "developer-aula" created.
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ 
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl config get-contexts                                                                    
CURRENT   NAME             CLUSTER      AUTHINFO     NAMESPACE
          developer-aula   kubernetes   developer    dev
*         k8s_40           kubernetes   k8s_40       default
          k8s_60           k8s_60       k8s_60       
          kind-girus       kind-girus   kind-girus   
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$
```

### Trocar de Contexto

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl config use-context developer-aula 
Switched to context "developer-aula".
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$
```

### Testando User Aula

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl get pods                                                                                                                                                      130 ↵
No resources found in dev namespace.
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl get pods -n kube-system          

Error from server (Forbidden): pods is forbidden: User "developer" cannot list resource "pods" in API group "" in the namespace "kube-system"
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ 
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl run -ti alpine --image alpine -n giropops  -- sh    
Error from server (Forbidden): pods is forbidden: User "developer" cannot create resource "pods" in API group "" in the namespace "giropops"
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl run -ti alpine --image alpine -n dev  -- sh    
E0328 11:07:07.227780 1539789 reflector.go:204] "Failed to watch" err="pods \"alpine\" is forbidden: User \"developer\" cannot watch resource \"pods\" in API group \"\" in the namespace \"dev\"" reflector="k8s.io/client-go/tools/watch/informerwatcher.go:162" type="*v1.Pod"
E0328 11:07:08.558641 1539789 reflector.go:204] "Failed to watch" err="pods \"alpine\" is forbidden: User \"developer\" cannot watch resource \"pods\" in API group \"\" in the namespace \"dev\"" reflector="k8s.io/client-go/tools/watch/informerwatcher.go:162" type="*v1.Pod"
E0328 11:07:11.133183 1539789 reflector.go:204] "Failed to watch" err="pods \"alpine\" is forbidden: User \"developer\" cannot watch resource \"pods\" in API group \"\" in the namespace \"dev\"" reflector="k8s.io/client-go/tools/watch/informerwatcher.go:162" type="*v1.Pod"
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
warning: couldn't attach to pod/alpine, falling back to streaming logs: pods "alpine" is forbidden: User "developer" cannot create resource "pods/attach" in API group "" in the namespace "dev"
Error from server (Forbidden): pods "alpine" is forbidden: User "developer" cannot get resource "pods/log" in API group "" in the namespace "dev"
```

**Criando pod**

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl run nginx-dev --image nginx 
pod/nginx-dev created
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ 
```

### Listando permissões

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl auth can-i --list 
Resources                                       Non-Resource URLs   Resource Names   Verbs
selfsubjectreviews.authentication.k8s.io        []                  []               [create]
selfsubjectaccessreviews.authorization.k8s.io   []                  []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                  []               [create]
pods                                            []                  []               [get list create]
                                                [/api/*]            []               [get]
                                                [/api]              []               [get]
                                                [/apis/*]           []               [get]
                                                [/apis]             []               [get]
                                                [/healthz]          []               [get]
                                                [/healthz]          []               [get]
                                                [/livez]            []               [get]
                                                [/livez]            []               [get]
                                                [/openapi/*]        []               [get]
                                                [/openapi]          []               [get]
                                                [/readyz]           []               [get]
                                                [/readyz]           []               [get]
                                                [/version/]         []               [get]
                                                [/version/]         []               [get]
                                                [/version]          []               [get]
                                                [/version]          []               [get]

```

## Alterando permissões

**File:** role-developer.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "create", "delete", "watch", "update", "patch"]
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl apply -f role-developer.yaml       
Error from server (Forbidden): error when retrieving current configuration of:
Resource: "rbac.authorization.k8s.io/v1, Resource=roles", GroupVersionKind: "rbac.authorization.k8s.io/v1, Kind=Role"
Name: "developer", Namespace: "dev"
from server for: "role-developer.yaml": roles.rbac.authorization.k8s.io "developer" is forbidden: User "developer" cannot get resource "roles" in API group "rbac.authorization.k8s.io" in the namespace "dev"
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectx k8s_40                                                                                                                                                          1 ↵
✔ Switched to context "k8s_40".
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl apply -f role-developer.yaml
role.rbac.authorization.k8s.io/developer configured
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$
```

**Validando**

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectx developer-aula              
✔ Switched to context "developer-aula".
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl get pods                    
NAME        READY   STATUS    RESTARTS   AGE
alpine      1/1     Running   0          14m
nginx-dev   1/1     Running   0          13m

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl delete pods alpine                                                                                                                                              1 ↵
pod "alpine" deleted from dev namespace

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl delete pods nginx-dev 
pod "nginx-dev" deleted from dev namespace

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectl get pods             
No resources found in dev namespace.

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ 
```

## ClusterRole e ClusterRoleBinding

### Criar certificado

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ openssl genrsa -out platform.key 2048 
```

### Criar Certificate Request

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ openssl req -new -key platform.key -out platform.csr -subj "/CN=platform" 
```

```bash
$ cat platform.csr | base64 | tr -d '\n' 
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dEQ0NBVUFDQVFBd0V6RVJNQThHQTFVRUF3d0ljR3hoZEdadmNtMHdnZ0VpTUEwR0NTcUdTSWIzRFFFQgpBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRRFFmSjhCb0lVZ0F6SW9pVTdxNzlOMlZERlltZ3cxbHk3ZVM2ejU2bVBhCkhnQWdZRG9KbGV5MVN1YU4yck42WlhnUEE0NC90L0pKMkEramhsWmhNVWJMUDI4WitNRnh5U0E5cmNMMENLeVIKZjBkWmVjb0EwVGhKc0hXZk1idXBDaEZ0UG5OdkoxNUZLb2hhamlSRitINVZ4ZVhvL3lib1RyUUFNamxKdUw5QwoyT2IzQjdteTFoQVZxNTgweUpieGh0cWlOQVprdFlPQkZaeUgwMXQvUWFrNHhoeXlFV2YrTVJqK1RvRjdLQ3NQCnZZeHlkV2lhTFhaaGpBRWViUW5YQ1h1alFPMXRRWWw3Z2RrZTF6UzZ3RDY4VkJ4V0cyaUJhMXh1MFFzdTZNYWoKbU5pdjR6ZkVRWGRydHNZNVh6UEQzZWtXUmI3NVUwcUdlbDZiSkZ1SitZQ2hBZ01CQUFHZ0FEQU5CZ2txaGtpRwo5dzBCQVFzRkFBT0NBUUVBcUE0ZE1NM1EzdU1mMHE2VGUwc2dOTzZ2czRtSHh4bkVyVTlnUkkrTWNEekhBaG1UCldrbWtUd08yQnkrZmlMVVBiVUIyeUR0c2RRSWI3V09Xa3RZdlRZTXRBYXpNZ3ZqczE3MkQydlFXeHVZQXR1SHQKcDdtVWRBcEhUM3hxa1RxNjhiREovT1RJUUoxQi9ZVDFqallTazlIUjlmVkR2ZFRtNS9aRzFoelVTSHZNQlJpOApWQWl1YVV4TEg3aUVhZ0lTbzFPaDhwU2ozdUtpNXNvbThGMmRCa2NhVDVkYWVOaFJwY0hHZUcxMkhJTk9VYm5xCnRMZU05VVdHS1FTZldYa1JhWG5NRkk3S0NDSVEvYVcxdG9VWHVpelB0MGdGTHhwUUVjV25ZUlFqUTVWcGhlQSsKYVJuNkkvTTNwWlR1VmlUQzdpVS9Tc1d2Y0Nzd3FNajJqeWlSRHc9PQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K% 
```

### Certificate Signing Request

**File:** platform.yaml

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: platform
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dEQ0NBVUFDQVFBd0V6RVJNQThHQTFVRUF3d0ljR3hoZEdadmNtMHdnZ0VpTUEwR0NTcUdTSWIzRFFFQgpBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRRFFm
  ...
  RVNULS0tLS0K
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 31536000
  usages:
    - client auth
```

**Aplicar**

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|developer-aula:dev)
╰─$ kubectx k8s_40                                                                                                                                                          1 ↵
✔ Switched to context "k8s_40".
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl apply -f platform.yaml
certificatesigningrequest.certificates.k8s.io/platform created
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ 
```

**Listando**

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get csr                                                                                
NAME       AGE     SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
platform   2m37s   kubernetes.io/kube-apiserver-client   kubernetes-admin   365d                Pending
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$
```

### Aprovar certificado

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl certificate approve platform     
certificatesigningrequest.certificates.k8s.io/platform approved

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get csr                     
NAME       AGE     SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
platform   4m27s   kubernetes.io/kube-apiserver-client   kubernetes-admin   365d                Approved,Issued

```

### Obter certificado

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get csr platform -o jsonpath='{.status.certificate}' | base64 --decode > platform.crt  
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ cat platform.crt                      
-----BEGIN CERTIFICATE-----
MIIC+DCCAeCgAwIBAgIQXtHXYIe0LTdeoUIKN7qewDANBgkqhkiG9w0BAQsFADAV
...
BM/ceQR5oYA1n1sJhAxPY6hCgSzoV8ojGuot8/sQTgHrP7uF60C+XMifRkTKTKly
n3wKRxVGbunfvQOV1/gSIUvSOKwZPIXrZ0/ewEreM+4XajJG3pwIQniR56viWp79
9yxnBOMLu96WUfRHzbiTARH2Ml0ex/h2upXyYKm+wWs69XAYL1YNmeo5/vyXTcJ6
+uF8+YsRSXpENoQxZ5/ncADIForYRz3v9LcILV589tDh7UMVTXCNVFDV/Ps=
-----END CERTIFICATE-----
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$
```

### ClusterRole

**File:** clusterrole-platform.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: platform
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments"]
  verbs: ["get", "list", "create", "delete", "watch", "update", "patch"]
```

**Apply**

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl apply -f clusterrole-platform.yaml                                                   
clusterrole.rbac.authorization.k8s.io/platform created

```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get clusterrole                                                                      
NAME                                                                   CREATED AT
admin                                                                  2026-01-04T11:58:09Z
blackbox-exporter                                                      2026-01-28T18:10:17Z
...
cilium-operator                                                        2026-01-04T15:46:01Z
cluster-admin                                                          2026-01-04T11:58:09Z
edit                                                                   2026-01-04T11:58:09Z
...
node-exporter                                                          2026-01-28T18:10:18Z
platform                                                               2026-03-28T11:41:43Z
...
traefik-traefik                                                        2026-01-04T19:33:25Z
view                                                                   2026-01-04T11:58:09Z

```

### ClusterRole Binding

**File:** clusterolebinding-platform.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: platform
subjects:
- kind: User
  name: platform
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: platform
  apiGroup: rbac.authorization.k8s.io
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl apply -f clusterrolebinding-platform.yaml 
clusterrolebinding.rbac.authorization.k8s.io/platform created
```

**Listando**

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get clusterrolebindings.rbac.authorization.k8s.io 
NAME                                                            ROLE                                                                               AGE
blackbox-exporter                                               ClusterRole/blackbox-exporter                                                      58d
...
node-exporter                                                   ClusterRole/node-exporter                                                          58d
platform                                                        ClusterRole/platform                                                               40s
...
traefik-traefik                                                 ClusterRole/traefik-traefik                                                        82d
```

### Configurando config

#### 1. Credentials

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl config set-credentials platform --client-certificate platform.crt --client-key platform.key --embed-certs
User "platform" set.
```

#### 2. Context

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl config set-context platform-aula --cluster kubernetes --user platform                                    
Context "platform-aula" created.
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl config get-contexts                                                  
CURRENT   NAME             CLUSTER      AUTHINFO     NAMESPACE
          developer-aula   kubernetes   developer    dev
*         k8s_40           kubernetes   k8s_40       default
          k8s_60           k8s_60       k8s_60       
          kind-girus       kind-girus   kind-girus   
          platform-aula    kubernetes   platform   
```

#### 3. Trocar de contexto

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl config use-context platform-aula 
Switched to context "platform-aula".
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|platform-aula:N/A)
╰─$ kubectl get pods                                         
NAME                                          READY   STATUS    RESTARTS      AGE
alpine                                        1/1     Running   2 (10d ago)   10d
giropops-senhas-deployment-69576c64cd-787dh   1/1     Running   9 (48d ago)   82d
giropops-senhas-deployment-69576c64cd-h7fg8   1/1     Running   9 (48d ago)   82d
giropops-senhas-deployment-69576c64cd-tnhwr   1/1     Running   0             21d
locust-giropops-56fd678dbc-8fqd4              1/1     Running   0             21d
minion-deployment-c84f94b49-q62tz             1/1     Running   0             21d
nginx-gpu-6578b8cc5-77m2j                     1/1     Running   0             21d
nginx-gpu-6578b8cc5-gh428                     1/1     Running   0             21d
nginx-gpu-6578b8cc5-lr2bc                     1/1     Running   0             21d
nginx-gpu-6578b8cc5-nc9v5                     1/1     Running   0             21d
nginx-hpa-58f69dfffc-cdmqj                    1/1     Running   0             21d
nginx-hpa-58f69dfffc-sw42m                    1/1     Running   0             36d
nginx-server-7d9cbf7d4f-dssgw                 2/2     Running   2 (48d ago)   51d
nginx-server-7d9cbf7d4f-s774s                 2/2     Running   0             21d
nginx-server-7d9cbf7d4f-wppbw                 2/2     Running   2 (48d ago)   51d
nginx-ssl-deployment-688dcb8fd-d47dj          1/1     Running   8 (48d ago)   72d
nginx-statefulset-0                           1/1     Running   0             21d
nginx-statefulset-1                           1/1     Running   0             21d
nginx-statefulset-2                           1/1     Running   0             21d
redis-deployment-d74599fc4-bt7vf              1/1     Running   9 (48d ago)   82d
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|platform-aula:N/A)
╰─$ kubectl get pods -n kube-system

NAME                                           READY   STATUS    RESTARTS       AGE
cilium-646tz                                   1/1     Running   9 (48d ago)    82d
cilium-6k2ms                                   1/1     Running   10 (48d ago)   82d
cilium-7px2w                                   1/1     Running   10 (48d ago)   82d
cilium-cjsbn                                   1/1     Running   10 (48d ago)   82d
cilium-envoy-4x5fs                             1/1     Running   10 (48d ago)   82d
cilium-envoy-9rgdv                             1/1     Running   10 (48d ago)   82d
cilium-envoy-blbfv                             1/1     Running   10 (48d ago)   82d
cilium-envoy-km2sf                             1/1     Running   10 (48d ago)   82d
cilium-operator-694fc6cc9c-pbbp5               1/1     Running   11 (48d ago)   82d
coredns-7d764666f9-2jq4d                       1/1     Running   0              21d
coredns-7d764666f9-qlmmf                       1/1     Running   10 (48d ago)   82d
csi-nfs-controller-58f68f6dc7-8vt25            5/5     Running   45 (48d ago)   82d
csi-nfs-node-4bnvl                             3/3     Running   27 (48d ago)   82d
csi-nfs-node-nbpth                             3/3     Running   27 (48d ago)   82d
csi-nfs-node-ph9zb                             3/3     Running   27 (48d ago)   82d
csi-nfs-node-zqkwb                             3/3     Running   27 (48d ago)   82d
etcd-dk8s-control-panel-1                      1/1     Running   11 (48d ago)   82d
hubble-relay-6bc74f85f9-ssvcj                  1/1     Running   9 (48d ago)    82d
hubble-ui-7bcb645fcd-snd5m                     2/2     Running   0              21d
kube-apiserver-dk8s-control-panel-1            1/1     Running   11 (48d ago)   82d
kube-controller-manager-dk8s-control-panel-1   1/1     Running   11 (48d ago)   82d
kube-proxy-q5gmh                               1/1     Running   11 (48d ago)   82d
kube-proxy-rkxq4                               1/1     Running   11 (48d ago)   82d
kube-proxy-rsmzl                               1/1     Running   11 (48d ago)   82d
kube-proxy-z5j45                               1/1     Running   11 (48d ago)   82d
kube-scheduler-dk8s-control-panel-1            1/1     Running   11 (48d ago)   82d
metrics-server-6cb56849d5-kn6c5                1/1     Running   0              21d
```

## ServiceAccount com Token

Um **ServiceAccount** é uma identidade usada por aplicações, Pods e controladores para autenticar no Kubernetes API Server.

Diferente de um usuário humano (ex.: certificado CN=developer), o ServiceAccount representa um processo rodando dentro do cluster.

### Para que serve

- Permitir que Pods se autentiquem na API do Kubernetes.
- Aplicar permissões com RBAC de forma controlada (Role/RoleBinding ou ClusterRole/ClusterRoleBinding).
- Separar acessos por aplicação/namespace (princípio do menor privilégio).
- Integrar workloads com outros serviços do cluster que exigem autenticação.

### Token do ServiceAccount

O token é uma credencial associada ao ServiceAccount e pode ser usada para chamadas autenticadas na API.  
Em versões atuais do Kubernetes, o uso recomendado é com **tokens temporários/projetados** (TokenRequest), evitando tokens estáticos de longa duração.

### Meu ServiceAccount

**File:** service-account.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-reader
  namespace: default
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl apply -f service-account.yaml            
serviceaccount/pod-reader created

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get serviceaccounts        
NAME         AGE
default      83d
pod-reader   105s

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl describe serviceaccounts pod-reader 
Name:                pod-reader
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Events:              <none>
```

#### Criando serviceAccount com kubectl

```bash
$ kubectl create serviceaccount deploy-bot -n dev
```

### ServiceAccount Secret

**File:** service-account-secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pod-reader-sa-secret
  annotations:
    kubernetes.io/service-account.name: pod-reader
type: kubernetes.io/service-account-token
```

```bash
$ kubectl apply -f service-account-secret.yaml
secret/pod-reader-sa-secret created
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl describe secret pod-reader-sa-secret      
Name:         pod-reader-sa-secret
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: pod-reader
              kubernetes.io/service-account.uid: f0397054-9feb-4b29-83b8-91c0dbdf848f

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1107 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI...wUWhFSGsifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb
...
XuCfnjc2k38gvz0QnDLtnZVHMIw3vMy_h47XH1yyw1Nw
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get secret pod-reader-sa-secret -o yaml                                                                                                                         1 ↵
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJSG80SWhXajd1Uzh3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN
  ..
  yeUplOVRabjUwCmxpRUIzQXhmeE1rQlgzVlF2eXJPRFFCcGhxamJOa3REL3dyQVdYc2IvTDhST3NqSkRlRXZjQUtNbWpmaHFuYlgKRmtZOS9nQmIrQ29DCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  namespace: ZGVmYXVsdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkluQk1lRkZvVEhZdGVrVkVPV05ZYm1wUFJIbHBTRXA2TlhKeWFFMTBiVEJvV2pNMk5WY3dVV2hGU0dzaWZRLmV5SnBjM01pT2lKcmRXSmxjbTV
  ...
  REx0blpWSE1JdzN2TXlfaDQ3WEgxeXl3MU53
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Secret","metadata":{"annotations":{"kubernetes.io/service-account.name":"pod-reader"},"name":"pod-reader-sa-secret","namespace":"default"},"type":"kubernetes.io/service-account-token"}
    kubernetes.io/service-account.name: pod-reader
    kubernetes.io/service-account.uid: f0397054-9feb-4b29-83b8-91c0dbdf848f
  creationTimestamp: "2026-03-28T16:56:12Z"
  name: pod-reader-sa-secret
  namespace: default
  resourceVersion: "25683897"
  uid: fff514d6-0858-47a2-928d-f43e4f318f84
type: kubernetes.io/service-account-token
```

#### Criando ServiceAccount Token com kubectl

```bash
$ kubectl create token deploy-bot -n dev
```

### Obter Token

**Opção 1:**

```bash
$ kubectl describe secret pod-reader-sa-secret      
Name:         pod-reader-sa-secret
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: pod-reader
              kubernetes.io/service-account.uid: f0397054-9feb-4b29-83b8-91c0dbdf848f

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1107 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1Ni...OTqxtnZVHMIw3vMy_h47XH1yyw1Nw
```

**Opção 2:**

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get secrets pod-reader-sa-secret -o jsonpath='{.data.token}' | base64 --decode 
eyJhbGciOiJ...BdaEZZpc4tFmjZM1RX0qp9geHjFFecJycA7OSa5Fix8yGLyyB202Qnv6aB084qxXEUeSKdpKM7ljc5_gh9-uix-k6hepBWBQZmyXuCfnjc2k38gvz0QnDLtnZVHMIw3vMy_h47XH1yyw1Nw%
```

### ServiceAccount Role

**File:** role-sa.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: service-account-role
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "create", "delete", "watch", "update", "patch"]
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl apply -f role-sa.yaml                                                          
role.rbac.authorization.k8s.io/service-account-role created
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get role                                                                       
NAME                   CREATED AT
prometheus-k8s         2026-01-28T18:10:19Z
service-account-role   2026-03-28T17:09:34Z
```

### ServiceAccount RoleBinding

**File:** rolebinding-sa.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rolebinding-sa
  namespace: default
subjects:
- kind: ServiceAccount
  name: pod-reader
roleRef:
  kind: Role
  name: service-account-role
  apiGroup: rbac.authorization.k8s.io
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl apply -f rolebinding-sa.yaml       
rolebinding.rbac.authorization.k8s.io/rolebinding-sa created

```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get rolebindings.rbac.authorization.k8s.io               
NAME             ROLE                        AGE
prometheus-k8s   Role/prometheus-k8s         58d
rolebinding-sa   Role/service-account-role   33s

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get rolebindings.rbac.authorization.k8s.io rolebinding-sa 
NAME             ROLE                        AGE
rolebinding-sa   Role/service-account-role   66s
```

### Testando 

**File:** pod-sa-example.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod-reader
  namespace: default
  labels:
    app: meu-pod-reader
spec:
  serviceAccountName: pod-reader
  containers:
  - name: leitor-de-pod
    image: alpine:latest
    command: ["sleep", "infinity"]
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl apply -f pod-sa-example.yaml                             
pod/meu-pod-reader created

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get pods                                                 
NAME                                          READY   STATUS    RESTARTS      AGE
alpine                                        1/1     Running   2 (10d ago)   10d
giropops-senhas-deployment-69576c64cd-787dh   1/1     Running   9 (48d ago)   82d
giropops-senhas-deployment-69576c64cd-h7fg8   1/1     Running   9 (48d ago)   82d
giropops-senhas-deployment-69576c64cd-tnhwr   1/1     Running   0             21d
locust-giropops-56fd678dbc-8fqd4              1/1     Running   0             21d
meu-pod-reader                                1/1     Running   0             26s
minion-deployment-c84f94b49-q62tz             1/1     Running   0             21d
nginx-gpu-6578b8cc5-77m2j                     1/1     Running   0             21d
nginx-gpu-6578b8cc5-gh428                     1/1     Running   0             21d
nginx-gpu-6578b8cc5-lr2bc                     1/1     Running   0             21d
nginx-gpu-6578b8cc5-nc9v5                     1/1     Running   0             21d
nginx-hpa-58f69dfffc-cdmqj                    1/1     Running   0             21d
nginx-hpa-58f69dfffc-sw42m                    1/1     Running   0             36d
nginx-server-7d9cbf7d4f-dssgw                 2/2     Running   2 (48d ago)   51d
nginx-server-7d9cbf7d4f-s774s                 2/2     Running   0             21d
nginx-server-7d9cbf7d4f-wppbw                 2/2     Running   2 (48d ago)   51d
nginx-ssl-deployment-688dcb8fd-d47dj          1/1     Running   8 (48d ago)   72d
nginx-statefulset-0                           1/1     Running   0             21d
nginx-statefulset-1                           1/1     Running   0             21d
nginx-statefulset-2                           1/1     Running   0             21d
redis-deployment-d74599fc4-bt7vf              1/1     Running   9 (48d ago)   82d
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-18 ‹main●› (⎈|k8s_40:default)
╰─$ kubectl exec -ti meu-pod-reader -- sh    
/ # apk add curl
( 1/10) Installing brotli-libs (1.2.0-r0)
( 2/10) Installing c-ares (1.34.6-r0)
( 3/10) Installing libunistring (1.4.1-r0)
( 4/10) Installing libidn2 (2.3.8-r0)
( 5/10) Installing nghttp2-libs (1.68.0-r0)
( 6/10) Installing nghttp3 (1.13.1-r0)
( 7/10) Installing libpsl (0.21.5-r3)
( 8/10) Installing zstd-libs (1.5.7-r2)
( 9/10) Installing libcurl (8.17.0-r1)
(10/10) Installing curl (8.17.0-r1)
Executing busybox-1.37.0-r30.trigger
OK: 13.2 MiB in 26 packages
/ # ls  /var/run/secrets/kubernetes.io/serviceaccount/
ca.crt     namespace  token
/ # cat  /var/run/secrets/kubernetes.io/serviceaccount/token
eyJhbGciOi...qDTy8rCT7KoT-l1VRvz3pEQBSi_usvO7wvsBcemAzQOrGpY7kim9smOTL-20hR3-s_hw
/ #
/ # 
/ # curl -k https://kubernetes.default.svc/api/v1/namespaces/default/pods 
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:anonymous\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}/ #
/ #
/ # curl -k -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default.svc/api/v1/namespaces/default/pods 
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "25696101"
  },
  "items": [
    {
      "metadata": {
        "name": "alpine",
        "namespace": "default",
        "uid": "24baa9f6-a071-4d65-8109-9232f2cb11d7",
        "resourceVersion": "20896506",
        "generation": 1,
        "creationTimestamp": "2026-03-17T18:21:05Z",
        "labels": {
          "run": "alpine"
        },
        "managedFields": [
          {
            "manager": "kubectl-run",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2026-03-17T18:21:05Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:labels": {
                  ".": {},
                  "f:run": {}
                }
              },
              "f:spec": {
                "f:containers": {
                  "k:{\"name\":\"alpine\"}": {
                    ".": {},
                    "f:args": {},
                    "f:image": {},
                    "f:imagePullPolicy": {},
                    "f:name": {},
                    "f:resources": {},
                    "f:stdin": {},
...
            "resources": {
              "limits": {
                "cpu": "500m",
                "memory": "256Mi"
              },
              "requests": {
                "cpu": "100m",
                "memory": "128Mi"
              }
            },
            "volumeMounts": [
              {
                "name": "kube-api-access-6bq2d",
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                "readOnly": true,
                "recursiveReadOnly": "Disabled"
              }
            ],
            "user": {
              "linux": {
                "uid": 0,
                "gid": 0,
                "supplementalGroups": [
                  0
                ]
              }
            }
          }
        ],
        "qosClass": "Burstable"
      }
    }
  ]
}
```

## Validando permissoes

```bash
$ kubectl auth can-i list pods --as=joao -n dev
```

**Verificar se o joao pode apagar pods no namespace dev**

Use "kubectl auth can-i delete pods --as=joao -n dev"

```bash
$ kubectl auth can-i delete pods --as=joao -n dev
```