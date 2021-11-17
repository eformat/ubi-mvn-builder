## ubi-mvn-builder

![images/ubi-mvn-builder.png](images/ubi-mvn-builder.png)

> 👷👷👷 Build Toolchain:
>   - Quarkus (native and fast-jar)
>   - OpenJDK 17, maven 3.8.1, Mandrel
>   - UBI 8.5 Minimal
>
> ⚡⚡⚡ Runtime:
>   - UBI 8.5 Minimal
>   - Your Application

1. Create the base s2i core build image and push to remote repo for reuse across clusters.

    ```bash
    podman build --squash -t quay.io/eformat/ubi-mvn-builder:latest -f builder/Dockerfile
    podman push quay.io/eformat/ubi-mvn-builder:latest
    ```

2. Create the middleware runtimes and push to remote repo for reuse across clusters.

    ```bash
    podman build --squash -t quay.io/eformat/ubi-mvn-builder-middle-jvm:latest -f middleware/Dockerfile.jvm
    podman push quay.io/eformat/ubi-mvn-builder-middle-jvm:latest

    podman build --squash -t quay.io/eformat/ubi-mvn-builder-middle-native:latest -f middleware/Dockerfile.native
    podman push quay.io/eformat/ubi-mvn-builder-middle-native:latest
    ```

3. Login to OpenShift and create an s2i build for your application.

    `JVM fast-jar`
    ```bash
    oc new-build --name=jvm-build quay.io/eformat/ubi-mvn-builder:latest~https://github.com/eformat/code-with-quarkus -e MAVEN_BUILD_OPTS="-Dquarkus.package.type=fast-jar -DskipTests" -e MAVEN_CLEAR_REPO="true"
    ```

    `Native binary`
    ```bash
    oc new-build --name=native-build quay.io/eformat/ubi-mvn-builder:latest~https://github.com/eformat/code-with-quarkus -e MAVEN_CLEAR_REPO="true"
    ```

4. Build runtime Applications.

    `JVM fast-jar`
    ```bash
    oc new-build --name=jvm --strategy docker --dockerfile - < ./runtime/Dockerfile.jvm
    ```

    `Native binary`
    ```bash
    oc new-build --name=native --strategy docker --dockerfile - < ./runtime/Dockerfile.native
    ```

5. Deploy Applications.

    `JVM fast-jar`
    ```bash
    oc new-app jvm
    oc expose svc/jvm
    oc patch route/jvm --type=json -p '[{"op":"add", "path":"/spec/tls", "value":{"termination":"edge","insecureEdgeTerminationPolicy":"Redirect"}}]'
    ```

    `Native binary`
    ```bash
    oc new-app native
    oc expose svc/native
    oc patch route/native --type=json -p '[{"op":"add", "path":"/spec/tls", "value":{"termination":"edge","insecureEdgeTerminationPolicy":"Redirect"}}]'
    ```

### Create triggers


```bash
oc tag --source=docker quay.io/eformat/ubi-mvn-builder:latest code-jvm/ubi-mvn-builder:latest --reference-policy=local --insecure=true
```

BuildConfig source trigger.

### Options to Maven Build

```bash
| Env Variable             | Example              | Description                                         |
|--------------------------|----------------------|-----------------------------------------------------|
| HTTPS_PROXY              |                      |                                                     |
| HTTP_PROXY_HOST          |                      |                                                     |
| HTTP_PROXY_PORT          |                      |                                                     |
| HTTP_PROXY_PASSWORD      |                      |                                                     |
| HTTP_PROXY_USERNAME      |                      |                                                     |
| HTTP_PROXY_NONPROXYHOSTS |                      |                                                     |
| MAVEN_MIRROR_URL         |                      |                                                     |
| MAVEN_BUILD_OPTS         |                      |                                                     |
| MAVEN_CLEAR_REPO         |                      |                                                     |
```
