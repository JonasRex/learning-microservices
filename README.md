# Microservice tutorial

## Section about KeyCloak

Make a docker container with KeyCloak: https://www.keycloak.org/getting-started/getting-started-docker

OBS1: The tutorial recommended to use port 8181 instead of the default 8080, so the command to run the container is:
OBS2: Should change the admin password to something more secure than admin

`docker run -p 8181:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:22.0.5 start-dev`

older version from the tutorial: `docker run -p 8181:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:18.0.0 start-dev`

After the container is running, go to http://localhost:8181 and login with the admin credentials. and create a new realm.

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

## Zipkin and Sleuth
Zipkin and sleuth are no longer used with spring cloud. MicroMeter is the new way to do it.
