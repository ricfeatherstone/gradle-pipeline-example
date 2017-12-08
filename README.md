
# OpenShift Pipelines Example

An OpenShift Pipelines example extending 
[this](https://github.com/OpenShiftDemos/openshift-cd-demo) CICD example

## Import Image Streams

```
oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/jboss-image-streams.json \
  -n openshift \
  --as=system:admin
```

## Project Creation

```
oc new-project dev --display-name="Example - Dev"
oc new-project test --display-name="Example - Test"
oc new-project pre-prod --display-name="Example - Pre-Prod"
oc new-project prod --display-name="Example - Prod"
oc new-project cicd --display-name="Example - CI/CD"
```

## Setup Jenkins Access

```
for i in dev test pre-prod prod
do
    oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n $i
done
```


## Deploy Jenkins with OpenShift Client Plugin

```
oc project cicd
oc new-app jenkins-ephemeral \
    -p JENKINS_IMAGE_STREAM_TAG=jenkins:2 \
    -p MEMORY_LIMIT=1Gi \
    -e OPENSHIFT_JENKINS_JVM_ARCH=x86_64 \
    -e INSTALL_PLUGINS=openshift-client:1.0.3
```

## Pipeline Creation
```
oc project cicd
for i in e2e adhoc
do
    oc new-build https://github.com/ricfeatherstone/openshift-pipelines-example.git --context-dir="pipelines/$i" --strategy=pipeline --name=$i-pipeline
done
oc delete build adhoc-pipeline-1
```


## Cleanup Deployments

```
for i in cicd dev test pre-prod prod
do
    oc project $i
    oc delete all --all
    oc delete project $i
done
```
