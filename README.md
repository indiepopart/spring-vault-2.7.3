# Tutorial: Secure Secrets With Spring Cloud Config and Vault

This repository contains all the code for testing a Spring Cloud Configuration Server using Vault as backend, and a demo client application with Okta OIDC authentication and Spring Boot 2.7.3.

**Prerequisites:**

- [Java OpenJDK 17](https://jdk.java.net/java-se-ri/17)
- [Okta CLI 0.10.0](https://cli.okta.com)
- [Docker 20.10.12](https://docs.docker.com/engine/install/)
- [HTTPie 3.2.1](https://httpie.io/docs/cli/installation)
- [Vault 1.11.3](https://hub.docker.com/_/vault)

## Getting Started

To install this example, run the following commands:
```bash
git clone https://github.com/indiepopart/spring-vault-2.7.3.git
```

## Create the OIDC Application in Okta

Open a command line session at the root of `vault-demo-app`.

Before you begin, you’ll need a free Okta developer account. Install the [Okta CLI](https://cli.okta.com/) and run `okta register` to sign up for a new account. If you already have an account, run `okta login`. Then, run `okta apps create`. Select the default app name, or change it as you see fit. Choose **Web** and press **Enter**.

Select **Okta Spring Boot Starter**. Accept the default Redirect URI values provided for you. That is, a Login Redirect of `http://localhost:8080/login/oauth2/code/okta` and a Logout Redirect of `http://localhost:8080`.

<p>
<details>
  <summary>What does the Okta CLI do?</summary>

  The Okta CLI will create an OIDC Web App in your Okta Org. It will add the redirect URIs you specified and grant access to the Everyone group. You will see output like the following when it’s finished:

  ```shell
  Okta application configuration has been written to: /path/to/app/src/main/resources/application.properties
  ```

  Open `src/main/resources/application.properties` to see the issuer and credentials for your app.

  ```shell
  okta.oauth2.issuer=https://dev-133337.okta.com/oauth2/default
  okta.oauth2.client-id=0oab8eb55Kb9jdMIr5d6
  okta.oauth2.client-secret=NEVER-SHOW-SECRETS
  ```

  **NOTE**: You can also use the Okta Admin Console to create your app. See [Create a Spring Boot App](https://developer.okta.com/docs/guides/sign-into-web-app/springboot/create-okta-application/) for more information.

</details>
</p>

Copy the values from `src/main/resources/application.properties` and delete the file.

## Run Vault

```shell
docker pull vault
docker run --cap-add=IPC_LOCK \
-e 'VAULT_DEV_ROOT_TOKEN_ID=00000000-0000-0000-0000-000000000000' \
-p 8200:8200 \
-v {hostPath}:/vault/logs \
--name my-vault vault
```

Store the secrets, using the values returned before from Okta CLI.

```shell
docker exec -it my-vault /bin/sh
export VAULT_TOKEN="00000000-0000-0000-0000-000000000000"
export VAULT_ADDR="http://127.0.0.1:8200"
vault kv put secret/vault-demo-app,dev \
vault kv put secret/vault-demo-app,dev \
okta.oauth2.clientId="{yourClientId}" \
okta.oauth2.clientSecret="{yourClientSecret}" \
okta.oauth2.issuer="{yourIssuerURI}"
```

## Run the applications with Maven

Run `vault-config-server`:

```shell
cd spring-vault/vault-config-server
./mvnw spring-boot:run
```

Run `vault-demo-app`:
```shell
SPRING_CLOUD_CONFIG_TOKEN=00000000-0000-0000-0000-000000000000 \
./mvnw spring-boot:run
```

Got to http://localhost:8080 and login with Okta.
