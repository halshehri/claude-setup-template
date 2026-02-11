# Bootstrap Questions Flow

This document defines the questions to ask during project setup.

## Question Flow

### Q1: Project Status
```
Is this a new (greenfield) project or an existing (brownfield) project you want to add Claude Code setup to?

Options:
- New project (greenfield)
- Existing project (brownfield)
```

**If brownfield**: Run auto-detection first, then confirm findings with user.

---

### Q2: Project Type
```
What type of project structure are you using?

Options:
- Monorepo (single repository, multiple services/packages inside)
- Multi-repo Microservices (parent folder with separate repos per service)
```

---

### Q3: Project Name
```
What is the project name?

(This will be used in documentation and file headers)
```

---

### Q4: Services/Packages

**For Monorepo:**
```
What services or packages does this project contain?

Please list them with their purpose:
Example:
- backend: REST API service
- frontend: React web application
- shared: Shared utilities and types
```

**For Multi-repo:**
```
What microservices are part of this project?

Please list them with their purpose and repo name:
Example:
- api-gateway: Request routing and auth (org/api-gateway)
- user-service: User management (org/user-service)
- frontend: React web application (org/frontend)
```

---

### Q5: Tech Stack Per Service
```
What tech stack does each service use?

For each service, specify:
- Runtime/Language (Node.js, Python, etc.)
- Framework (Express, FastAPI, React, etc.)
- Key libraries

Example:
- backend: Node.js, Express, TypeScript, Prisma
- frontend: React, TypeScript, Vite, TailwindCSS
- agents: Python, Google ADK
```

---

### Q6: Database
```
What database(s) does this project use?

Options (select all that apply):
- PostgreSQL
- MySQL
- MongoDB
- SQLite
- Redis
- None / Other
```

---

### Q7: Additional Patterns (Optional)
```
Are there any specific patterns or conventions you follow?

Examples:
- Specific folder structure
- Naming conventions
- Error handling patterns
- Testing approach

(Leave blank to use defaults)
```

---

### Q8: GitHub Organization (Multi-repo only)
```
What is your GitHub organization name?

This will be used to reference repos in documentation.
Example: my-company
```

---

## Auto-Detection (Brownfield)

Before asking questions, detect:

### Package Detection
```javascript
// Check for Node.js
if (exists('package.json')) {
  tech.push('Node.js');
  // Check for frameworks
  if (deps.includes('express')) tech.push('Express');
  if (deps.includes('fastify')) tech.push('Fastify');
  if (deps.includes('react')) tech.push('React');
  if (deps.includes('vue')) tech.push('Vue');
  if (deps.includes('next')) tech.push('Next.js');
}

// Check for Python
if (exists('pyproject.toml') || exists('requirements.txt')) {
  tech.push('Python');
  if (deps.includes('fastapi')) tech.push('FastAPI');
  if (deps.includes('django')) tech.push('Django');
  if (deps.includes('flask')) tech.push('Flask');
  if (deps.includes('google-adk')) tech.push('Google ADK');
}

// Check for Google ADK (additional detection)
if (exists('agent.py') && fileContains('agent.py', 'google.adk')) {
  tech.push('Google ADK');
}
// Also check npm deps for TypeScript ADK
if (deps.includes('@google/adk')) tech.push('Google ADK');

// Check for database
if (deps.includes('prisma') || deps.includes('pg')) db = 'PostgreSQL';
if (deps.includes('mongoose') || deps.includes('mongodb')) db = 'MongoDB';
if (deps.includes('better-sqlite3') || deps.includes('sqlite3')) db = 'SQLite';
```

### Structure Detection
```javascript
// Detect monorepo patterns
if (exists('packages/') || exists('apps/') || exists('services/')) {
  type = 'monorepo';
  services = listDirectories('packages/') || listDirectories('apps/');
}

// Detect multi-repo (look for multiple .git or no root .git)
if (!exists('.git') && subdirsHaveGit()) {
  type = 'multirepo';
}
```

### Confirmation Template
```
I've analyzed your project and found:

**Project Type**: {detected_type}

**Services Detected**:
| Service | Tech Stack | Purpose |
|---------|------------|---------|
| backend | Node.js, Express | (please describe) |
| frontend | React, Vite | (please describe) |

**Database**: {detected_db}

Is this correct? Would you like to modify anything?
```

---

## Question Response Processing

After collecting answers, create a configuration object:

```json
{
  "projectName": "my-project",
  "projectType": "monorepo",
  "isExisting": false,
  "services": [
    {
      "name": "backend",
      "purpose": "REST API service",
      "tech": {
        "runtime": "Node.js",
        "framework": "Express",
        "language": "TypeScript",
        "libraries": ["Prisma", "Zod"]
      }
    },
    {
      "name": "frontend",
      "purpose": "Web application",
      "tech": {
        "runtime": "Node.js",
        "framework": "React",
        "language": "TypeScript",
        "libraries": ["Vite", "TailwindCSS", "TanStack Query"]
      }
    }
  ],
  "databases": ["PostgreSQL"],
  "githubOrg": null,
  "customPatterns": null
}
```

This object drives template generation.
