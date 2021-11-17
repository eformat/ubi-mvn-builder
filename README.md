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
>
> 🐒🐒🐒 Why do this ?
>   - Latest Java toolchain
>   - A [Secure Supply Chain](https://www.redhat.com/en/blog/architecting-containers-part-5-building-secure-and-manageable-container-software-supply-chain)
>   - Smaller image sizes, less attack surface:
>       - Builder image size            = 522 MiB
>       - JVM runtime image size        = 128 MiB
>       - JVM application image size    = 143 MiB
>       - Native runtime image size     = 43 MiB
>       - Native application image size = 75 MiB

1. Create the base s2i core build image and push to remote repo for reuse across clusters.

    ```bash
    podman build --squash -t quay.io/eformat/ubi-mvn-builder:latest -f builder/Dockerfile
    podman push quay.io/eformat/ubi-mvn-builder:latest
    ```

2. Create the runtimes and push to remote repo for reuse across clusters.

    `JVM fast-jar`
    ```bash
    podman build --squash -t quay.io/eformat/ubi-mvn-runtime-jvm:latest -f runtime/Dockerfile.jvm
    podman push quay.io/eformat/ubi-mvn-runtime-jvm:latest
    ```

    `Native binary`
    ```
    podman build --squash -t quay.io/eformat/ubi-mvn-runtime-native:latest -f runtime/Dockerfile.native
    podman push quay.io/eformat/ubi-mvn-runtime-native:latest
    ```

3. Login to OpenShift, create a project then create the s2i build for your applications.

    ```bash
    oc new-project demo
    ```

    `JVM fast-jar`
    ```bash
    oc new-build --name=jvm-build \
      quay.io/eformat/ubi-mvn-builder:latest~https://github.com/eformat/code-with-quarkus \
      -e MAVEN_BUILD_OPTS="-Dquarkus.package.type=fast-jar -DskipTests" \
      -e MAVEN_CLEAR_REPO="true"
    ```

    `Native binary`
    ```bash
    oc new-build --name=native-build \
      quay.io/eformat/ubi-mvn-builder:latest~https://github.com/eformat/code-with-quarkus \
      -e MAVEN_CLEAR_REPO="true"
    ```

4. Build runtime Applications.

    `JVM fast-jar`
    ```bash
    oc new-build --name=jvm \
      --build-arg BUILD_IMAGE=image-registry.openshift-image-registry.svc:5000/$(oc project -q)/jvm-build:latest \
      --strategy docker --dockerfile - < ./application/Dockerfile.jvm
    ```

    `Native binary`
    ```bash
    oc new-build --name=native \
      --build-arg BUILD_IMAGE=image-registry.openshift-image-registry.svc:5000/$(oc project -q)/native-build:latest \
      --strategy docker --dockerfile - < ./application/Dockerfile.native
    ```

5. Deploy Applications.

    `JVM fast-jar`
    ```bash
    oc new-app jvm
    oc expose svc/jvm
    oc patch route/jvm \
      --type=json -p '[{"op":"add", "path":"/spec/tls", "value":{"termination":"edge","insecureEdgeTerminationPolicy":"Redirect"}}]'
    ```

    `Native binary`
    ```bash
    oc new-app native
    oc expose svc/native
    oc patch route/native \
      --type=json -p '[{"op":"add", "path":"/spec/tls", "value":{"termination":"edge","insecureEdgeTerminationPolicy":"Redirect"}}]'
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
