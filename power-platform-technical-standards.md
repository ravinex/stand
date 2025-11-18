# Power Platform Technical Standards  
_Design, Development, Testing & Deployment_

## 1. Scope & Principles

**Applies to:**

- Power Apps (Canvas & Model-driven)  
- Dataverse (tables, plugins, custom APIs)  
- Power Automate flows  
- SharePoint & SPFx  
- Copilot Studio / Agents (and other copilots built on Power Platform)

**Guiding principles**

1. **Solution-first**  
   All assets (apps, flows, plugins, agents) must live in **Power Platform solutions**.

2. **Environment strategy**  
   Minimum lifecycle:
   - `DEV → TEST/UAT → PROD` (optionally `SANDBOX/PLAYGROUND` for experiments).

3. **Security by design**  
   Use **Dataverse security roles, teams, field-level security (FLS)** and **SharePoint permissions** instead of custom security logic in apps.

4. **Governed extensibility**  
   All custom code (plugins, SPFx, APIs) follows shared coding standards, security practices, and CI/CD gates.

5. **Quality gates**  
   No deployment to TEST/PROD without passing **checklists** in sections 6–8.

---

## 2. Naming Conventions

Use consistent, descriptive names across the platform.

### 2.1 Solutions & Environments

**Solutions**

- Pattern:  
  `"[Domain] – [Capability]"`

- Examples:
  - `Customer – Onboarding`
  - `Operations – Case Management`

**Environments**

- Pattern:  
  `"[Org]-[Domain]-[Stage]"`

- Examples:
  - `ORG-Platform-Dev`
  - `ORG-Platform-Test`
  - `ORG-Platform-Prod`

---

### 2.2 Dataverse

| Artifact       | Pattern                                      | Example                               |
|----------------|----------------------------------------------|---------------------------------------|
| Table          | `prefix_[EntityName]`                        | `cm_Case`, `ops_Request`              |
| Column         | `prefix_[entity]_[AttributeName]` or `[prefix]_[name]` | `cm_case_ReferenceNumber`, `ops_dueDate` |
| Relationship   | `prefix_[From]_[To]`                         | `cm_Case_Account`                     |
| Business rule  | `[Table] – [Purpose]`                        | `cm_Case – Auto set Priority`         |

---

### 2.3 Power Apps (Canvas)

| Element    | Pattern               | Example                  |
|------------|-----------------------|--------------------------|
| Screen     | `scr[Name]`           | `scrHome`, `scrDetails` |
| Component  | `cmp[Name]`           | `cmpHeader`             |
| Gallery    | `gal[EntityPlural]`   | `galCases`              |
| Text input | `txt[Meaning]`        | `txtSearch`, `txtEmail` |
| Dropdown   | `dd[Meaning]`         | `ddStatus`              |
| Variable (global) | `gbl[Name]`    | `gblCurrentUser`        |
| Variable (context) | `ctx[Name]`   | `ctxSelectedRecord`     |
| Collection | `col[Plural]`         | `colStatuses`           |

---

### 2.4 Power Automate Flows

- Pattern:  
  `"[System] – [Process] – [Trigger]"`

**Examples**

- `CustomerOnboarding – Send Welcome Email – When Account Created`  
- `CaseManagement – Update SLA – Recurrence Daily`

---

### 2.5 SharePoint & SPFx

**SharePoint**

- Site: `"[Domain] [Purpose]"`  
  - `Operations Documents`, `Customer Content`
- Library: `"[Purpose]"`  
  - `Policies`, `CaseFiles`

**SPFx Web Parts**

- Pattern:  
  `"spfx-[Domain]-[Feature]"`  
  - `spfx-operations-case-header`

---

### 2.6 Agents / Copilots

- Agent name:  
  `"Agent – [Domain/Capability]"`  
  - `Agent – Customer Support`  
  - `Agent – Reporting & Analytics`

- Topic/scene name (if applicable):  
  `"T-[Area]-[Intent]"`  
  - `T-Cases-Create`, `T-Reporting-MonthlySummary`

