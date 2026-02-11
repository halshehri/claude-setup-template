# {{SERVICE_NAME}}

{{SERVICE_PURPOSE}}

## Tech Stack

- **Runtime**: {{RUNTIME}}
- **Framework**: {{FRAMEWORK}}
- **Language**: {{LANGUAGE}}
- **Database**: {{DATABASE}}

## Directory Structure

```
{{SERVICE_NAME}}/
{{DIRECTORY_STRUCTURE}}
```

## Development Commands

```bash
{{INSTALL_COMMAND}}          # Install dependencies
{{DEV_COMMAND}}              # Start with hot-reload
{{BUILD_COMMAND}}            # Production build
{{TEST_COMMAND}}             # Run tests
{{LINT_COMMAND}}             # Run linter
```

## Key Responsibilities

{{RESPONSIBILITIES}}

## API Endpoints (if applicable)

| Method | Endpoint | Purpose |
|--------|----------|---------|
{{API_ENDPOINTS}}

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
{{ENV_VARIABLES}}

## Code Conventions

{{CODE_CONVENTIONS}}

## Key Files

| File | Purpose |
|------|---------|
{{KEY_FILES}}

## Integration Points

- **Communicates with**: {{INTEGRATIONS}}
- **Depends on**: {{DEPENDENCIES}}

## Testing

```bash
# Unit tests
{{UNIT_TEST_COMMAND}}

# Integration tests
{{INTEGRATION_TEST_COMMAND}}
```

## Common Tasks

### Adding a new endpoint
1. Create route in `{{ROUTES_PATH}}`
2. Add controller in `{{CONTROLLERS_PATH}}`
3. Add service logic in `{{SERVICES_PATH}}`
4. Add tests
5. Run validation

### Adding a new model
1. Create model in `{{MODELS_PATH}}`
2. Create migration (if applicable)
3. Add repository methods
4. Add tests
