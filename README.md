# acme-mulesoft-cicd

MuleSoft CI/CD repository for Acme. It holds the Mule application source that the pipelines build, test and deploy.

## Uploaded Mule application: `northbridge-aml-api`

The `northbridge-aml-api/` folder contains a Mule 4 application that provides Anti-Money Laundering (AML) and sanctions screening for customer onboarding.

| Item | Value |
|---|---|
| Folder | `northbridge-aml-api/` |
| Artifact | `com.northbridge.bank:northbridge-aml-api:1.0.0-SNAPSHOT` (`mule-application`) |
| Mule runtime | 4.6.0 (`minMuleVersion` 4.6.0) |
| Java | 17 |
| Build | Maven, `mule-maven-plugin` 4.7.0, MUnit 3.2.0 |
| HTTP listener | `0.0.0.0`, port from `http.port` (default `8081`) |

### Endpoints

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/api/v1/aml/screen` | Runs AML screening for a legal entity and returns the result. |
| `GET` | `/mcp/tools` | MCP tool discovery. Lists the `runAmlScreening` tool and its input schema. |
| `POST` | `/mcp/tools/runAmlScreening/invoke` | MCP tool invocation. Accepts the screening input either wrapped in an `input` object or as the bare payload. |

### Request and response

Request fields: `legalName`, `registrationNumber`, `jurisdiction` (required), plus optional `beneficialOwners[]` and `directors[]`.

Response fields: `status` (`PASS`, `FAIL` or `INCONCLUSIVE`), `sanctionsHit`, `reasons[]`, `matchedSources[]`, `checkId`, `checkedAt` and `sourceRef`. `sourceRef` carries the `x-correlation-id` request header, or a generated UUID when the header is absent.

### Screening logic

The screening is a demo implementation, not a call to a real sanctions provider. The entity name, beneficial owners and directors are upper-cased and checked for these indicator words: `SANCTIONED`, `TERROR`, `BLACKLIST`, `OFAC`, `ISIS`, `ALQAEDA`.

- A missing `legalName` returns `INCONCLUSIVE`.
- Any match returns `FAIL` with `sanctionsHit: true`.
- No match returns `PASS`.

### Key files

- `src/main/mule/northbridge-aml-api.xml`: the three flows (`post-aml-screen-flow`, `mcp-tools-discovery-flow`, `mcp-aml-invoke-flow`).
- `src/main/resources/api/`: the RAML and OpenAPI contract.
- `src/main/resources/mule-artifact.properties`: default properties (`http.port`).
- `pom.xml` and `mule-artifact.json`: build and runtime descriptors.

### Connectors

`mule-apikit-module` 1.10.0, `mule4-a2a-connector` 1.1.1, `mule-mcp-connector` 1.6.0, `mule-sockets-connector` 1.2.7 and `mule-http-connector` 1.11.1.

### Build locally

```bash
cd northbridge-aml-api
mvn clean package
```

Build output (`target/`) and the local `.mule/` cache are excluded by `.gitignore`.

### Security note

This repository is public. Do not commit Anypoint credentials, Connected App secrets or tokens. Use placeholders in any `settings.xml` and keep real values in a masked pipeline store.