---

## 3. Design Standards

### 3.1 Data Storage Decision

**Use Dataverse when:**

- Data is **relational**, **secured**, **audited**, or **transactional**.
- You need:
  - Model-driven apps  
  - Business rules  
  - Plugins / custom APIs  
  - Advanced querying and row-level security

**Use SharePoint when:**

- You primarily manage:
  - **Documents**
  - Basic lists
  - Intranet-style content and pages

**Use external stores (SQL, Data Lake, etc.) when:**

- You handle:
  - High-volume analytical data
  - Large historical logs
  - Heavy reporting or ML scenarios

**Typical pattern**

- Core business entities (customers, cases, requests, approvals) → **Dataverse**  
- Supporting documents (forms, letters, attachments) → **SharePoint** library with a column storing the **Dataverse row ID** or business key.

---

### 3.2 Security Design

**Rules**

- Use **Azure AD groups** to assign:
  - Environment access
  - Dataverse security roles

- Use **Dataverse security roles** for:
  - Table-level access (organization, business unit, user/team)
  - Sensitive fields via field-level security (FLS)

- Use **SharePoint permissions** and groups for document access.

**Generic security roles**

- `AppName – User`  
  - User-level create/read on main tables.
- `AppName – Manager`  
  - Business-unit-level read, user-level write.
- `AppName – Admin`  
  - Organization-level read/write.

**Sharing**

- Prefer **Teams (owner/access)** in Dataverse to ad-hoc per-user sharing.

---

### 3.3 Integration Design

**Rules**

- Wrap external integrations behind:
  - **Power Automate** flows, or  
  - **Azure Functions / Web APIs** + **Custom connectors**

- Do **not**:
  - Call internal APIs directly from Canvas apps or flows without a properly configured **custom connector** and DLP approval.

**Pattern**

- External system:
  - Exposed via secure API or Function.
  - Accessed via a **custom connector**.
  - Connection reference managed per environment.

---

### 3.4 Agent / Copilot Design

Each agent must define:

1. **Domain & scope**
   - What it can and cannot answer.
2. **Data sources**
   - Dataverse tables, SharePoint sites, Power BI models, other APIs.
3. **Security assumptions**
   - It must respect the user’s access rights.
4. **Escalation path**
   - What to do for out-of-scope or uncertain questions.

**System prompt template**

```text
You are the [AgentName] for [Domain].

You:
- Answer questions only about [AllowedDomains] using [DataSources].
- Respect the user's security context and only show data they are allowed to see.
- If you are unsure or the question is out of scope, say so clearly and direct the user to [SupportChannel] or the appropriate system.
- Do not provide HR, legal, medical, or personal financial advice.
```

---

## 4. Coding Standards

### 4.1 Power Apps – Canvas

**Rules**

1. Use naming conventions from **section 2.3**.
2. Keep formulas readable:
   - Avoid more than 3–4 nested functions.
   - Extract repeated logic into components or helper functions.
3. Prefer **delegable queries**:
   - Avoid iterating (`ForAll`) over large datasets whenever possible.
4. Do not hard-code:
   - Environment-specific URLs
   - IDs
   - Secrets

**Example (gallery Items property)**

```powerfx
With(
    {
        src: '[@YourTable]'
    },
    Filter(
        src,
        (IsBlank(txtSearch.Text) 
            || StartsWith(PrimaryNameField, txtSearch.Text))
        && (IsBlank(ddStatus.Selected.Value)
            || Status = ddStatus.Selected.Value)
    )
)
```

---

### 4.2 Model-driven Apps & Dataverse Plugins

**Rules**

- Group related tables, forms, views, and plugins into logically scoped **solutions**.
- Use a **clean architecture** approach for plugin code:
  - `Domain` project for business logic.
  - `Infrastructure` project for Dataverse services.
  - `Plugins` project for plugin implementations.

- Avoid:
  - Long-running synchronous plugins.
  - UI-dependent logic in plugins (plugins should be business-logic only).

**Naming conventions (examples)**

