# Hattori Work API Documentation

This directory contains language-agnostic API specifications for the Hattori Work Agent system.

## 📁 Structure

```
hattori-work-docs/
├── README.md           # This file
├── openapi.json        # OpenAPI 3.1 spec for HTTP/REST endpoints
├── asyncapi.json       # AsyncAPI 3.0 spec for WebSocket endpoints
└── schemas/            # Shared type definitions (JSON format)
    ├── message.json    # Message-related schemas
    ├── common.json     # Common utility types
    └── chat.json       # Chat-specific schemas
```

**Note:** All files use JSON format for consistency and error prevention.

## 🎯 Purpose

These specifications serve as the **single source of truth** for API contracts between the frontend (TypeScript) and backend (Python). They enable:

- **Language-agnostic contracts**: Define APIs in YAML, generate code in any language
- **Type safety**: Generate strongly-typed interfaces for both frontend and backend
- **Documentation**: Auto-generate interactive API documentation
- **Testing**: Generate mock servers and test clients
- **Consistency**: Ensure frontend and backend never drift apart

## 📚 Quick Reference

### API Overview

**HTTP Endpoints:**
- `POST /api/chats/{user_id}/messages` - Send user messages to the agent
- `GET /api/chats/{user_id}/messages` - Retrieve message history
- `GET /health` - Health check
- `GET /` - Root endpoint

**WebSocket:**
- `WS /ws/chats/{user_id}` - Real-time notifications for assistant responses

For complete details, see [API-SUMMARY.md](./API-SUMMARY.md)

## 📚 Specifications

### OpenAPI (`openapi.yaml`)
Documents the HTTP/REST API endpoints with full request/response schemas and examples.

### AsyncAPI (`asyncapi.yaml`)
Documents the WebSocket API for real-time notifications from the Hattori Work Agent.

### Schemas
Shared type definitions that can be referenced by both OpenAPI and AsyncAPI specs.

## 🛠️ Usage

### Generate TypeScript Types

```bash
# Install generator
npm install -g @openapi-generator-plus/cli

# Generate TypeScript types from OpenAPI
openapi-generator-plus --generator typescript-fetch \
  --input openapi.json \
  --output ../hattori-work-frontend/src/types/api/

# Generate TypeScript types from AsyncAPI
npm install -g @asyncapi/generator
asyncapi generate asyncapi.yaml -o ../hattori-work-frontend/src/types/websocket/
```

### Generate Python Types

```bash
cd ../hattori-work-agent

# Install generator
pip install datamodel-code-generator

# Generate Pydantic models from OpenAPI
datamodel-codegen \
  --input ../hattori-work-docs/openapi.yaml \
  --input-file-type openapi \
  --output src/hattori_work_agent/models/openapi.py

# Generate Pydantic models from AsyncAPI (experimental)
# Most AsyncAPI generators don't have great Python support yet
```
```

### View Interactive Documentation

#### OpenAPI (Swagger UI)
```bash
# Using Docker
docker run -p 8080:8080 -e SWAGGER_JSON=/openapi.json \
  -v $(pwd):/docs swaggerapi/swagger-ui

# Or using Swagger Editor online
# https://editor.swagger.io/ (upload openapi.json)
```

#### AsyncAPI Studio
```bash
# Install globally
npm install -g @asyncapi/studio

# Start studio
asyncapi studio asyncapi.json
```

### Generate Mock Servers

```bash
# Generate mock server from OpenAPI
npx @mockoon/cli --data openapi.json

# Generate mock server from AsyncAPI
# Use AsyncAPI Studio's mock server feature
```

## 🔄 Workflow

### 1. Update API Specification

When you need to change the API:
1. Edit `openapi.yaml` or `asyncapi.yaml`
2. Run validation: `swagger-cli validate openapi.yaml`
3. Update schema files in `schemas/` if needed

### 2. Regenerate Types

**Frontend:**
```bash
cd hattori-work-frontend
npm run generate:types  # Uses openapi.json
```

**Backend:**
```bash
cd hattori-work-agent
make generate:types  # Uses openapi.json
```

### 3. Use Generated Types

Import and use the generated types in your code:

**TypeScript:**
```typescript
import { CreateMessageRequest, CreateMessageResponse } from '@/types/api';
```

**Python:**
```python
from hattori_work_agent.models.openapi import CreateMessageRequest, CreateMessageResponse
```

## 🔧 Setup Scripts

### Frontend Setup

Add to `hattori-work-frontend/package.json`:
```json
{
  "scripts": {
    "generate:types": "openapi-generator-plus --generator typescript-fetch --input ../hattori-work-docs/openapi.json --output src/types/api/"
  }
}
```

### Backend Setup

Add to `hattori-work-agent/Makefile`:
```makefile
.PHONY: generate-types
generate-types:
	datamodel-codegen \
		--input ../hattori-work-docs/openapi.json \
		--input-file-type openapi \
		--output src/hattori_work_agent/models/openapi.py
```

## 📖 Schema References

### In OpenAPI
```json
{
  "components": {
    "schemas": {
      "MyRequest": {
        "$ref": "../schemas/message.json#ChatMessage"
      }
    }
  }
}
```

### In AsyncAPI
```json
{
  "components": {
    "schemas": {
      "MyMessage": {
        "$ref": "../schemas/message.json#EnhancedChatMessage"
      }
    }
  }
}
```

## 📊 Files Status

All API specifications are now in JSON format:
- ✅ `openapi.json` - Primary HTTP/REST API specification
- ✅ `asyncapi.json` - Primary WebSocket API specification  
- ✅ `schemas/*.json` - All schema definitions in JSON
- 📝 YAML files are kept for historical reference

## ✅ Validation

Validate specifications before committing:

```bash
# Validate OpenAPI
swagger-cli validate openapi.json

# Validate AsyncAPI
asyncapi lint asyncapi.json

# Both
npm run validate:all
```

Add to `package.json`:
```json
{
  "scripts": {
    "validate:openapi": "swagger-cli validate openapi.json",
    "validate:asyncapi": "asyncapi lint asyncapi.json",
    "validate:all": "npm run validate:openapi && npm run validate:asyncapi"
  }
}
```

## 🚀 CI/CD Integration

Add validation to your CI pipeline:

```yaml
# .github/workflows/validate-api.yml
- name: Validate API specs
  run: |
    npx swagger-cli validate openapi.json
    npx asyncapi lint asyncapi.json
```

## 📝 Best Practices

1. **Keep specs synced**: Always regenerate types after changing specs
2. **Version control**: Commit changes to specs in the same PR as code changes
3. **Document breaking changes**: Use version tags in OpenAPI/AsyncAPI
4. **Use references**: Don't duplicate schema definitions - reference from `schemas/`
5. **Validate early**: Run validation before merging changes
6. **Provide examples**: Include realistic examples in all schemas

## 🔗 Resources

- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [AsyncAPI Specification](https://www.asyncapi.com/docs/specifications/latest)
- [Swagger Editor](https://editor.swagger.io/)
- [AsyncAPI Studio](https://studio.asyncapi.com/)
- [Swagger Codegen](https://swagger.io/tools/swagger-codegen/)
- [Datamodel Codegen](https://koxudaxi.github.io/datamodel-code-generator/)
