# Apigee X Target Platform Analysis
**Task**: RESEARCH-002 - Complete Apigee X research for Layer 7 migration tool  
**Status**: ✅ COMPLETED  
**Date**: January 2025  

## Executive Summary
Apigee X is Google's cloud-native API management platform that uses a proxy bundle structure with XML-based policies. This analysis provides comprehensive details on Apigee X architecture, policy structures, and Maven deployment mechanisms needed for our Layer 7 v11.1 migration tool.

## 1. Apigee X Architecture Overview

### Core Concepts
- **API Proxy**: A proxy that sits between client applications and backend services
- **ProxyEndpoint**: Inbound configuration (client-facing interface)
- **TargetEndpoint**: Outbound configuration (backend service interface)
- **Policies**: XML configuration files that implement specific functionality
- **Flows**: Request/response processing pipelines

### Bundle Structure
```
apiproxy/
├── {proxy-name}.xml          # Base configuration
├── policies/                 # Policy XML files
├── proxies/                  # ProxyEndpoint configurations
├── targets/                  # TargetEndpoint configurations
└── resources/                # Scripts, JARs, XSLT files
```

## 2. Authentication Policy Analysis

### BasicAuthentication Policy Structure
Found comprehensive policy XML structure for basic authentication:

```xml
<BasicAuthentication async="false" continueOnError="false" enabled="true" name="Basic-Authentication-1">
    <DisplayName>Basic Authentication 1</DisplayName>
    <Operation>Encode</Operation>
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <User ref="request.queryparam.username" />
    <Password ref="request.queryparam.password" />
    <AssignTo ref="request.header.Authorization" />
</BasicAuthentication>
```

#### Key Elements:
- **Operation**: `Encode` (create header) or `Decode` (extract credentials)
- **User/Password**: References to flow variables containing credentials
- **AssignTo**: Target location for the authorization header
- **IgnoreUnresolvedVariables**: Error handling for missing variables

#### Flow Variables Available:
- `basicauthentication.{policy_name}.username`
- `basicauthentication.{policy_name}.password`
- `basicauthentication.{policy_name}.user.authentication.status`

## 3. Proxy Structure and Configuration

### Base Configuration (`/apiproxy/{name}.xml`)
```xml
<APIProxy name="weatherapi">
    <ConfigurationVersion majorVersion="4" minorVersion="0"/>
    <Description>API proxy description</Description>
    <DisplayName>Weather API</DisplayName>
    <Policies>
        <Policy>Basic-Authentication-1</Policy>
    </Policies>
    <ProxyEndpoints>
        <ProxyEndpoint>default</ProxyEndpoint>
    </ProxyEndpoints>
    <TargetEndpoints>
        <TargetEndpoint>default</TargetEndpoint>
    </TargetEndpoints>
</APIProxy>
```

### ProxyEndpoint Configuration (`/apiproxy/proxies/default.xml`)
```xml
<ProxyEndpoint name="default">
    <PreFlow>
        <Request>
            <Step>
                <Name>Basic-Authentication-1</Name>
                <Condition>request.header.authorization = null</Condition>
            </Step>
        </Request>
    </PreFlow>
    <Flows/>
    <PostFlow/>
    <HTTPProxyConnection>
        <BasePath>/weather</BasePath>
        <Properties/>
    </HTTPProxyConnection>
    <RouteRule name="default">
        <TargetEndpoint>default</TargetEndpoint>
    </RouteRule>
</ProxyEndpoint>
```

### TargetEndpoint Configuration (`/apiproxy/targets/default.xml`)
```xml
<TargetEndpoint name="default">
    <PreFlow/>
    <Flows/>
    <PostFlow/>
    <HTTPTargetConnection>
        <URL>http://api.backend.com</URL>
        <Authentication>
            <GoogleAccessToken>
                <Scopes>
                    <Scope>https://www.googleapis.com/auth/cloud-platform</Scope>
                </Scopes>
            </GoogleAccessToken>
        </Authentication>
    </HTTPTargetConnection>
</TargetEndpoint>
```

## 4. Flow Processing Pipeline

### Request Flow Sequence:
1. Proxy Request PreFlow
2. Proxy Request Conditional Flows (Optional)
3. Proxy Request PostFlow
4. Target Request PreFlow
5. Target Request Conditional Flows (Optional)
6. Target Request PostFlow