- Plugin assembly: `[AppName].Plugins`
- Plugin classes: `[Entity][Event][Purpose]`
  - `CaseCreateSetDefaults`, `RequestUpdateEnforceRules`
- Domain services: `[AppName].Domain.Services.[ServiceName]`

---

### 4.3 Power Automate Flows

**Rules**

- Use naming pattern from **section 2.4**.
- Use **environment variables** for:
  - API URLs
  - External IDs
  - Email addresses for notifications

- Error handling:
  - Use `Scope` actions and configure **run after** to handle failures.
  - Enable **retry policies** on HTTP actions and connectors where appropriate.

- Secrets:
  - Stored in Key Vault or secure environment constructs.
  - Never in expressions, comments, or static configuration fields.

---

### 4.4 SharePoint & SPFx

**Rules**

- Use **PnP provisioning** or site templates for repeatable configuration of:
  - Sites
  - Lists
  - Libraries
  - Content types

- SPFx projects:
  - Use **TypeScript strict** mode.
  - Enforce **ESLint** and **Prettier**.
  - No inline secrets, tokens, or environment-specific URLs.

**Typical structure**

```text
src/
  webparts/
    [featureName]/
      [FeatureName]WebPart.ts
      components/
        [FeatureName].tsx
        I[FeatureName]Props.ts
        I[FeatureName]State.ts
      services/
        SharePointService.ts
```

---

### 4.5 Agents / Copilots

**Rules**

- Topics/scenes naming:  
  `"T-[Area]-[Intent]"` (e.g., `T-Reporting-MonthlySummary`)

- Use **system messages** to:
  - Limit scope
  - Enforce security
  - Define tone and escalation paths

- Prefer:
  - **Connectors and actions** for data access
  - Avoid arbitrary HTTP requests when a connector pattern is available and approved.

---

## 5. Testing Standards

### 5.1 Unit Testing

**Canvas Apps**

- Use **Test Studio** for:
  - Core journeys (Create, Read, Update, Delete, Search, Submit).
- Tests should:
  - Navigate to the correct screen.
  - Populate controls.
  - Trigger the action (e.g., Submit button).
  - Assert expected outcomes (gallery/list updates, labels, error messages).

**Plugins & Custom Code**

- Use test frameworks (xUnit, NUnit, MSTest).
- Each important rule must have:
  - At least one **happy path** test.
  - At least one **edge/negative** test.

---

### 5.2 Integration & System Testing

**Flows**

- Maintain test scripts specifying:
  - Input data.
  - Expected system behavior.
  - Verification steps (rows created/updated, emails, notifications).

**Apps**

- For critical scenarios, consider:
  - **Playwright**, **Selenium**, or **EasyRepro** for end-to-end testing.

**Agents**

- Maintain **test sets** (where supported) with:
  - Representative user questions.
  - Expected answer shape (not exact wording, but content and constraints).

---

### 5.3 Non-functional Testing

**Performance**

- Define baseline SLOs:
  - Example: key screens load within 3 seconds under normal load.
- Use platform monitoring tools (Monitor, App Insights, etc.) to identify bottlenecks.

**Telemetry**

- Log:
  - Errors
  - Long-running operations
  - Key user journeys
- Use:
  - Application Insights
  - Power Platform analytics
  - Custom logging in plugins/APIs

---

## 6. Code Review & Pull Request Standards

All changes must go through **Pull Requests (PRs)** with at least one reviewer (ideally two: technical + functional).

**Standard PR template (example)**

