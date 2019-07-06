# CloudBees Core Teams GitOps

## Premise

## Team Recipe

## Teams

## Create Team in Alternative Namespace

### Setup OPS Team

#### Steps

* create new namespace for ops `cb-ops`
    * namespace `cb-ops`
    * resource quota
* create service account for `cb-ops`
    * role for managing pods
    * role for creating namespaces
    * role bindings for both
* allow Operations Center (OC) to create `cb-ops` team in its own namespace
    * create **Role** for creating masters
    * create **RoleBinding** for `cjoc` service account in OC's namespace
* configure Operations Center to use `cb-ops` namespace
* create new team master (`cb-ops`) in the alternative namespace
* let `cb-ops` master create new namespaces

!!!! Have to set Requests/Limits, else ResourceQuota denies Pod !!!!
!!!! Service Account Invalid? !!!!

### Create Alternative Namespace


### Configure Operations Center to use Alternative Namespace

#### xml config

```xml
<?xml version='1.1' encoding='UTF-8'?>
<com.cloudbees.masterprovisioning.kubernetes.KubernetesMasterProvisioning_-DescriptorImpl plugin="master-provisioning-kubernetes@2.2.6">
  <clusterEndpoints>
    <com.cloudbees.masterprovisioning.kubernetes.KubernetesClusterEndpoint>
      <id>default</id>
      <name>kubernetes</name>
      <skipTlsVerify>false</skipTlsVerify>
      <namespace>cb-ops</namespace>
    </com.cloudbees.masterprovisioning.kubernetes.KubernetesClusterEndpoint>
  </clusterEndpoints>
  <javaOptions>-XshowSettings:vm -XX:MaxRAMFraction=1 -XX:+AlwaysPreTouch -XX:+UseG1GC -XX:+ExplicitGCInvokesConcurrent -XX:+ParallelRefProcEnabled -XX:+UseStringDeduplication -Dhudson.slaves.NodeProvisioner.initialDelay=0</javaOptions>
  <jenkinsOptions></jenkinsOptions>
  <envVars></envVars>
  <systemProperties></systemProperties>
  <jenkinsUrl>http://cjoc.cloudbees-core.svc.cluster.local/cjoc</jenkinsUrl>
  <disk>50</disk>
  <memory>3072</memory>
  <ratio>0.7</ratio>
  <cpus>1.0</cpus>
  <masterUrlPattern></masterUrlPattern>
  <terminationGracePeriodSeconds>1200</terminationGracePeriodSeconds>
  <livenessInitialDelaySeconds>300</livenessInitialDelaySeconds>
  <livenessPeriodSeconds>10</livenessPeriodSeconds>
  <livenessTimeoutSeconds>10</livenessTimeoutSeconds>
  <fsGroup>1000</fsGroup>
  <nodeSelectors></nodeSelectors>
  <yaml></yaml>
</com.cloudbees.masterprovisioning.kubernetes.KubernetesMasterProvisioning_-DescriptorImpl>
```

#### Groovy

```groovy
import com.cloudbees.masterprovisioning.kubernetes.KubernetesMasterProvisioning
import com.cloudbees.masterprovisioning.kubernetes.KubernetesClusterEndpoint

println "=== KubernetesMasterProvisioning Configuration - start"

println "== Retrieving main configuration"
def descriptor = Jenkins.getInstance().getInjector().getInstance(KubernetesMasterProvisioning.DescriptorImpl.class)
def namespace = 'cloudbees-core'

// def kubernetesMasterProvisioning.setNamespace(namespace)

def currentKubernetesClusterEndpoint =  descriptor.getClusterEndpoints().get(0)
println "= Found current endpoint"
println "= " + currentKubernetesClusterEndpoint.toString()
def id = currentKubernetesClusterEndpoint.getId()
def name = currentKubernetesClusterEndpoint.getName()
def url = currentKubernetesClusterEndpoint.getUrl()
def credentialsId = currentKubernetesClusterEndpoint.getCredentialsId()

println "== Setting Namspace to " + namespace
def updatedKubernetesClusterEndpoint = new KubernetesClusterEndpoint(id, name, url, credentialsId, namespace)
def clusterEndpoints = new ArrayList<KubernetesClusterEndpoint>()
clusterEndpoints.add(updatedKubernetesClusterEndpoint)
descriptor.setClusterEndpoints(clusterEndpoints)

println "== Saving Jenkins configuration"
descriptor.save()

println "=== KubernetesMasterProvisioning Configuration - finish"
```
