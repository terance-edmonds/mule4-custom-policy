# Custom API Call Policy on Mule 4

This guide provides instructions for creating, configuring, and deploying a custom Mule 4 policy that sends incoming request payloads and headers to an external API endpoint (e.g., a webhook) when a request is received.

## Prerequisites

- Refer to the top-level `README.md` for prerequisites (Java 17, Anypoint Studio 7.21.0, MuleSoft account) and `settings.xml` setup.

## Creating a Custom API Call Policy

Reference: MuleSoft Custom Policy Documentation

A sample custom policy is available in this directory (`custom-api-call-policy`).

### Steps

1. **Obtain Organization ID**:

   - Go to https://anypoint.mulesoft.com/accounts/businessGroups.
   - Select your business group and note the **Business Group ID**.

2. **Generate Custom Policy**:

   Run the following Maven command, replacing `${orgId}` with your organization ID:

   ```bash
   mvn -Parchetype-repository archetype:generate \
   -DarchetypeGroupId=org.mule.tools \
   -DarchetypeArtifactId=api-gateway-custom-policy-archetype \
   -DarchetypeVersion=1.2.0 \
   -DgroupId=${orgId} \
   -DartifactId=custom-api-call-policy \
   -Dversion=1.0.0 \
   -Dpackage=mule-policy
   ```

3. **Provide Prompt Details**:

- `policyDescription`: Custom policy to call an external API endpoint
- `policyName`: custom-api-call-policy
- `groupId`: ${orgId}

4. **Update Policy Configuration**:

- Replace the contents of `custom-api-call-policy.yaml` with the sample provided below or at `./custom-api-call-policy/custom-api-call-policy.yaml`:

  ```yaml
  id: api-call-policy
  name: api-call-policy
  description: Custom policy to call an external API endpoint when a request is received
  category: Custom
  type: custom
  resourceLevelSupported: true
  encryptionSupported: false
  standalone: true
  supportedJavaVersions: ["8", "11", "17"]
  requiredCharacteristics: []
  providedCharacteristics: []
  configuration:
    - propertyName: apiEndpointUrl
      name: API Endpoint URL
      description: The URL of the external API to call
      type: string
      optional: false
      defaultValue: "https://webhook.site/e18#pragmaf3116-ff52-4b6c-9d60-42e82ac56d1a"
    - propertyName: apiHost
      name: API Host
      description: The hostname of the external API
      type: string
      optional: false
      defaultValue: "webhook.site"
    - propertyName: apiPort
      name: API Port
      description: The port number of the external API (e.g., 443 for HTTPS)
      type: string
      optional: false
      defaultValue: "443"
  ```

1. **Modify** `template.xml`:

- In `src/main/mule/template.xml`, replace the `<http-policy:proxy>` section with:

  ```xml
  <http-policy:proxy name="{{{policyId}}}-custom-api-call-policy">
      <http-policy:source>
          <!-- Store the incoming request payload to restore it after the API call -->
          <set-variable variableName="originalPayload" value="#[payload]" />
  
          <!-- Send the incoming request's payload and headers to the external API -->
          <http:request method="POST" url="{{{apiEndpointUrl}}}" config-ref="HTTP_Request_configuration">
              <http:headers>#[attributes.headers]</http:headers>
          </http:request>
  
          <!-- Restore the original payload to ensure the request continues unchanged -->
          <set-payload value="#[vars.originalPayload]" />
  
          <!-- Proceed with the original request processing -->
          <http-policy:execute-next/>
      </http-policy:source>
  </http-policy:proxy>
  
  <!-- HTTP connector configuration for the external API -->
  <http:request-config name="HTTP_Request_configuration">
      <http:request-connection host="{{{apiHost}}}" port="{{{apiPort}}}" />
  </http:request-config>
  ```

- Ensure the `<mule>` root element includes the necessary namespaces:

  ```xml
  <mule xmlns="http://www.mulesoft.org/schema/mule/core"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:http-policy="http://www.mulesoft.org/schema/mule/http-policy"
        xmlns:http="http://www.mulesoft.org/schema/mule/http"
        xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd
                            http://www.mulesoft.org/schema/m_markup://mule/http-policy http://www.mulesoft.org/schema/mule/http-policy/current/mule-http-policy.xsd
                            http://www.mulesoft.org/schema/mule/http http://www.mules#pragmaoft.org/schema/mule/http/current/mule-http.xsd">
  ```