```markdown
# Summary
- [Short description of the change]

## Type of change
- [ ] New feature
- [ ] Bug fix
- [ ] Refactor
- [ ] Configuration only (no code)

## Components impacted
- [ ] Canvas app
- [ ] Model-driven app
- [ ] Dataverse (tables, views, plugins)
- [ ] Power Automate / Logic App
- [ ] SharePoint / SPFx
- [ ] Agent / Copilot
- [ ] Other: __________________

## Standards
- [ ] Naming conventions applied (solutions, tables, columns, flows, screens, controls).
- [ ] No secrets or environment-specific values in code/flows.
- [ ] Security implications reviewed (roles/permissions unchanged or updated appropriately).

## Testing
- [ ] Unit tests added/updated and passing.
- [ ] Canvas Test Studio tests updated and passing (if applicable).
- [ ] Flow test cases executed and verified.
- [ ] Agent/copilot test sets rerun; results acceptable.

## Quality Gates
- [ ] Solution Checker executed (no critical/high issues).
- [ ] App Checker executed (no errors).
- [ ] Linting and build passed for any SPFx/custom code.
```

---

## 7. Deployment & ALM Standards

### 7.1 Environment Promotion

**Rule:** All deployments to TEST/PROD occur via **automated pipelines** (Azure DevOps, GitHub Actions, Pipelines for Power Platform). Manual imports into PROD are not allowed.

**Typical lifecycle**

1. **DEV**
   - Developers work in feature branches.
   - Unmanaged solutions exported as part of CI.
   - Solution Checker runs as a quality gate.

2. **TEST/UAT**
   - Managed solutions imported via pipeline.
   - Environment variables and connection references set.
   - Smoke tests executed automatically.

3. **PROD**
   - Triggered only after:
     - TEST/UAT success
     - Formal approval
   - Same managed solution artefact deployed.
   - Post-deployment smoke tests performed.

---

### 7.2 Versioning

- Use **Semantic Versioning**:  
  `MAJOR.MINOR.PATCH` (e.g., `1.4.2.0`).
- Increment:
  - **MAJOR** for breaking changes.
  - **MINOR** for new features.
  - **PATCH** for bug fixes.

---

### 7.3 Rollback Strategy

- Maintain at least the last **2–3 managed solution versions** per environment.
- Document rollback steps:
  1. Import previous managed version.
  2. Re-apply environment variables / connection references if needed.
  3. Run regression tests or smoke tests.

---

## 8. Governance Checklists

These checklists can be used as Confluence templates, DevOps work item checklists, or QA forms.

### 8.1 Design Gate Checklist

_Complete before build starts._

- [ ] Data storage choice documented (Dataverse vs SharePoint vs other).
- [ ] Security model documented (roles, teams, FLS, SharePoint permissions).
- [ ] Integration endpoints identified and patterns agreed (custom connectors/APIs).
- [ ] Agent/copilot scope and guardrails defined (if applicable).
- [ ] Non-functional requirements documented (performance, volumes, retention).

---

### 8.2 Build Gate Checklist

_Complete before merging to main/release._

- [ ] Naming conventions followed (solutions, tables, columns, flows, screens, controls).
- [ ] No secrets hard-coded (using Key Vault/env variables/connection references).
- [ ] Canvas App Checker run; errors fixed, warnings reviewed.
- [ ] Solution Checker run; no Critical/High severity issues.
- [ ] Unit tests for plugins/custom code passing.
- [ ] SPFx/custom code linting and builds passing.

---

### 8.3 Test Gate Checklist

_Complete before deploying to TEST/PROD._

- [ ] Functional test cases created and executed for key scenarios.
- [ ] Canvas Test Studio tests updated and re-run (where applicable).
- [ ] Flow test scripts executed and results recorded.
- [ ] Agent/copilot test sets run; unexpected behaviours reviewed.
- [ ] Regression tests run on TEST/UAT; sign-off recorded.

---

### 8.4 Release Gate Checklist

_Complete before PROD deployment._

- [ ] PR approved by required reviewers (technical + product/BA where appropriate).
- [ ] CI pipeline (build + tests + checkers) passed.
- [ ] Managed solution version recorded and tagged.
- [ ] UAT sign-off captured (e.g., DevOps work item, Confluence page).
- [ ] Deployment executed via pipeline only; no manual solution import.
- [ ] Monitoring & alerts configured/updated (App Insights, CoE dashboards, platform analytics).

---

_This document is intended as a baseline. Teams can extend it with application-specific details, but must not weaken the minimum standards defined here._
