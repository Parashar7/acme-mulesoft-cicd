# northbridge-aml-api

Anti-Money Laundering (AML) screening API for Northbridge Bank, built on MuleSoft 4.  
Exposes a REST interface (APIkit-driven) for transaction screening and alert routing.

## Runtime requirements

| Item | Value |
|---|---|
| Mule runtime | 4.6.0 |
| Java | 17 |
| HTTP listener port | 8081 |
| Target platform | Anypoint CloudHub 2.0 |

## Connectors

| Connector | Version | Purpose |
|---|---|---|
| mule-apikit-module | 1.10.0 | REST routing from RAML spec |
| mule-http-connector | 1.11.1 | Outbound HTTP calls |
| mule-sockets-connector | 1.2.7 | Low-level socket transport |
| mule4-a2a-connector | 1.1.1 | Agent-to-agent messaging |
| mule-mcp-connector | 1.6.0 | MuleSoft Control Plane integration |

Versions are centrally managed by `../mule-parent-pom/pom.xml`.

## Local build

Requires JDK 17 and Maven 3.9+.

```bash
# Build and package (no deploy)
mvn -f northbridge-aml-api/pom.xml package -s settings.xml

# Build and run MUnit tests
# Note: no test files exist yet under src/test/munit — MUnit exits with 0 tests.
mvn -f northbridge-aml-api/pom.xml verify -s settings.xml

# Validate that the parent POM resolves correctly
mvn -f northbridge-aml-api/pom.xml validate
```

Run from the **repo root** so that `settings.xml` and the relative parent path
`../mule-parent-pom/pom.xml` both resolve.

## Configuration

### Required before deploying

1. Fill in the Anypoint org ID in `mule-parent-pom/pom.xml` (property `anypoint.org.id`)  
   and in the workflow env var `ANYPOINT_ORG_ID` (`.github/workflows/deploy-northbridge-aml-api.yml`).
2. Create the GitHub Environments `sandbox` and `production` in the repo settings.
3. Add a required reviewer to the `production` environment.
4. Set the secrets listed below in each environment.

### Secrets (per GitHub Environment)

| Secret name | Description |
|---|---|
| `ANYPOINT_CLIENT_ID` | Connected App client ID for this environment |
| `ANYPOINT_CLIENT_SECRET` | Connected App client secret for this environment |

Create a separate Connected App for `sandbox` and `production` in  
Anypoint Platform → Access Management → Connected Apps.

## CI/CD pipeline

The workflow at `.github/workflows/deploy-northbridge-aml-api.yml` runs on:
- Merge to `main`
- `workflow_dispatch`
- Pull requests to `main` (build and MUnit only — no secrets, no deploy)

### Pipeline stages

```
build-and-test  →  publish-to-exchange  →  deploy-sandbox  →  deploy-production
```

| Stage | What it does |
|---|---|
| Build and MUnit | Compiles, packages, runs MUnit; uploads the jar as a workflow artifact |
| Publish to Exchange | Downloads the pre-built jar; publishes to Anypoint Exchange under the org ID |
| Deploy → Sandbox | Downloads the pre-built jar; deploys to CloudHub 2.0 `Sandbox` environment |
| Deploy → Production | Downloads the same jar; deploys to CloudHub 2.0 `Production` environment (gated: required reviewer) |

Production always deploys the same artifact built in the first job — no rebuild.

### Replica and vCore sizing (`[TBD]`)

The workflow defaults to `CH2_REPLICAS=1` and `CH2_VCORES=0.1`.  
Update these env vars in the workflow file before the first production deploy.
