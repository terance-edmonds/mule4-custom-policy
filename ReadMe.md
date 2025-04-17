# Custom Policy and API Setup on Mule 4

This guide provides step-by-step instructions for creating a custom policy and setting up an API using Mule 4, including prerequisites, configuration, and testing.

## Prerequisites

- **Java 17**
- **Anypoint Studio 7.21.0**
- **MuleSoft Account**: Create an account at https://anypoint.mulesoft.com/login/signup?apintent=generic

## Setup `settings.xml` (Linux)

1. A sample `settings.xml` file is located at <a target="_blank" href="./samples/.m2/settings.xml">`./samples/.m2/settings.xml`</a>.
2. Replace `<username>{username}</username>` and `<password>{password}</password>` with your Anypoint Platform credentials.
3. Update the `settings.xml` file:

   ```bash
   nano ~/.m2/settings.xml
   ```
4. Copy the modified `settings.xml` content into `~/.m2/settings.xml`.

## Creating a Custom Policy

Reference: MuleSoft Custom Policy Documentation

A sample custom policy is available at <a target="_blank" href="./custom-logging-policy">`./custom-logging-policy`</a>.

### Steps

1. **Obtain Organization ID**:

   - Go to https://anypoint.mulesoft.com/accounts/businessGroups.
   - Select your business group and note the **Business Group ID**.

2. **Generate Custom Policy**: Run the following Maven command, replacing `${orgId}` with your organization ID:

   ```bash
   mvn -Parchetype-repository archetype:generate \
   -DarchetypeGroupId=org.mule.tools \
   -DarchetypeArtifactId=api-gateway-custom-policy-archetype \
   -DarchetypeVersion=1.2.0 \
   -DgroupId=${orgId} \
   -DartifactId=custom-logging-policy \
   -Dversion=1.0.0 \
   -Dpackage=mule-policy
   ```

3. **Provide Prompt Details**:

   - `policyDescription`: Custom logging policy
   - `policyName`: custom-logging-policy
   - `groupId`: ${orgId}

4. **Update Policy Configuration**:

   - Replace the contents of `custom-logging-policy/custom-logging-policy.yaml` with the sample provided at <a target="_blank" href="./samples/custom-logging-policy/custom-logging-policy.yaml">`./samples/custom-logging-policy/custom-logging-policy.yaml`</a>.

5. **Modify** `template.xml`:

   - In `custom-logging-policy/src/main/mule/template.xml`, replace the `<http-policy:source>` section with:

     ```xml
     <http-policy:source>
         <logger message="Started inboundLogExpression Policy execution" />
         <logger level="INFO" message="{{{inboundLogExpression}}}"/>
         <logger message="Ended inboundLogExpression Policy execution" />
     
         {{#if requestPayload}}
         <logger message="Started requestPayload Policy execution" />
         <logger level="INFO" message="#[payload]"/>
         <logger message="Ended requestPayload Policy execution" />
         {{/if}}
     
         {{#each logRequestHeader}}
         <logger message="Started logRequestHeader Policy execution" />
         <logger level="INFO" message="{{{this.key}}}:{{{this.value}}}"/>
         <logger message="Ended logRequestHeader Policy execution" />
         {{/each}}
     
         <http-policy:execute-next/>
     
         {{#each logResponseHeader}}
         <logger message="Started logResponseHeader Policy execution" />
         <logger level="INFO" message="{{{this.key}}}:{{{this.value}}}"/>
         <logger message="Ended logResponseHeader Policy execution" />
         {{/each}}
     
         {{#if responsePayload}}
         <logger message="Started responsePayload Policy execution" />
         <logger level="INFO" message="#[payload]"/>
         <logger message="Ended responsePayload Policy execution" />
         {{/if}}
     </http-policy:source>
     ```

6. **Update** `pom.xml`:

   - In `custom-logging-policy/pom.xml`, comment out the following dependency (as it is for enterprise use only):

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

   - Navigate to the `custom-logging-policy` directory and run:

     ```bash
     mvn clean package
     mvn clean deploy
     ```

8. **Verify Policy**:

   - Visit https://anypoint.mulesoft.com/exchange.
   - Search for **custom logging policy** to confirm the policy is available.

## Creating an API

Follow the guide: [API Auto-Discovery for Mule 4](https://apisero.medium.com/api-auto-discovery-for-mule-4-application-3022202b9556)

### Steps

1. **Use Sample RAML**:

   - Use the sample RAML file located at <a target="_blank" href="./samples/raml/hello-world.raml">`./samples/raml/hello-world.raml`</a>.

2. **Find Environment Credentials**:

   - Navigate to https://anypoint.mulesoft.com/accounts/businessGroups/${your-orgId-here}/environments to find credentials for the environment where the application will be deployed.

3. **Deploy Application**:

   - When exporting the application in Anypoint Studio:
     - Under the **Mule** section, select **Anypoint Studio Project to Mule Deployable Archive**.
   - Deploy the application in **Runtime Manager** to activate API Auto-Discovery.

## Testing

1. **Check Public Endpoint**:

   - In **Runtime Manager** → **Settings**, locate the **Public Endpoint** for your application. For example:

     ```
     Public Endpoint: https://hello-world-k4qecw.5sc6y6-2.usa-e2.cloudhub.io
     ```

2. **Invoke the API**:

   - Use the endpoint to invoke the API, e.g.:

     ```
     https://hello-world-k4qecw.5sc6y6-2.usa-e2.cloudhub.io/api/greeting
     ```

3. **Verify Logs**:

   - Check the **Runtime Logs** in Runtime Manager to view the logs generated by the custom policy.