6. **Update** `pom.xml`:

- In `pom.xml`, ensure the following dependency is included for the HTTP connector:

  ```xml
  <dependency>
      <groupId>org.mule.connectors</groupId>
      <artifactId>mule-http-connector</artifactId>
      <version>1.9.3</version>
      <classifier>mule-plugin</classifier>
  </dependency>
  ```

- Comment out or remove any enterprise-only dependencies (e.g., `mule-http-policy-transform-extension`):

  ```xml
  <!--
  <dependency>
      <groupId>com.mulesoft.anypoint</groupId>
      <artifactId>mule-http-policy-transform-extension</artifactId>
      <version>${http.policy.transform.extension}</version>
      <classifier>mule-plugin</classifier>
      <exclusions>
          <exclusion>
              <groupId>org.mule.connectors</groupId>
              <artifactId>mule-http-connector</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  -->
  ```

7. **Build and Deploy Policy**:

- Navigate to the `custom-api-call-policy` directory and run:

  ```bash
  mvn clean package
  mvn clean deploy
  ```

8. **Verify Policy**:

- Visit https://anypoint.mulesoft.com/exchange.
- Search for **custom API call policy** to confirm the policy is available.

## Applying the Policy

1. **Apply Policy in API Manager**:

- In Anypoint Platform, navigate to **API Manager**.
- Select your API (e.g., `hello-world`).
- Go to **Policies** and click **Apply New Policy**.
- Choose **custom-api-call-policy** from the list.
- Configure the policy with the following values:
  - **API Endpoint URL**: `https://webhook.site/e18f3116-ff52-4b6c-9d60-42e82ac56d1a`
  - **API Host**: `webhook.site`
  - **API Port**: `443`

2. **Save and Apply**:

- Save the policy configuration and apply it to the API.

## Testing

1. **Invoke the API**:

- Follow the API invocation steps in the top-level `README.md` to send a POST request to your API (e.g., `https://hello-world-k4qecw.5sc6y6-2.usa-e2.cloudhub.io/api/greeting`) with a sample payload.

2. **Verify Webhook**:

- For testing, you can use the provided webhook.site endpoint (`https://webhook.site/e18f3116-ff52-4b6c-9d60-42e82ac56d1a`).
- Visit https://webhook.site and check the dashboard for your endpoint.
- Confirm the request’s payload (e.g., `{"test": "data"}`) and headers are received.

3. **Verify Logs**:

- In **Runtime Manager**, check the **Logs** to ensure no errors occurred during the webhook call. (Note: This policy does not log the payload; add a logger if needed, e.g., `<logger level="INFO" message="Payload to webhook: #[payload]" />`.)

## Notes

- **Payload Handling**: The policy sends the incoming payload as-is to the webhook. To transform the payload (e.g., wrap in JSON), add a DataWeave transformation before `<http:request>`:

```xml
<dw:transform-message>
   <dw:set-payload><![CDATA[%dw 2.0
output application/json
---
{ "originalPayload": payload, "timestamp": now() }]]></dw:set-payload>
</dw:transform-message>
```

Update the `<mule>` element to include the DataWeave namespace:

```xml
xmlns:dw="http://www.mulesoft.org/schema/mule/ee/dw"
xsi:schemaLocation="... http://www.mulesoft.org/schema/mule/ee/dw http://www.mulesoft.org/schema/mule/ee/dw/current/dw.xsd"
```

- **Error Handling**: For production, add error handling around `<http:request>`:

  ```xml
  <try>
      <http:request method="POST" url="{{{apiEndpointUrl}}}" config-ref="HTTP_Request_configuration">
          <http:headers>#[attributes.headers]</http:headers>
      </http:request>
      <error-handler>
          <on-error-continue>
              <logger level="ERROR" message="Failed to call webhook: #[error.description]"/>
          </on-error-continue>
      </error-handler>
  </try>
  ```

- **Webhook Testing**: The webhook.site endpoint is temporary and ideal for testing. For production, replace it with your actual API endpoint in the policy configuration.