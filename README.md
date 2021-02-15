# compliance-operator
How to that walks through installing and running the [compliance-operator](https://github.com/openshift/compliance-operator) on [OpenShift version 4.6](https://docs.openshift.com/container-platform/4.6/welcome/index.html) on OpenShift version 4.6 

Use the [Walk-Through](#walk-through) if you are in a hurry.

## Table Of Contents
- [Installation](#installation)
  - [Prerequisites](#prerequisites)
    - [Assumptions](#assumptions)
    - [Verify Operator Availability](#verify-operator-availability)
    - [Verify Install Modes and Channels](#verify-install-modes-and-channels)
  - [Create Namespace](#create-namespace)
  - [View Catalog Source](#view-catalog-source)
  - [Create Operator Group](#create-operator-group)
  - [Create Subscription](#create-subscription)
  - [View Deployment](#view-deployment)
  - [View Profile](#view-profile)
  - [View Profile Bundle](#view-profile-bundle)
- [Create Scans](#create-scans)
  - [Create Compliance Suite](#create-compliance-suite)
  - [View Compliance Scan](#view-compliance-scan)
  - [View Scan Settings](#view-scan-settings)
  - [View Scan Setting Binding](#view-scan-setting-binding)
- [Apply Compliance Remediation](#apply-compliance-remediation)
- [Automated Walk-through](#automated-walkthrough)
- [References](#references)

  
## Installation
The [compliance-operator](https://github.com/openshift/compliance-operator) is installable on OpenShift by an account with cluster-admin permissions. See [Adding Operators to a cluster](https://docs.openshift.com/container-platform/4.6/operators/admin/olm-adding-operators-to-cluster.html) for generalized operator installation instructions.

### Prerequisites
#### Assumptions
* Access to an OpenShift Container Platform cluster using an account with `cluster-admin` permissions.

* Assume the ` oc ` command is installed on your local system.

* Assume the environment variable `NAMESPACE` has been exported on your local system.

#### Verify Operator Availability
To ensure that the [compliance-operator](https://github.com/openshift/compliance-operator) is available to the cluster verify the [compliance-operator](https://github.com/openshift/compliance-operator) using the following command:
```bash
oc get packagemanifests -n openshift-marketplace | grep compliance-operator
``` 

#### Verify Install Modes and Channels
Verify the supported install modes and channels to see namespaces tenancy supported by the operator using the following command:
```bash
oc describe packagemanifests compliance-operator -n openshift-marketplace
```

### Create Namespace
We will be creating a new namespace, `${NAMESPACE}`, to deploy the [compliance-operator](https://github.com/openshift/compliance-operator).

Create the namespace using the following command:
```bash
oc new-project ${NAMESPACE}
```

### View Catalog Source
A catalog source, defined by a [CatalogSource](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/catalogsource-operators-coreos-com-v1alpha1.html) object is a repository of [Cluster Service Versions](https://docs.openshift.com/container-platform/4.6/operators/operator_sdk/osdk-generating-csvs.html), [Custom Resource Definitions](https://docs.openshift.com/container-platform/4.6/operators/understanding/crds/crd-extending-api-with-crds.html#crd-extending-api-with-crds), and operator packages. For this how-to we will be using the Red Hat supported version `4.6` of the operator. 

View the `redhat-marketplace` [CatalogSource](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/catalogsource-operators-coreos-com-v1alpha1.html) object in the `openshift-marketplace` namespace using the following command:
```bash
oc describe catalogsource redhat-marketplace -n openshift-marketplace | less
```

### Create Operator Group
An Operator group, defined by an [OperatorGroup](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/operatorgroup-operators-coreos-com-v1.html)  object, selects target namespaces in which to generate required RBAC access for all Operators in the same namespace as the Operator group.

The namespace to which you subscribe the Operator must have an [OperatorGroup](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/operatorgroup-operators-coreos-com-v1.html) that matches the install mode of the Operator. We will be installing the [compliance-operator](https://github.com/openshift/compliance-operator) in the `${NAMESPACE}` namespace.

Create a new [OperatorGroup](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/operatorgroup-operators-coreos-com-v1.html) object using the following command:
```bash
oc apply -n ${NAMESPACE} -f- <<EOF
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: ${NAMESPACE}-compliance-operator
spec:
  targetNamespaces:
  - ${NAMESPACE}
EOF
```

List the [OperatorGroup](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/operatorgroup-operators-coreos-com-v1.html) object using the following command:
```bash
oc get OperatorGroup -n ${NAMESPACE} 
```

Inspect the [OperatorGroup](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/operatorgroup-operators-coreos-com-v1.html) object using the following command:
```bash
oc describe OperatorGroup -n ${NAMESPACE} ${NAMESPACE}-compliance-operator | less
```

### Create Subscription 
The [Subscription](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/subscription-operators-coreos-com-v1alpha1.html) object keep operators up to date by tracking changes to [Catalogs](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/catalogsource-operators-coreos-com-v1alpha1.html).

Create [Subscription](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/subscription-operators-coreos-com-v1alpha1.html) object using the following command:
```bash
oc apply -n ${NAMESPACE} -f- <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ${NAMESPACE}-subscription
  namespace: ${NAMESPACE}
spec:
  channel: '4.6'
  installPlanApproval: Automatic
  name: compliance-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  startingCSV: compliance-operator.v0.1.17  
EOF
```

List the [Subscription](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/subscription-operators-coreos-com-v1alpha1.html) object using the following command:
```bash
oc get subscription ${NAMESPACE}-subscription -n ${NAMESPACE}
```

Inspect the [Subscription](https://docs.openshift.com/container-platform/4.6/rest_api/operatorhub_apis/subscription-operators-coreos-com-v1alpha1.html) object using the following command:
```bash
oc describe subscription ${NAMESPACE}-subscription -n ${NAMESPACE} | less
```

### View Deployment
At this point, [OpenShift Lifecycle Manager](https://docs.openshift.com/container-platform/4.6/operators/understanding/olm/olm-understanding-olm.html) is now aware of the selected Operator. A cluster service version (CSV) for the Operator should appear in the target namespace, and APIs provided by the Operator should be available for creation.

List the [Cluster Service Version](https://docs.openshift.com/container-platform/4.6/operators/operator_sdk/osdk-generating-csvs.html) version using the following command:

```bash
oc get clusterserviceversion -n ${NAMESPACE}
```

List the `Install Plan` using the following command:
```bash
oc get installplan -n ${NAMESPACE} 
```

At this point, the operator should be up and running.

List the `Deployment` using the following command:
```bash
oc get deploy -n ${NAMESPACE}
```

List the Running `Pods` using the following command:
```bash
oc get pods -n ${NAMESPACE}
```

### View Profile Bundle
OpenSCAP content for consumption by the Compliance Operator is distributed as container images. In order to make it easier for users to discover what profiles a container image ships, a [ProfileBundle](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-profilebundle-object) object can be created, which the Compliance Operator then parses and creates a [Profile](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-profile-object) object for each profile in the bundle. 

List the [ProfileBundle](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-profilebundle-object) using the following command:
```bash
oc get profilebundle -n ${NAMESPACE}
```

### View Profile
The [Profile](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-profile-object) objects are never created manually, but rather based on a
[ProfileBundle](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-profilebundle-object) object, typically one [ProfileBundle](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-profilebundle-object) would result in
several [Profiles](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-profile-object). The [Profile](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-profile-object) object contains parsed out details about
an OpenSCAP profile such as its XCCDF identifier, what kind of checks the
profile contains (node vs platform) and for what system or platform.

List the out-of-the-box [Profile](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-profile-object) objects that are part of the [compliance-operator](https://github.com/openshift/compliance-operator) installation and can be listed using the following command:
```bash
oc get -n ${NAMESPACE} profiles.compliance
```

## Create Scans 
After we have installed the [compliance-operator](https://github.com/openshift/compliance-operator) in the `${NAMESPACE}` namespace we are ready to start creating scans.

### Create Compliance Suite
[ComplianceSuite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object) is a collection of [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) objects, each of which describes a scan. 

The [ComplianceSuite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object) in the background will create as many [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) objects as you specify in the `scans` field. The fields will be described in the section referring to [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) objects.

Create a new [ComplianceSuite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object) object:

```bash
oc apply -n ${NAMESPACE} -f - <<EOF
apiVersion: compliance.openshift.io/v1alpha1
kind: ComplianceSuite
metadata:
  name: ${NAMESPACE}-compliance-suite
spec:
  autoApplyRemediations: false
  schedule: '0 1 * * *'
  scans:
    - name: ${NAMESPACE}-rhcos4-scan
      scanType: Node
      # This compliance profile reflects the core set of Moderate-Impact Baseline
      # configuration settings for deployment of Red Hat Enterprise
      # Linux CoreOS into U.S. Defense, Intelligence, and Civilian agencies.
      profile: xccdf_org.ssgproject.content_profile_moderate
      # Content file that contains checks
      # https://atopathways.redhatgov.io/compliance-as-code/scap/ssg-rhcos4-ds.xml
      content: ssg-rhcos4-ds.xml
      nodeSelector:
        node-role.kubernetes.io/worker: ''
EOF
```

At this point the operator reconciles the [ComplianceSuite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object) custom resource, we can use this to track the progress of our scans using the following command:

Watch the [ComplianceSuite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object) objects:
```bash
oc get -n ${NAMESPACE} compliancesuites -w
```

### View Compliance Scan
Similarly to `Pods` in Kubernetes, a [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) is the base object that the compliance-operator introduces. Also similarly to `Pods`, you normally don't want to create a [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) object directly, and would instead want a [ComplianceSuite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object) to manage it.

When a [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) is created by a [ComplianceSuite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object), the [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) is owned by it. Deleting a [ComplianceSuite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object) object will result in deleting all the [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) objects that it created.

Once a [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) has finished running it will generate the results as Custom Resources of the [ComplianceCheckResult](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancecheckresult-object) kind. However, the raw results in ARF format will also be available. These will be stored in a Persistent Volume which has a Persistent Volume Claim associated that has the same name as the scan.

Note that [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) objects will generate events which you can fetch programmatically. 

View [ComplianceScan](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancescan-object) object:
```bash
oc get compliancescan -n ${NAMESPACE} ${NAMESPACE}-rhcos4-scan
```

View the events for the scan called `${NAMESPACE}-rhcos4-scan` use the following command:
```bash
oc get events --field-selector involvedObject.kind=ComplianceScan,involvedObject.name=${NAMESPACE}-rhcos4-scan
```

### View Scan Settings
[ScanSetting](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-scansetting-and-scansettingbinding-objects) fall into two basic categories - platform and node. The platform scans are for the cluster itself, in the listing above they're the ocp4-* scans, while the purpose of the node scans is to scan the actual cluster nodes. All the rhcos4-* profiles above can be used to create node scans.

List the [ScanSetting](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-scansetting-and-scansettingbinding-objects) object:
```bash
oc get scansetting -n ${NAMESPACE}
```

View the [ScanSetting](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-scansetting-and-scansettingbinding-objects) object:
```bash
oc get scansetting -n ${NAMESPACE} -oyaml | less
```

### View Scan Setting Binding
Before using one, you will need to configure how the scans will run. We can do this with the [ScanSetting](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-scansetting-and-scansettingbinding-objects) custom resource.

To run rhcos4-moderate profile, the system will create a [ScanSettingBinding](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-scansetting-and-scansettingbinding-objects) object.

List the [ScanSettingBinding](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-scansetting-and-scansettingbinding-objects) object using the following command:
```bash
oc get scansettingbinding -n ${NAMESPACE} 
```

View the [ScanSettingBinding](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-scansetting-and-scansettingbinding-objects) object using the following command:
```bash
oc get scansettingbinding -n ${NAMESPACE} -o yaml | less
```

The [ComplianceSuite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object) object then creates scan pods that run on each node in the cluster. The scan pods execute openscap-chroot on every node and eventually report the results. The scan takes several minutes to complete.

List the scan pods of you're interested in seeing the individual pods using the following command:
```bash
oc get -n ${NAMESPACE} pods -w
```

To get all the [ComplianceCheckResult](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancecheckresult-object) results from the [ComplianceSuite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object) object by using the label.

View [ComplianceCheckResult](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancecheckresult-object) using the following command:
```bash
oc get compliancecheckresults.compliance.openshift.io -n ${NAMESPACE} | less
```

### Apply Compliance Remediation
When the scan is done, the operator changes the state of the ComplianceSuite object to "Done" and all the pods are transitioned to the "Completed" state. You can then check the [ComplianceRemediation](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-complianceremediation-object) that were found with:

List [ComplianceRemediation](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-complianceremediation-object) using the following command:
```bash
oc get -n ${NAMESPACE} complianceremediations
```

Apply remediation by setting `apply` item to `true` for example the `<scan_name>-sysctl-net-ipv4-conf-all-accept-redirects`  remediation using one of the following commands:
```bash
oc edit -n ${NAMESPACE} complianceremediation/<scan_name>-sysctl-net-ipv4-conf-all-accept-redirects
```

```bash
oc patch complianceremediations/<scan_name>-sysctl-net-ipv4-conf-all-accept-redirects --patch '{"spec":{"apply":true}}' --type=merge
```

The [compliance-operator](https://github.com/openshift/compliance-operator) then aggregates all applied remediations and creates a `MachineConfig` object per scan. This `MachineConfig` object is rendered to a `MachinePool` and the `MachineConfigDeamon` running on nodes in that pool pushes the configuration to the nodes and reboots the nodes.

You can watch the node status with using the following command:
```bash
oc get nodes -w
```

Once the nodes reboot, you might want to run another [Compliance Suite](https://github.com/openshift/compliance-operator/blob/master/doc/crds.md#the-compliancesuite-object) to ensure that the remediation that you applied previously was no longer found.

## Walk Through
Please note that the walk through requires `Curl` and `Pipe Viewer` to be installed on your system. 

1. Download demo-magic script using the following commands:
```bash
curl https://raw.githubusercontent.com/paxtonhare/demo-magic/master/demo-magic.sh \
     --output demo-magic.sh
```

2. Download walk-through script using the following command:
```bash
curl https://raw.githubusercontent.com/ocp4opsandsecurity/compliance-operator/main/compliance-operator-walk-through.sh \
     --output walk-through.sh
```

3. Execute the walk-through using the following command:
```bash
sh ./compliance-operator-walk-through.sh
```

## References
[Compliance Operator Git Repository](https://github.com/openshift/compliance-operator)

[Compliance Operator OpenShift Documentation](https://docs.openshift.com/container-platform/4.6/security/compliance_operator/compliance-operator-understanding.html)

[Demo Magic](https://github.com/paxtonhare/demo-magic)


