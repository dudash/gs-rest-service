# OpenShift Examples - SpringBoot REST Service
Example SpringBoot app running on OpenShift

Requires the [OpenShift Java S2I builder image](https://access.redhat.com/containers/#/registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift)

To build/run it:
`> oc new-app redhat-openjdk18-openshift~https://github.com/dudash/openshiftexamples-springbootrestservice.git --context-dir='complete' --name=springbootrest`

The above command uses the `redhat-openjdk18-openshift` container image from your OpenShift cluster as a builder.  It fetches the code from the referenced github URI, then it creates all the resources needed for Kubernetes and OpenShift, then it does a springboot specific build, then it containerizes the build results, and then it deploys that built image into your cluster.

Results will look something like:
```
--> Found image 81bc084 (3 weeks old) in image stream "openshift/redhat-openjdk18-openshift" under tag "latest" for "redhat-openjdk18-openshift"

    Java Applications 
    ----------------- 
    Platform for building and running plain Java applications (fat-jar and flat classpath)

    Tags: builder, java

    * A source build using source code from https://github.com/dudash/openshiftexamples-springbootrestservice.git will be created
      * The resulting image will be pushed to image stream "springbootrest:latest"
      * Use 'start-build' to trigger a new build
    * This image will be deployed in deployment config "springbootrest"
    * Ports 8080/tcp, 8443/tcp, 8778/tcp will be load balanced by service "springbootrest"
      * Other containers can access this service through the hostname "springbootrest"

--> Creating resources ...
    imagestream "springbootrest" created
    buildconfig "springbootrest" created
    deploymentconfig "springbootrest" created
    service "springbootrest" created
--> Success
    Build scheduled, use 'oc logs -f bc/springbootrest' to track its progress.
    Run 'oc status' to view your app.
 ```

To make it accessible (expose it):
`> oc expose service springbootrest --path="/greeting"`

![Screenshot](./.screens/2017-07-17.png?raw=true)


## Extra Credit - give it some health checks
1. Edit the `complete/pom.xml` file and add the following into your dependencies section:
```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
```

2. Kick off a rebuild of the source in the console via "Start Build" or in the CLI via `oc start-build springbootrest`

Once the build finishes, [Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html) will be enabled and we hook up those endpoins for health checks in OpenShift.

3. Hook them up in the console by going to the springbootrest Deployment, click the Configuration tab, click "Add Health Checks" (map liveness to /status with 45 sec deplay and readiness to /health with a 10 sec delay)

OR use the CLI with:
* ```oc set probe dc/springbootrest --readiness --get-url=http://:8080/health --initial-delay-seconds=10```
* ```oc set probe dc/springbootrest --liveness --get-url=http://:8080/status --initial-delay-seconds=45```

And notice that since we a doing a rolling deployment type, the old app won't go away until the new changes are ready and deployed.

-----

To see the original README [click here](README-orig.adoc)
