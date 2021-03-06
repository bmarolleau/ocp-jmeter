# Apache JMeter & Jenkins on OpenShift

## JMeter image creation

Building the JMeter image is pretty straightforward.

The build configuration and image stream can be generated by launching the following command from the container directory:

 `$ oc new-build --binary=true --name=apt-jmeter`

A binary build using the Dockerfile and context can then be launched with

 `$ oc start-build apt-jmeter --from-dir=.`

Alternatively the object definitions for build configuration and image stream available at ../openshift can be used.

## JMeter deployment on OpenShift

After a JMeter image has been created you can run your test plan in a container. Therefore an example has been provided, which is used for demonstrating the approach.
The few generic steps are described here. The ones specific to the test plan provided as example are described there.

### Test plan

A configMap is used for injecting the test plan into the image. From the directory where your jmx file is located you can create the configMap with a single command.

 `$ oc create configmap apt-jmx --from-file=jms-code.jmx --from-file=pain_samples.csv --from-file=pain_template.xml`

A label can also be added to the config

 `$ oc label configMap apt-jmx job-name=apt-jmeter`

This configMap has been exported and stored for reuse by Jenkins (it could also be recreated each time from files)

 `$ oc export configMap apt-jmx > ../openshift/apt-jmx-cm.yaml`

### JMeter properties

https://jmeter.apache.org/usermanual/properties_reference.html[Prorperties] allow to customize JMeter. Interesting for our purpose is for instance the possibility to customize the result file. They are two ways of setting properties, the first one is to inject a user.properties file. The second one is to pass them as command line argument with -J, example: -Jproperty_example=property_value. The values then overwrite what has been defined in jmeter.properties or reportgenerator.properties. To facilitate this second approach the run script transforms any environment variable with the J_ prefix to a command line argument. For instance the environment variable J_myprop will be passed as the parameter -Jmyprop. 


## JMeter Run

As we want our test plan to be run and terminate at the end a job is better suited for the purpose than a deployment configuration. A template containing the job definition may also be used to pass parameters. An example is available in this directory. The template can be processed with this command to create a new job.

 `$ oc process -f apt-jmeter-job-tm.yaml | oc create -f -`

If you want to replace it with a new version you can use the following.

 `$ oc process -f apt-jmeter-job-tm.yaml | oc replace --force -f -`

It is possible to pass the namespace and name of configMap with the JMeter test suite as parameter.

It may also be useful to first create the template in OpenShift, so that it can easily be called from Jenkins for instance:

`$ oc create -f apt-jmeter-job-tm.yaml`

Another template apt-jmeter-job-persistent-tm.yaml allows to mount a persistent volume instead of emptyDir for storing the test results. An additional parameter allows the specification of the PVC name.
This is useful when the test results are to be read after the job has terminated, from Jenkins for instance.

## Jenkins Instructions

Refer to the jenkins [folder](./jenkins).