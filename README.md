# JMeter on Openshift

This repository helps you to deploy the latest version of JMeter on Openshift taking advantage of RH UBI images and Openshift resources.

## Introduction

The https://jmeter.apache.org/[Apache JMeter™] application is open source software, a 100% pure Java application designed to load test functional behavior and measure performance. It was originally designed for testing Web Applications but has since expanded to other test functions.

## JMeter on your laptop: Building the container image

In order to perform some testing locally, you may want to build the JMeter image directly on your laptop.

```bash
podman build -t localhost/jmeter:latest ./container-image
```

## JMeter On Your Laptop: Running the image

```bash
podman run --rm -it --name jmeter localhost/jmeter:latest
```

## JMeter on Openshift: Building The Container Image

If Your Have A Disconnected Openshift Installation, You Will Also Require To Build Your Application Locally.

```bash
APP_NAMESPACE="jmeter"
GIT_REPOSITORY="https://github.com/alvarolop/jmeter-on-ocp.git"
oc new-project ${APP_NAMESPACE} --display-name="JMeter Testing" --description="This Namespace Contains Resources To Deploy JMeter"

oc process -f templates/jmeter-bc.yaml -p APP_NAMESPACE=$APP_NAMESPACE -p GIT_REPOSITORY=$GIT_REPOSITORY | oc apply -f -
```

## JMeter on Openshift: Deploying the image


First, create a ConfigMap that will store your application configuration:

```bash
JMETER_TEST="example"
oc -n "${APP_NAMESPACE}"  create configmap jmeter-config --from-file=${JMETER_TEST}.jmx=tests/${JMETER_TEST}/jmeter-test-plan.jmx --from-file=config.properties=tests/${JMETER_TEST}/config-k8s.properties
```

```bash
oc process -f templates/jmeter-dc.yaml \
    -p APP_NAMESPACE=$APP_NAMESPACE \
    -p TEST_NAME=$JMETER_TEST | oc apply -f -
```

## JMeter on Openshift: Obtaining JMeter reports

```bash
# Get JMeter pod name
JMETER_POD=$(oc get pods -l app=jmeter -n $APP_NAMESPACE --template='{{(index .items 0).metadata.name}}')
NOW=$(date +"%Y-%m-%d_%H-%M-%S")
mkdir ./results/$NOW-$TEST
oc rsync $JMETER_POD:/opt/jmeter/results/$TEST-report/ ./results/$NOW-$TEST
```

## Useful links

* Official [Apache JMeter documentation](https://jmeter.apache.org/usermanual/get-started.html)
* Medium: [Start Learning JMeter With Sample Test Cases](https://medium.com/chaya-thilakumara/start-learning-jmeter-with-sample-test-cases-2dc2a4963b62)

## Annex: Running jmeter in your laptop

```bash
JMETER_BASE=$(pwd)
TEST_PLAN=test-03
NOW=$(date +"%Y-%m-%d_%H-%M-%S"); jmeter -n -p "$JMETER_BASE/tests/${TEST_PLAN}/config.properties" -t "$JMETER_BASE/tests/${TEST_PLAN}/jmeter-test-plan.jmx" -l "$JMETER_BASE/results/${NOW}-${TEST_PLAN}.jtl" -e -o "$JMETER_BASE/results/${NOW}-${TEST_PLAN}-report"; cp -r $JMETER_BASE/tests/${TEST_PLAN}/ $JMETER_BASE/results/${NOW}-${TEST_PLAN}-report
```
