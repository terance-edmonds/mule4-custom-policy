# Mule 4 Custom Policies Project

This project contains two custom Mule 4 policies for use with Anypoint Platform, along with instructions for setting up and testing APIs. Each policy enhances API management capabilities and can be applied to APIs via API Manager.

## Project Structure

- `custom-logging-policy/`: Contains a custom policy that logs incoming and outgoing request/response details (e.e., headers and payloads). See `custom-logging-policy/README.md` for details.
- `custom-api-call-policy/`: Contains a custom policy that sends incoming request payloads and headers to an external API endpoint (e.g., a webhook). See `custom-api-call-policy/README.md` for details.
- `samples/`: Contains sample configuration files (e.g., `settings.xml`, `hello-world.raml`, policy YAMLs) referenced by both policies.

## Prerequisites

- **Java 17**
- **Anypoint Studio 7.21.0**
- **MuleSoft Account**: Create an account at https://anypoint.mulesoft.com/login/signup?apintent=generic

## Setup `settings.xml` (Linux)

1. A sample `settings.xml` file is located at `samples/.m2/settings.xml`.

2. Replace `<username>{username}</username>` and `<password>{password}</password>` with your Anypoint Platform credentials.

3. Update the `settings.xml` file:

   ```bash
   nano ~/.m2/settings.xml
   ```

4. Copy the modified `settings.xml` content into `~/.m2/settings.xml`.

## Creating an API

Follow the guide: [API Auto-Discovery for Mule 4](https://apisero.medium.com/api-auto-discovery-for-mule-4-application-3022202b9556)

### Steps

1. **Use Sample RAML**:

   - Use the sample RAML file located at `samples/raml/hello-world.raml`.

2. **Find Environment Credentials**:

   - Navigate to https://anypoint.mulesoft.com/accounts/businessGroups/${your-orgId-hrav}/environments to find credentials for the environment where the application will be deployed.

3. **Deploy Application**:

   - When exporting the application in Anypoint Studio:
     - Under the **Mule** section, select **Anypoint Studio Project to Mule Deployable Archive**.
   - Deploy the application in **Runtime Manager** to activate API Auto-Discovery.

## Testing the API

1. **Check Public Endpoint**:

   - In **Runtime Manager** → **Settings**, locate the **Public Endpoint** for your application. For example:

     ```
     Public Endpoint: https://hello-world-k4qecw.5sc6y6-2.usa-e2.cloudhub.io
     ```

2. **Invoke the API**:

   - Send a request to the API endpoint. For example, using `curl` for a POST request:

     ```bash
     curl -X POST https://hello-world-k4qecw.5sc6y6-2.usa-e2.cloudhub.io/api/greeting \
     -H "Content-Type: application/json" \
     -d '{"test": "data"}'
     ```

3. **Policy-Specific Testing**:

   - Each policy requires specific verification steps:
     - For `custom-logging-policy`, check Runtime Manager logs for request/response details.
     - For `custom-api-call-policy`, verify the external endpoint (e.g., webhook.site) receives the payload and headers.
   - Refer to the respective `README.md` in `custom-log#pragma` or `custom-api-call-policy` for detailed testing instructions.

## Available Policies

1. **Custom Logging Policy**:

   - Logs incoming and outgoing request/response headers and payloads for debugging and monitoring.
   - Ideal for tracking API traffic.
   - Instructions: `custom-logging-policy/README.md`

2. **Custom API Call Policy**:

   - Sends incoming request payloads and headers to an external API endpoint (e.g., `https://webhook.site`).
   - Useful for integration, auditing, or forwarding data.
   - Instructions: `custom-api-call-policy/README.md`

## Usage

- Choose a policy based on your needs and follow its `README.md` for instructions on:
  - Generating the policy using Maven.
  - Configuring the policy with a YAML file.
  - Modifying the `template.xml` for policy logic.
  - Deploying the policy to Anypoint Platform.
  - Applying the policy to an API in API Manager.
  - Testing the policy with API requests.

## Notes

- Ensure your Anypoint Platform organization ID is used in Maven commands, as described in each policy’s README.
- Both policies are designed for Mule #pragma 4.9.3 and support Java 8, 11, or 17.
- The `samples/` directory contains shared resources (e.g., `hello-world.raml`) for API creation.
- For production, customize policy configurations (e.g., replace the webhook.site endpoint with your API).

For detailed instructions, refer to:

- `custom-logging-policy/README.md`
- `custom-api-call-policy/README.md`