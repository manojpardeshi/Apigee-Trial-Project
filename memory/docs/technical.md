# Technical Specifications Document
## Layer 7 to Apigee X Migration Tool

### Document Information
**Version**: 1.0  
**Date**: July 21, 2025  
**Status**: Draft

### 1. Technical Environment

#### 1.1 Development Stack
- **Language**: Python 3.11+
- **API Framework**: FastAPI with async/await
- **XML Processing**: lxml for robust XML parsing
- **Templating**: Jinja2 for Apigee code generation
- **HTTP Client**: httpx for Apigee X API calls
- **MCP Protocol**: Python MCP SDK for server implementation

#### 1.2 Layer 7 Source Requirements
- **Version**: Layer 7 API Gateway v11.1
- **Format**: XML policies and configurations
- **Scope**: Basic authentication and routing policies only
- **Exclusions**: TLS, certificates, caches, persistent storage, transformations

#### 1.3 Apigee X Target Requirements
- **Platform**: Google Cloud Apigee X
- **Output Format**: Maven-deployable project structure
- **Features**: Analytics and security focus
- **Environments**: Single or multiple environment support

### 2. API Specifications

#### 2.1 FastAPI Backend Endpoints

##### POST /migrate
**Purpose**: Initiate Layer 7 to Apigee X migration

**Request**:
```json
{
  "layer7_project_path": "/path/to/layer7/project",
  "target_environment": "test",
  "project_name": "my-api-migration",
  "options": {
    "enable_analytics": true,
    "auto_correct_errors": true,
    "javascript_fallback": true
  }
}
```

**Response**:
```json
{
  "migration_id": "uuid-string",
  "status": "processing",
  "message": "Migration started successfully",
  "estimated_completion": "2025-07-21T15:30:00Z"
}
```

##### GET /status/{migration_id}
**Purpose**: Check migration progress

**Response**:
```json
{
  "migration_id": "uuid-string",
  "status": "completed|processing|failed",
  "progress_percentage": 85,
  "current_step": "generating_apigee_code",
  "errors": [],
  "warnings": [
    {
      "type": "unsupported_feature",
      "message": "Complex transformation policy converted to JavaScript",
      "file": "policies/transform-request.xml"
    }
  ]
}
```

##### GET /download/{migration_id}
**Purpose**: Download generated Maven project

**Response**: ZIP file containing Maven project

#### 2.2 Layer 7 Input Specifications

##### Expected Folder Structure
```
layer7_project/
├── policies/
│   ├── auth-basic.xml
│   ├── route-backend.xml
│   └── other-policies.xml
├── services/
│   ├── api-service-1.xml
│   └── api-service-2.xml
├── folders/
│   └── folder-structure.xml
└── config/
    └── service-configs.xml
```

##### Supported Policy Types
1. **Basic Authentication Policies**
   - XML structure: `<policy type="BasicAuth">`
   - Attributes: realm, user store references
   - Conversion: → Apigee BasicAuthentication policy

2. **Routing Policies**
   - XML structure: `<policy type="RouteHttp">`
   - Attributes: target URL, method routing
   - Conversion: → Apigee RouteRule + TargetEndpoint

##### XML Policy Schema (Layer 7 v11.1)
```xml
<!-- Basic Auth Policy Example -->
<policy type="BasicAuth" version="1.1">
  <name>BasicAuthentication</name>
  <realm>API Gateway</realm>
  <userStore>internal</userStore>
  <credentialSource>header</credentialSource>
</policy>

<!-- Routing Policy Example -->
<policy type="RouteHttp" version="1.1">
  <name>RouteToBackend</name>
  <targetUrl>https://backend.api.com</targetUrl>
  <method>*</method>
  <pathPattern>/api/*</pathPattern>
</policy>
```

#### 2.3 Apigee X Output Specifications

