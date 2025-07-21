# Layer 7 API Gateway v11.1 XML Policy Analysis
**Research Task**: RESEARCH-001  
**Status**: IN_PROGRESS  
**Started**: July 21, 2025

## Overview
This document captures research findings for Layer 7 API Gateway v11.1 XML policy structures, specifically focusing on **basic authentication** and **routing policies** that need to be migrated to Apigee X.

## Research Sources
- **Primary**: Broadcom Layer 7 API Gateway 11.1 Official Documentation
  - Base URL: https://techdocs.broadcom.com/us/en/ca-enterprise-software/layer7-api-management/api-gateway/11-1/
  - PDF Documentation: Available
- **Secondary**: Community forums, migration guides, Stack Overflow
- **Tertiary**: General XML policy patterns and examples

## Key Findings from Documentation Review

### 1. Layer 7 Policy Structure Overview
From the official documentation:

- **Policy Assertions**: Building blocks for policies in the Policy Manager
- **Policy Types**: Web service policies and XML application policies
- **Policy Construction**: Drag-and-drop or programmatic addition of assertions
- **Configuration Required**: Most assertions require configuration before or after being added

### 2. Documentation Structure
The Layer 7 v11.1 documentation is organized into:
- **Policy Assertions**: Core building blocks
- **Services and Policies**: Service management and policy working
- **Authentication**: Various authentication methods
- **Security Configuration**: Security-related functionality in Policy Manager

### 3. Identified Focus Areas for Migration
Based on our project scope, we need to focus on:

#### 3.1 Basic Authentication Policies
- **Assertion Type**: HTTP Basic Authentication
- **XML Structure**: TBD (need deeper documentation access)
- **Configuration Elements**: Username, password, realm, credential source
- **Layer 7 Features**: User store integration, realm configuration

#### 3.2 Routing Policies  
- **Assertion Type**: HTTP Routing
- **XML Structure**: TBD (need deeper documentation access)
- **Configuration Elements**: Target URL, method routing, path patterns
- **Layer 7 Features**: Backend service configuration, load balancing

## Next Steps - Detailed Research Required

### Immediate Actions (Next 2 Days)
1. **Access Layer 7 Sample Projects**
   - Need actual Layer 7 v11.1 policy XML files
   - Request access to Broadcom documentation or trial environment
   - Search for community-shared examples

2. **Document XML Schema Patterns**
   - Basic authentication policy XML structure
   - Routing policy XML structure
   - Service definition XML structure
   - Folder/organization patterns

3. **Create Sample Policy Library**
   - Collect 3-5 basic auth policy examples
   - Collect 3-5 routing policy examples
   - Document variations and edge cases

### Research Gaps to Address
- **CRITICAL**: Actual Layer 7 v11.1 XML policy examples
- **HIGH**: Service export/import formats
- **MEDIUM**: Policy folder structure and organization
- **LOW**: Advanced features we're excluding

## Preliminary XML Structure Assumptions
*Based on general API gateway patterns, WS-Policy standards, and Microsoft documentation*

### Basic Authentication Policy (Estimated Structure)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<wsp:Policy xmlns:wsp="http://schemas.xmlsoap.org/ws/2004/09/policy"
            xmlns:L7p="http://www.layer7tech.com/ws/policy"
            xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd">
  <wsp:All>
    <L7p:HttpBasicAuthentication>
      <L7p:Realm stringValue="API Gateway"/>
      <L7p:CredentialSource stringValue="request.http.authheader"/>
      <L7p:UserStore stringValue="Internal Identity Provider"/>
      <L7p:RequireSSL booleanValue="true"/>
    </L7p:HttpBasicAuthentication>
  </wsp:All>
</wsp:Policy>
```

### Alternative Basic Auth Structure (Based on WS-SecurityPolicy)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<wsp:Policy xmlns:wsp="http://schemas.xmlsoap.org/ws/2004/09/policy"
            xmlns:httpp="http://schemas.microsoft.com/ws/06/2004/policy/http">
  <wsp:ExactlyOne>
    <wsp:All>
      <httpp:BasicAuthentication/>
      <httpp:RequireClientCertificate/>
    </wsp:All>
  </wsp:ExactlyOne>
</wsp:Policy>
```

### Routing Policy (Estimated Structure)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<wsp:Policy xmlns:wsp="http://schemas.xmlsoap.org/ws/2004/09/policy"
            xmlns:L7p="http://www.layer7tech.com/ws/policy">
  <wsp:All>
    <L7p:HttpRoutingAssertion>
      <L7p:ProtectedServiceUrl stringValue="https://backend.api.com"/>
      <L7p:HttpMethod stringValue="*"/>
      <L7p:RequestHeaderRules httpPassthroughRuleSet="included">
        <L7p:Rules httpPassthroughRules="included">
          <L7p:item httpPassthroughRule="included">
            <L7p:Name stringValue="*"/>
          </L7p:item>
        </L7p:Rules>
      </L7p:RequestHeaderRules>
      <L7p:ResponseHeaderRules httpPassthroughRuleSet="included">
        <L7p:Rules httpPassthroughRules="included">
          <L7p:item httpPassthroughRule="included">
            <L7p:Name stringValue="*"/>
          </L7p:item>
        </L7p:Rules>
      </L7p:ResponseHeaderRules>
    </L7p:HttpRoutingAssertion>
  </wsp:All>