### Response Flow Sequence:
1. Target Response PreFlow
2. Target Response Conditional Flows (Optional)
3. Target Response PostFlow
4. Proxy Response PreFlow
5. Proxy Response Conditional Flows (Optional)
6. Proxy Response PostFlow
7. PostClientFlow Response (Optional)

### Policy Attachment Format:
```xml
<Step>
    <Name>PolicyName</Name>
    <Condition>optional_condition</Condition>
</Step>
```

## 5. Routing and Conditional Logic

### RouteRule Examples:
```xml
<!-- Simple route to named target -->
<RouteRule name="default">
    <TargetEndpoint>backend-service</TargetEndpoint>
</RouteRule>

<!-- Conditional routing -->
<RouteRule name="v2-route">
    <Condition>request.header.version = "v2"</Condition>
    <TargetEndpoint>backend-v2</TargetEndpoint>
</RouteRule>

<!-- Direct URL routing -->
<RouteRule name="external">
    <URL>https://api.external.com/v1</URL>
</RouteRule>

<!-- Null route (no backend call) -->
<RouteRule name="cached-response"/>
```

### Conditional Flow Examples:
```xml
<Flow name="GetWeather">
    <Condition>request.verb = "GET" AND proxy.pathsuffix MatchesPath "/weather"</Condition>
    <Request>
        <Step>
            <Name>Verify-API-Key</Name>
        </Step>
    </Request>
    <Response>
        <Step>
            <Name>Add-CORS-Headers</Name>
        </Step>
    </Response>
</Flow>
```

## 6. Maven Deployment Structure

### Project Structure:
```
src/
└── gateway/
    ├── shared-pom.xml          # Parent POM with shared configuration
    └── {proxy-name}/
        ├── pom.xml             # Child POM for specific proxy
        ├── config.json         # Environment-specific configurations
        └── apiproxy/           # Proxy bundle directory
            ├── {proxy-name}.xml
            ├── policies/
            ├── proxies/
            └── targets/
```

### Parent POM Configuration:
```xml
<groupId>com.company.apigee</groupId>
<artifactId>parent-pom</artifactId>
<packaging>pom</packaging>

<properties>
    <apigee.org>${org}</apigee.org>
    <apigee.env>${env}</apigee.env>
    <apigee.hosturl>https://apigee.googleapis.com</apigee.hosturl>
    <apigee.apiversion>v1</apigee.apiversion>
</properties>

<profiles>
    <profile>
        <id>test</id>
        <properties>
            <apigee.profile>test</apigee.profile>
            <apigee.env>test</apigee.env>
            <apigee.org>my-org-test</apigee.org>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <apigee.profile>prod</apigee.profile>
            <apigee.env>prod</apigee.env>
            <apigee.org>my-org-prod</apigee.org>
        </properties>
    </profile>
</profiles>

<build>
    <plugins>
        <plugin>
            <groupId>io.apigee.build-tools.enterprise4g</groupId>
            <artifactId>apigee-edge-maven-plugin</artifactId>
            <version>2.5.2</version>
        </plugin>
    </plugins>
</build>
```

### Child POM Configuration:
```xml
<parent>
    <groupId>com.company.apigee</groupId>
    <artifactId>parent-pom</artifactId>
    <version>1.0</version>
    <relativePath>../shared-pom.xml</relativePath>
</parent>

<artifactId>weather-api-v1</artifactId>
<name>weather-api-v1</name>
<packaging>pom</packaging>
```

### Deployment Commands:
```bash
# Deploy to test environment
mvn clean install -Ptest -Dbearer=$(gcloud auth print-access-token)

# Deploy to production
mvn clean install -Pprod -Dbearer=$(gcloud auth print-access-token)

# Deploy with Google service account authentication
mvn clean install -Ptest -Dbearer=$(gcloud auth print-access-token) \
    -DgoogleTokenEmail=service-account@project.iam.gserviceaccount.com

# Async deployment (non-blocking)
mvn clean install -Ptest -Dbearer=$(gcloud auth print-access-token) \
    -Dapigee.options=async

# Import without activation
mvn clean install -Ptest -Dbearer=$(gcloud auth print-access-token) \
    -Dapigee.options=inactive
```

## 7. Environment Configuration (config.json)

