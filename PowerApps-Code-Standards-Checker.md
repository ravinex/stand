# 🧩 Power Apps Code Standards Checker

## 🎯 Objective
Ensure all Microsoft Power Apps (Canvas Apps) comply with defined coding and naming standards automatically — for example, verifying that *all control names follow camelCase conventions* and adhere to organisational best practices.

---

## 🧰 Tooling Overview

| Tool / Feature | Purpose | Notes |
|----------------|----------|-------|
| **App Checker** | Built-in Power Apps validation tool | Detects formula, performance, accessibility, and delegation issues. Does **not** enforce naming conventions. |
| **Power Apps Code Review Tool** | Microsoft-provided review solution | Allows unpacking `.msapp` files, viewing control lists, formulas, and enforcing configurable code rules. |
| **Power Apps Language Tooling** | Enables unpacking `.msapp` to source files (YAML/JSON) | Provides structured metadata for controls, screens, and properties — ideal for static analysis. |
| **Custom Naming Validation Script** | Validates naming conventions (camelCase, prefixes, etc.) | Developed internally (PowerShell/Python/TypeScript). Integrates with CI/CD pipeline. |

---

## 🧾 Example Rule: Control Naming (camelCase)

**Rule Definition:**
> All controls must be named using lower camel case, optionally with type prefixes.

| Element | Example | Description |
|----------|----------|-------------|
| Button | `btnSubmit` | Lowercase prefix for control type + Pascal suffix |
| Text Input | `txtUserName` | camelCase with type prefix |
| Gallery | `galCustomerOrders` | camelCase with descriptive suffix |
| Screen | `scrHomePage` | Prefix + Pascal-style suffix |

**Regex Rule:**
```regex
^[a-z][A-Za-z0-9]*$
```

This ensures:
- First character is lowercase  
- No spaces or underscores  
- Capital letter begins each new word  

---

## ⚙️ Automated Validation Steps

### Step 1. Define Standards
Create a document specifying:
- Control naming pattern  
- Variable naming pattern  
- Collection naming pattern  
- Screen/component naming pattern  
- Formula length and complexity limits  

---

### Step 2. Export or Unpack Canvas App
Unpack `.msapp` file using Power Apps Language Tooling or the Code Review Tool.

**Command Example (PowerShell):**
```powershell
pac canvas unpack --msapp "MyApp.msapp" --source "src"
```

This creates structured files under `/src/` including control metadata and properties.

---

### Step 3. Run Naming Checker Script

Example (PowerShell / Python):

```powershell
# PowerShell sample to validate camelCase control names
Get-ChildItem -Path "src" -Recurse -Filter *.json | ForEach-Object {
    $json = Get-Content $_.FullName | ConvertFrom-Json
    foreach ($control in $json.Controls) {
        if ($control.Name -notmatch '^[a-z][A-Za-z0-9]*$') {
            Write-Host "❌ Invalid control name: $($control.Name)"
        }
    }
}
```

---

### Step 4. Integrate into CI/CD

- Add a “**lint**” step in your DevOps pipeline to run the naming script.
- Fail the build if violations are found.
- Store logs in your pipeline artifacts or GitHub Actions summary.

**Example YAML (Azure DevOps):**
```yaml
- script: pwsh ./scripts/Check-NamingConvention.ps1
  displayName: 'Validate PowerApps Naming Conventions'
```

---

### Step 5. Review and Remediate

Use:
- **Power Apps Code Review Tool** for visual inspection.
- **CI reports** to highlight offending controls.
- **Peer reviews** before deployment to production.

---

## 🧩 Example Violations Report

| Control Name | Expected Format | Status |
|---------------|-----------------|--------|
| `Submit_Button` | `btnSubmit` | ❌ Invalid (underscore) |
| `Gallery1` | `galCustomerList` | ❌ Missing prefix |
| `btnSubmit` | ✅ | ✅ Valid |
| `txtuserName` | `txtUserName` | ❌ Incorrect casing |

---

## 📦 Integration with Source Control
When storing apps in unpacked source format:
- Commit the unpacked folder (`src`) to Git.
- Run naming checks in PRs using GitHub Actions.
- Add rule configuration under `.lintconfig.json`.

Example `.lintconfig.json`:
```json
{
  "rules": {
    "camelCaseControlNames": true,
    "prefixMap": {
      "button": "btn",
      "textInput": "txt",
      "gallery": "gal"
    }
  }
}
```

---

## 🚀 Benefits

- Enforces consistency across all apps and environments  
- Improves maintainability and readability  
- Reduces production defects caused by inconsistent control references  
- Supports automated code reviews and ALM compliance  

---

## 🔗 References

- [Power Apps Code Review Tool (Microsoft)](https://www.microsoft.com/en-us/power-platform/blog/2021/03/30/power-apps-code-review-tool/)
- [Power Apps Source Code Tooling](https://learn.microsoft.com/en-us/power-platform/developer/power-apps/cli/reference/pac-canvas)
- [Power Apps Naming Conventions – PlatformsofPower](https://platformsofpower.net/power-apps-standards-guidelines-best-practices/)
- [Power Platform ALM Best Practices](https://learn.microsoft.com/en-us/power-platform/alm/)

---

**Author:** Power Platform Architecture & Governance Team  
**Version:** 1.0  
**Last Updated:** November 2025
