
# OpenShift Pipelines Example

An OpenShift Pipelines example.  

## Import Image Streams

```
oc login -u system:admin
oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/jboss-image-streams.json -n openshift
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
oc new-app jenkins-persistent \
    -p JENKINS_IMAGE_STREAM_TAG=jenkins:2 \
    -p MEMORY_LIMIT=2Gi \
    -e OPENSHIFT_JENKINS_JVM_ARCH=x86_64 \
    -e INSTALL_PLUGINS=openshift-client:1.0.2
```

## Pipeline Creation
```
oc project cicd
for i in e2e adhoc
do
    oc new-build https://github.com/ricfeatherstone/openshift-pipelines-example.git --context-dir="pipelines/$i" --strategy=pipeline --name=$i-pipeline
    oc delete build adhoc-pipeline-1
done
```


## Cleanup Deployments

```
for i in cicd dev test pre-prod prod
do
    oc project $i
    oc delete all --all
    oc delete project $i
done
oc delete sa jenkins
oc delete rolebinding jenkins_edit
oc delete pvc jenkins
```