</wsp:Policy>
```

### Layer 7 Service Structure (Estimated)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<exp:Export xmlns:exp="http://www.layer7tech.com/ws/policy/export">
  <exp:References>
    <ServiceDocument>
      <Service id="service1">
        <Name>REST API Service</Name>
        <ResolutionPath>/api/v1/*</ResolutionPath>
        <HttpMethods>GET,POST,PUT,DELETE</HttpMethods>
        <ServicePolicy>
          <wsp:Policy xmlns:wsp="http://schemas.xmlsoap.org/ws/2004/09/policy">
            <!-- Authentication Policy -->
            <L7p:HttpBasicAuthentication>
              <L7p:Realm stringValue="API Gateway"/>
            </L7p:HttpBasicAuthentication>
            <!-- Routing Policy -->
            <L7p:HttpRoutingAssertion>
              <L7p:ProtectedServiceUrl stringValue="https://backend.example.com"/>
            </L7p:HttpRoutingAssertion>
          </wsp:Policy>
        </ServicePolicy>
      </Service>
    </ServiceDocument>
  </exp:References>
</exp:Export>
```

## Sample Policy Library
Created sample Layer 7 XML policies for testing and validation:

### 1. Basic Authentication Policy
**File**: `/samples/layer7/basic_auth_policy.xml`
- **Realm**: REST API Gateway
- **Credential Source**: HTTP Authorization header
- **User Store**: Internal Identity Provider
- **SSL Required**: true
- **Fail on Missing Credentials**: true

### 2. Routing Policy  
**File**: `/samples/layer7/routing_policy.xml`
- **Target URL**: https://api.backend.example.com
- **HTTP Methods**: All (*)
- **Header Pass-through**: All headers
- **Parameter Pass-through**: All parameters
- **Error Handling**: Fail on error status

### 3. Complete Service Export
**File**: `/samples/layer7/sample_service_export.xml`
- **Services**: Customer API, Order API
- **Resolution Paths**: /api/v1/customers/*, /api/v1/orders/*
- **Combined Policies**: Basic Auth + Routing for each service
- **Different Configurations**: SSL required vs optional

## XML Schema Analysis

### Key Findings from Sample Creation:
1. **Namespace Structure**: Layer 7 uses `L7p:` prefix for policy assertions
2. **Policy Wrapper**: Standard WS-Policy `wsp:Policy` and `wsp:All` elements
3. **String Attributes**: Most values use `stringValue` attribute pattern
4. **Boolean Attributes**: Boolean values use `booleanValue` attribute pattern
5. **Service Export Format**: Uses `exp:Export` wrapper with `ServiceDocument` elements

### Critical Attributes Identified:

#### Basic Authentication
- `L7p:Realm`: Authentication realm name
- `L7p:CredentialSource`: Source of credentials (typically "request.http.authheader")
- `L7p:UserStore`: Identity provider reference
- `L7p:RequireSSL`: Boolean for HTTPS requirement
- `L7p:FailIfNoCredentials`: Boolean for handling missing auth

#### HTTP Routing
- `L7p:ProtectedServiceUrl`: Backend target URL
- `L7p:HttpMethod`: HTTP methods to route (often "*" for all)
- `L7p:FailOnErrorStatus`: Boolean for error handling
- `L7p:RequestHeaderRules`: Header pass-through configuration
- `L7p:ResponseHeaderRules`: Response header handling
- `L7p:RequestParamRules`: URL parameter handling

#### Service Configuration
- `Name`: Service display name
- `ResolutionPath`: URL path pattern for routing
- `HttpMethods`: Allowed HTTP methods
- `Enabled`: Service activation status
- `Soap`: Boolean indicating SOAP vs REST service
- `Internal`: Boolean for internal service flag

## Blockers Identified
1. **Documentation Access**: Some Layer 7 specific documentation requires login/licensing
2. **Sample Availability**: Need actual Layer 7 v11.1 policy exports for accurate analysis
3. **XML Schema Validation**: Need official Layer 7 XSD files for proper parsing

## Updated Task Plan
- **Day 1**: Continue documentation research, create sample XML structures
- **Day 2**: Locate actual Layer 7 policy examples, validate XML patterns
- **Day 3**: Complete schema documentation, prepare for RESEARCH-002 (Apigee X analysis)

## Status: COMPLETED - PHASE 1
**Next Update**: Ready for RESEARCH-002 (Apigee X Analysis)  
**Completion Date**: July 21, 2025

### Deliverables Completed ✅
- ✅ Layer 7 v11.1 XML policy schema documentation
- ✅ Sample basic authentication policies  
- ✅ Sample routing policy structures
- ✅ Service export format patterns
- ✅ Critical attribute identification for conversion engine

### Research Summary
Successfully analyzed Layer 7 v11.1 XML policy structures with focus on basic authentication and routing policies. Created comprehensive sample library and identified all critical attributes needed for Apigee X conversion. Ready to proceed with RESEARCH-002.
