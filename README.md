# Microservice tutorial

## Section about KeyCloak (Security)

Make a docker container with KeyCloak: https://www.keycloak.org/getting-started/getting-started-docker

OBS1: The tutorial recommended to use port 8181 instead of the default 8080, so the command to run the container is:
OBS2: Should change the admin password to something more secure than admin

`docker run -p 8181:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:22.0.5 start-dev`

older version from the
tutorial: `docker run -p 8181:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:18.0.0 start-dev`

After the container is running, go to http://localhost:8181 and login with the admin credentials. and create a new
realm.

Now create a new client, and set the following values:

- Client ID: `spring-cloud-client`
- access type = `confidential`
- standard flow enabled = `OFF`
- direct access grants enabled = `OFF`
- service accounts enabled = `ON`

- Save

Now go to the `Credentials` tab and copy the `Secret` value, we will need it later.

Goto realms settings and click OpenId Endpoint Configuration, and copy the `issuer` value, we will need it later.

See this video for more details: https://youtu.be/rbKzR6QWKLI?si=wNxusI295Wfp4jm4


## Dockerfiles 3 ways to do it

### Jib: (What we use and it's very easy)

Add plugin in the root project build file:

```
<build>
...
    
    <plugins>
    ...
     <plugin>
                <groupId>com.google.cloud.tools</groupId>
                <artifactId>jib-maven-plugin</artifactId>
                <version>3.2.1</version>
                <configuration>
                    <from>
                        <image>eclipse-temurin:17.0.4.1_1-jre</image>
                    </from>
                    <to>
                        <image>[YOUR_DOCKER_ACCOUNT]/${project.artifactId}</image>
                    </to>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>dockerBuild</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
    </plugins>
</build>
```

You have to have a docker hub account and be logged in to it in your local machine.

Then run:
`mvn compile jib:dockerBuild` for local docker build

`mvn clean compile jib:build`

### Layered: 

```
FROM eclipse-temurin:17.0.4.1_1-jre as builder
WORKDIR extracted
ADD target/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

FROM eclipse-temurin:17.0.4.1_1-jre
WORKDIR application
COPY --from=builder extracted/dependencies/ ./
COPY --from=builder extracted/spring-boot-loader/ ./
COPY --from=builder extracted/snapshot-dependencies/ ./
COPY --from=builder extracted/application/ ./
EXPOSE 8080
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]

```

### Normal (fat jar):

```
FROM openjdk:17

COPY target/*.jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```


## Example of a deployment.yaml file for a single microservice

```
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-minikube
spec:
  selector:
    app: spring-boot-minikube
  ports:
    - protocol: TCP
      port: 8081 # You can map the service to a different port if needed
      targetPort: 8081 # Match the target port with your container's exposed port
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-minikube
spec:
  replicas: 1 # You can adjust the number of replicas as needed
  selector:
    matchLabels:
      app: spring-boot-minikube
  template:
    metadata:
      labels:
        app: spring-boot-minikube
    spec:
      containers:
        - name: spring-boot-minikube
          image: jonasrex/spring-boot-minikube:latest # Replace with your Docker image name and tag
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8081 # Adjust the port if your Spring Boot app listens on a different port

```

### make sure minikube is running

`minikube start`


### Open minikube dashboard

`minikube dashboard`

### Apply to minikube

`kubectl apply -f deployment.yaml`



### Open deployed service in browser.
`minikube service [name-of-service]`