##### Maven Project Structure
```
generated_project/
├── pom.xml
├── src/main/apigee/
│   ├── proxies/
│   │   └── {service-name}-v1/
│   │       ├── {service-name}-v1.xml
│   │       ├── policies/
│   │       │   ├── BasicAuth-1.xml
│   │       │   ├── RouteToBackend-1.xml
│   │       │   └── JS-Fallback-*.xml
│   │       ├── targets/
│   │       │   └── default.xml
│   │       └── resources/
│   │           └── jsc/
│   │               └── fallback-scripts.js
│   └── resources/
│       └── edge/env/test/
├── README.md
└── migration-report.md
```

##### Apigee Proxy Definition
```xml
<!-- Generated Proxy Bundle -->
<APIProxy name="{service-name}-v1">
  <Description>Migrated from Layer 7 v11.1</Description>
  <Flows>
    <Flow name="default">
      <Request>
        <Step><Name>BasicAuth-1</Name></Step>
      </Request>
      <Response/>
    </Flow>
  </Flows>
  <HTTPProxyConnection>
    <BasePath>/api/v1</BasePath>
    <VirtualHost>default</VirtualHost>
  </HTTPProxyConnection>
  <RouteRule name="default">
    <TargetEndpoint>default</TargetEndpoint>
  </RouteRule>
</APIProxy>
```

##### Policy Templates
```xml
<!-- Basic Authentication Policy -->
<BasicAuthentication name="BasicAuth-1">
  <User ref="request.header.username"/>
  <Password ref="request.header.password"/>
  <Source>request.header.authorization</Source>
</BasicAuthentication>

<!-- Target Endpoint -->
<TargetEndpoint name="default">
  <HTTPTargetConnection>
    <URL>{backend_url}</URL>
  </HTTPTargetConnection>
</TargetEndpoint>
```

### 3. Conversion Logic Specifications

#### 3.1 Policy Mapping Rules

| Layer 7 Policy | Apigee X Equivalent | Conversion Logic |
|----------------|-------------------|------------------|
| BasicAuth | BasicAuthentication | Direct mapping with credential extraction |
| RouteHttp | RouteRule + TargetEndpoint | URL and path pattern conversion |
| Unsupported | JavaScript Policy | Fallback with notification |

#### 3.2 Error Handling Specifications

##### Syntax Error Auto-Correction
```python
# Example auto-correction rules
syntax_corrections = {
    "malformed_xml": "attempt_xml_repair",
    "missing_attributes": "use_default_values", 
    "invalid_characters": "sanitize_and_escape",
    "encoding_issues": "force_utf8_encoding"
}
```

##### Policy Conflict Resolution
```python
# Conflict resolution strategies
conflict_resolution = {
    "duplicate_policies": "merge_with_suffix",
    "incompatible_features": "remove_and_notify",
    "resource_conflicts": "rename_with_prefix"
}
```

##### Fallback Strategies
```python
# When auto-correction fails
fallback_strategies = {
    "critical_parse_error": "exclude_file_continue",
    "unsupported_feature": "convert_to_javascript",
    "complex_logic": "create_placeholder_policy"
}
```

### 4. MCP Server Specifications

#### 4.1 MCP Tools Definition
```python
# Available MCP tools for Apigee X
@mcp.tool("generate_apigee_maven_project")
async def generate_maven_project(
    layer7_path: str,
    project_name: str,
    target_env: str = "test"
) -> Dict[str, Any]:
    """Generate Apigee X Maven project from Layer 7 source"""
    
@mcp.tool("validate_apigee_proxy")
async def validate_proxy(
    proxy_path: str
) -> Dict[str, Any]:
    """Validate generated Apigee proxy structure"""

@mcp.tool("get_conversion_report")
async def get_conversion_report(
    migration_id: str
) -> Dict[str, Any]:
    """Get detailed migration report with warnings and errors"""
```

#### 4.2 Apigee X API Integration
```python
# Apigee X Management API endpoints
APIGEE_ENDPOINTS = {
    "base_url": "https://apigee.googleapis.com/v1",
    "organizations": "/organizations/{org}",
    "proxies": "/organizations/{org}/apis",
    "deployments": "/organizations/{org}/environments/{env}/deployments",
    "logs": "/organizations/{org}/environments/{env}/logs"
}

# Authentication
AUTH_METHODS = {
    "oauth2": "google_oauth2_service_account",
    "api_key": "google_api_key_authentication"
}
```

