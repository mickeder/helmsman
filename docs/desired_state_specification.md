---
version: v0.1.2
---

# Helmsman desired state specification

This document describes the specification for how to write your Helm charts desired state file. The desired state file consists of:

- [Metadata](#metadata) [Optional] -- metadata for any human reader of the desired state file.
- [Certificates](#certificates) [Optional] -- only needed when you want Helmsman to connect kubectl to your cluster for you.
- [Settings](#settings) -- data about your k8s cluster. 
- [Namespaces](#namespaces) -- defines the namespaces where you want your Helm charts to be deployed.
- [Helm Repos](#helm-repos) -- defines the repos where you want to get Helm charts from.
- [Apps](#apps) -- defines the applications/charts you want to manage in your cluster.

## Metadata

Optional : Yes.

Synopsis: Metadata is used for the human reader of the desired state file. While it is optional, we recommend having a maintainer and scope/cluster metadata.

Options: 
- you can define any key/value pairs.

Example: 

```
[metadata]
scope = "cluster foo"
maintainer = "k8s-admin"
```

## Certificates

Optional : Yes, if you don't want Helmsman to connect kubectl to your cluster for you.

Synopsis: defines where to find the certifactions needed for connecting kubectl to a k8s cluster.

Options: 
- caCrt : a valid s3 bucket to a CRT file.
- caKey : a valid s3 bucket to a key file.

Example: 

```
[certificates]
caCrt = "s3://mybucket/ca.crt" 
caKey = "s3://mybucket/ca.key" 
```

## Settings

Optional : No.

Synopsis: provides data about your k8s cluster.

Options: 
- kubeContext : this is always required and defines what context to use in kubectl. Helmsman will try connect to this context first, if it does not exist, it will try to create it (i.e. connect to a k8s cluster) using the options below.

The following options can be skipped if your kubectl context is already created and you don't want Helmsman to connect kubectl to your cluster for you. When using Helmsman in CI pipeline, these details are required to connect to your cluster everytime the pipeline is executed.

- username   : the username to be used for kubectl credentials.
- password   : an environment variable name (starting with `$`) where your password is stored. Get the password from your k8s admin or consult k8s docs on how to get it. 
- clusterURI : the URI for your cluster API.

Example: 

```
[settings]
kubeContext = "minikube" 
# username = "admin"
# password = "$PASSWORD" 
# clusterURI = "https://192.168.99.100:8443" 
```

## Namespaces

Optional : No.

Synopsis: defines the namespaces to be used/created in your k8s cluster. You can add as many namespaces as you like.
If a namespaces does not already exist, Helmsman will be created.

Options: 
- you can define any key/value pairs.

Example: 

```
[namespaces]
staging = "staging" 
production = "default"
```

## Helm Repos

Optional : No.

Purpose: defines the Helm repos where your charts can be found. You can add as many repos as you like. Public repos do not require authentication. Private repos require authentication. 

> AS of version v0.1.2, only AWS S3 buckets can be used for private repos (using the [Helm S3 plugin](https://github.com/hypnoglow/helm-s3)). For that you need to have valid AWS access keys in your environment variables. See [here](https://github.com/hypnoglow/helm-s3#note-on-aws-authentication) for more details.

Options: 
- you can define any key/value pairs.

Example: 

```
[helmRepos]
stable = "https://kubernetes-charts.storage.googleapis.com"
incubator = "http://storage.googleapis.com/kubernetes-charts-incubator"
myrepo = "s3://my-private-repo/charts"
```

## Apps

Optional : Yes.

Synopsis: defines the releases (instances of Helm charts) you would like to manage in your k8s cluster. 

Releases must have unique names which are defined under `apps`. Example: in `[apps.jenkins]`, the release name will be `jenkins` and it should be unique in your cluster. 

Options: 
- name        : the Helm release name. Releases must have unique names within a cluster.
- description : a release metadata for human readers.
- env         : the namespace where the release should be deployed. The namespace should map to one of the ones defined in [namespaces](#namespaces).  
- enabled     : describes the required state of the release (true for enabled, false for disabled). Once a release is deployed, you can change it to false if you want to delete this app release [empty = flase].
- chart       : the chart name. It should contain the repo name as well. Example: repoName/chartName. Changing the chart name means delete and reinstall this release using the new Chart.
- version     : the chart version.
- valuesFile  : a valid path to custom Helm values.yaml file. File extension must be `yaml`. Leaving it empty uses the default chart values.
- purge       : defines whether to use the Helm purge flag wgen deleting the release. (true/false)
- test        : defines whether to run the chart tests whenever the release is installed/upgraded/rolledback.

Example: 

> Whitespace does not matter in TOML files. You could use whatever indentation style you prefer for readability.

```
[apps]

    [apps.jenkins]
    name = "jenkins" 
    description = "jenkins"
    env = "staging" 
    enabled = true 
    chart = "stable/jenkins" 
    version = "0.9.0"
    valuesFile = "" 
    purge = false 
    test = true 
```
