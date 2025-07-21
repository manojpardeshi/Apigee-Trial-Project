# GitHub Copilot Instructions for Layer 7 to Apigee Migration Tool

## Project Context
You are working on a Layer 7 to Apigee migration tool that automatically converts Layer 7 API Gateway configurations to Google Apigee. This is a complex enterprise migration project requiring deep understanding of both platforms.

## Core Project Files
- **Requirements**: @memory/docs/product_requirement_docs.md
- **Task Plan**: @memory/tasks/tasks_plan.md
- **Architecture**: @memory/docs/architecture.md (when created)
- **Technical Specs**: @memory/docs/technical.md (when created)

## Development Workflow
Follow the 5-phase rulebook-ai methodology:

### 1. Requirements & Clarifications
- Always reference the PRD for context
- Ask clarifying questions about Layer 7 and Apigee specifics
- Validate assumptions against real-world API gateway patterns

### 2. Exhaustive Search & Optimal Plan
- Research Layer 7 configuration structures thoroughly
- Identify equivalent Apigee patterns and policies
- Consider edge cases and complex configurations
- Plan for extensibility and maintainability

### 3. User Validation
- Present architectural decisions clearly
- Explain Layer 7 to Apigee mapping rationale
- Validate technical approach before implementation

### 4. Implementation
- Build incrementally with testing at each step
- Focus on separation of concerns (parser, converter, deployer)
- Implement robust error handling for API gateway complexities
- Follow FastAPI and Python best practices

### 5. Further Suggestions
- Recommend optimizations and additional features
- Suggest testing strategies for complex migration scenarios
- Identify opportunities for enhanced error correction

## Technical Guidelines

### FastAPI Development
- Use modern FastAPI patterns with async/await
- Implement comprehensive input validation
- Create detailed API documentation with examples
- Handle file uploads for Layer 7 project directories

### MCP Server Development
- Follow Model Context Protocol specifications
- Implement secure Apigee API integration
- Create robust logging and error handling
- Design for extensibility to other API gateways

### Layer 7 Analysis
- Support multiple Layer 7 configuration formats
- Handle complex policy chains and routing rules
- Extract security policies and transformations
- Preserve API versioning and deployment patterns

### Apigee Generation
- Generate standard Apigee proxy structures
- Create equivalent policy flows and configurations
- Maintain API functionality and security requirements
- Ensure Maven project compatibility

### Error Handling & Auto-Correction
- Implement comprehensive error detection
- Create mapping for common migration issues
- Design self-healing deployment processes
- Provide detailed error reporting and resolution logs

## Code Quality Standards
- Write comprehensive unit tests for all components
- Use type hints throughout Python code
- Implement proper logging at all levels
- Follow enterprise security best practices
- Create detailed documentation for complex conversions

## Project-Specific Considerations
- This is an enterprise-grade migration tool, not a simple converter
- Layer 7 and Apigee have different paradigms - handle gracefully
- Consider performance for large API gateway configurations
- Plan for incremental migration scenarios
- Design for multiple customer environments and patterns

## When Working on Tasks
1. Always check current task status in @memory/tasks/tasks_plan.md
2. Update task status as you progress
3. Reference architecture decisions in @memory/docs/architecture.md
4. Update technical specifications as you learn more about requirements
5. Create detailed documentation for complex Layer 7 to Apigee mappings

## Key Success Criteria
- Generate deployable Apigee code from Layer 7 configurations
- Achieve high success rate in automated deployments
- Implement effective error detection and auto-correction
- Maintain API functionality equivalence post-migration
- Provide clear migration reports and validation

Remember: This is a complex enterprise migration project. Take time to understand both platforms deeply before implementing conversion logic.