### 5. Data Models

#### 5.1 Layer 7 Data Models
```python
@dataclass
class Layer7Policy:
    name: str
    type: str
    version: str
    attributes: Dict[str, Any]
    xml_content: str
    file_path: str

@dataclass 
class Layer7Service:
    name: str
    policies: List[Layer7Policy]
    routing_config: Dict[str, Any]
    folder_path: str
```

#### 5.2 Apigee X Data Models
```python
@dataclass
class ApigeeProxy:
    name: str
    base_path: str
    flows: List[ApigeeFlow]
    policies: List[ApigeePolicy]
    target_endpoints: List[TargetEndpoint]

@dataclass
class ApigeePolicy:
    name: str
    type: str
    configuration: Dict[str, Any]
    template_path: str
```

#### 5.3 Migration Models
```python
@dataclass
class MigrationResult:
    migration_id: str
    status: str
    source_files_processed: int
    policies_converted: int
    warnings: List[MigrationWarning]
    errors: List[MigrationError]
    output_path: str

@dataclass
class MigrationWarning:
    type: str
    message: str
    source_file: str
    suggested_action: str
```

### 6. Configuration Specifications

#### 6.1 Environment Configuration
```yaml
# config.yaml
layer7:
  supported_versions: ["11.1"]
  policy_types: ["BasicAuth", "RouteHttp"]
  max_project_size_mb: 100

apigee:
  target_platform: "apigee-x"
  naming_convention: "{service-name}-v{version}"
  max_policies_per_proxy: 20
  
conversion:
  auto_correct_syntax: true
  javascript_fallback: true
  preserve_comments: false
  
mcp:
  timeout_seconds: 300
  max_concurrent_operations: 5
```

#### 6.2 Template Configuration
```python
# Template paths and settings
TEMPLATE_CONFIG = {
    "proxy_template": "templates/proxy.xml.j2",
    "policy_templates": {
        "BasicAuth": "templates/policies/basic_auth.xml.j2",
        "RouteRule": "templates/policies/route_rule.xml.j2",
        "JavaScript": "templates/policies/javascript.xml.j2"
    },
    "maven_template": "templates/pom.xml.j2",
    "readme_template": "templates/README.md.j2"
}
```

### 7. Performance Requirements

#### 7.1 Processing Limits
- **Maximum Layer 7 project size**: 100MB
- **Maximum policies per service**: 50
- **Maximum concurrent migrations**: 5
- **Processing timeout**: 5 minutes per migration

#### 7.2 Response Times
- **Simple project (< 10 policies)**: < 30 seconds
- **Medium project (10-25 policies)**: < 60 seconds  
- **Large project (25-50 policies)**: < 120 seconds

#### 7.3 Resource Requirements
- **Memory**: 512MB per migration process
- **Disk space**: 1GB temporary storage per migration
- **CPU**: 2 cores recommended for concurrent processing

### 8. Security Specifications

#### 8.1 Input Validation
- File path sanitization (prevent directory traversal)
- XML parsing security (XXE prevention)
- File size and count limits
- Content type validation

#### 8.2 Output Security
- Generated code sanitization
- Secure temporary file handling
- Credential masking in logs
- Clean up of sensitive data

#### 8.3 Apigee Integration Security
- OAuth 2.0 service account authentication
- API key rotation support
- Rate limiting compliance
- Audit logging for all API calls

### 9. Testing Specifications

#### 9.1 Unit Test Coverage
- **Target**: 90%+ code coverage
- **Focus**: Policy parsing, conversion logic, error handling
- **Framework**: pytest with async support

#### 9.2 Integration Test Scenarios
- End-to-end migration of sample Layer 7 projects
- Error handling and auto-correction validation
- Apigee X deployment testing (via MCP)
- Performance testing with various project sizes

#### 9.3 Test Data Requirements
- Sample Layer 7 v11.1 projects with various policy combinations
- Invalid XML files for error testing
- Apigee X test environment access
- Mock Layer 7 project structures