Environment-specific configurations using XPath-based replacements:

```json
{
  "configurations": [
    {
      "name": "test",
      "policies": [
        {
          "name": "BasicAuthentication-1.xml",
          "tokens": [
            {
              "xpath": "/BasicAuthentication/User/@ref",
              "value": "test.user.variable"
            },
            {
              "xpath": "/BasicAuthentication/Password/@ref", 
              "value": "test.password.variable"
            }
          ]
        }
      ],
      "proxies": [
        {
          "name": "default.xml",
          "tokens": [
            {
              "xpath": "/ProxyEndpoint/HTTPProxyConnection/BasePath",
              "value": "/test/api/v1"
            }
          ]
        }
      ],
      "targets": [
        {
          "name": "default.xml", 
          "tokens": [
            {
              "xpath": "/TargetEndpoint/HTTPTargetConnection/URL",
              "value": "https://test-backend.company.com"
            }
          ]
        }
      ]
    }
  ]
}
```

## 8. Key Mapping Insights for Layer 7 Migration

### Authentication Mapping:
- **Layer 7 HTTP Basic Authentication** → **Apigee BasicAuthentication Policy**
- Layer 7 `L7p:Authentication` assertion → Apigee `<Operation>Decode</Operation>`
- WS-Policy credential extraction → Apigee flow variable references

### Routing Mapping:
- **Layer 7 Routing Assertion** → **Apigee RouteRule**
- Layer 7 `L7p:HttpRoutingAssertion` → Apigee `<TargetEndpoint>` or `<URL>`
- Layer 7 conditional routing → Apigee conditional RouteRules

### Policy Structure Mapping:
- Layer 7 nested policy XML → Apigee flat policy files in `/policies/`
- Layer 7 WS-Policy structure → Apigee flow attachment points
- Layer 7 `wsp:All` containers → Apigee PreFlow/PostFlow execution

## 9. TLS/SSL Configuration

Apigee supports comprehensive TLS/SSL configuration:

```xml
<HTTPTargetConnection>
    <URL>https://secure-backend.com</URL>
    <SSLInfo>
        <Enabled>true</Enabled>
        <Enforce>true</Enforce>
        <ClientAuthEnabled>true</ClientAuthEnabled>
        <KeyStore>client-keystore</KeyStore>
        <KeyAlias>client-cert</KeyAlias>
        <TrustStore>trusted-ca-store</TrustStore>
        <IgnoreValidationErrors>false</IgnoreValidationErrors>
        <Protocols>
            <Protocol>TLSv1.2</Protocol>
            <Protocol>TLSv1.3</Protocol>
        </Protocols>
    </SSLInfo>
</HTTPTargetConnection>
```

## 10. Error Handling and Fault Rules

```xml
<FaultRules>
    <FaultRule name="AuthenticationFailure">
        <Condition>basicauthentication.{policy_name}.failed = true</Condition>
        <Step>
            <Name>Return-401-Unauthorized</Name>
        </Step>
    </FaultRule>
</FaultRules>

<DefaultFaultRule>
    <Step>
        <Name>Return-Generic-Error</Name>
    </Step>
</DefaultFaultRule>
```

## Next Steps for RESEARCH-003

With this comprehensive Apigee X analysis complete, the next research phase should focus on:

1. **Policy Mapping Matrix**: Create detailed mapping between Layer 7 policy types and Apigee equivalents
2. **XML Transformation Rules**: Define conversion patterns from Layer 7 WS-Policy to Apigee policy XML
3. **Flow Logic Translation**: Map Layer 7 assertion sequences to Apigee flow structures
4. **Maven Template Generation**: Create reusable Maven project templates for converted proxies

## Conclusion

Apigee X provides a robust, XML-based policy system with comprehensive Maven deployment support. The platform's flow-based architecture and conditional routing capabilities provide excellent mapping opportunities for Layer 7 v11.1 policies. The Maven plugin offers sophisticated deployment automation that will integrate well with our migration tool's output generation.

Key advantages for migration:
- ✅ XML-based configuration similar to Layer 7
- ✅ Comprehensive authentication policy support
- ✅ Flexible routing and conditional logic
- ✅ Mature Maven deployment tooling
- ✅ Environment-specific configuration management
- ✅ Enterprise-grade TLS/SSL support
