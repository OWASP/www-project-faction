---
title: faction_appstore_extensions
layout:  null
tab: true
order: 2
tags: reporting, vulnerability management, pentesting, java, application-security
---

## Faction App Store Extensions

![image](/www-project-faction/assets/images/faction-appstore.png)

Faction supports server-side extensions, similar in concept to BurpSuite extensions. An extension is a Java JAR that implements one or more Faction Extender interfaces, packaged with a small set of resource files, and uploaded through the Faction App Store UI. Faction invokes the extension when specific events fire — for example, an assessment being finalized, a report being generated, or a retest being completed.

Typical use cases: pushing vulnerabilities into an external tracker (Jira, ServiceNow), querying an external CMDB during assessment scheduling, or injecting custom HTML (charts, tables) into generated reports.

Available since Faction 1.2. Example project: <https://github.com/factionsecurity/Faction-Jira-Extension>

## Extension Hooks

An extension class extends `BaseExtension` and implements one of the following interfaces.

| Interface | Triggers on | Receives | Returns |
|-----------|-------------|----------|---------|
| `com.faction.extender.ApplicationInventory` | Assessment scheduling | Application ID or name | `InventoryResult[]` from an external source (replaces local DB lookup) |
| `com.faction.extender.AssessmentManager` | Assessment create / update / delete / finalize, peer review created / complete / accepted | `Assessment`, `List<Vulnerability>`, `Operation` | `AssessmentManagerResult` with updated assessment and vulns (or `null` for no local update) |
| `com.faction.extender.ReportManager` | Report create or regenerate | `Assessment`, `List<Vulnerability>`, current `reportText` | Modified report text (HTML or raw); `null` leaves the report unchanged |
| `com.faction.extender.VulnerabilityManager` | Vulnerability create / update / delete | `Assessment`, `Vulnerability`, `Operation` | Updated `Vulnerability` (or `null` for no local update) |
| `com.faction.extender.VerificationManager` | Retest pass / fail / cancel / assigned | `User`, `Vulnerability`, comment, start date, end date, `Operation` | void |

The `Operation` enum lets a single hook differentiate between events — for example, only acting on `Operation.Finalize` inside `AssessmentManager`.

## Project Layout

Extensions are built as Maven projects. Add the Faction Extender dependency to `pom.xml`:

```xml
<dependency>
    <groupId>com.factionsecurity</groupId>
    <artifactId>faction-extender</artifactId>
    <version>2.5</version>
</dependency>
```

Use `maven-assembly-plugin` with `jar-with-dependencies` so the JAR includes all transitive dependencies. The manifest must include `Title`, `Version`, `Author`, and `URL` entries — these are surfaced in the App Store UI.

Required resource files:

```
src/main/resources/META-INF/
├── resources/
│   ├── config.json        # User-configurable settings
│   ├── description.md     # Help text shown in App Store
│   └── logo.png           # Icon shown in App Store
└── services/
    └── com.faction.extender.<InterfaceName>   # Service descriptor
```

### `config.json`

Defines the settings the user can edit in the App Store UI. Each entry produces an input field. Values defined here are defaults; users can overwrite them after install.

```json
{
  "Jira Host":    { "type": "text",     "value": "https://yourhost.com" },
  "Jira API Key": { "type": "password", "value": "your api key" },
  "Jira Email":   { "type": "text",     "value": "your@email.com" }
}
```

`type` must be `text` or `password`. `password` fields are masked in the UI and are not returned to the UI after save.

### `description.md`

Markdown describing what the extension does, why a user would install it, and how to configure it. Rendered in the App Store detail view.

### `logo.png`

PNG icon shown in the App Store listing.

### `services/` descriptor file

The filename **must** be the fully-qualified interface name your extension implements — one of:

- `com.faction.extender.ApplicationInventory`
- `com.faction.extender.AssessmentManager`
- `com.faction.extender.ReportManager`
- `com.faction.extender.VerificationManager`
- `com.faction.extender.VulnerabilityManager`

The file contents are the fully-qualified class name of your implementation, e.g. `org.faction.JiraPlugin`. Without this file, Faction cannot load the extension.

An extension that hooks multiple events includes one service descriptor file per interface implemented.

## Minimal Example

A Jira extension that pushes vulnerabilities to Jira only when an assessment is finalized, and writes the returned issue IDs back to Faction as tracking IDs:

```java
package org.faction;

public class JiraPlugin extends BaseExtension implements com.faction.extender.AssessmentManager {

    @Override
    public AssessmentManagerResult assessmentChange(Assessment assessment,
                                                    List<Vulnerability> vulns,
                                                    Operation opcode) {
        String project = "KAN"; // Jira project key

        if (opcode == Operation.Finalize) {
            for (Vulnerability vuln : vulns) {
                String issueId = sendVulnerabilityToJira(vuln, project);
                if (issueId != null) {
                    vuln.setTracking(issueId); // Sync Jira ID back to Faction
                }
            }
        }

        AssessmentManagerResult result = new AssessmentManagerResult();
        result.setAssessment(assessment);
        result.setVulnerabilities(vulns);
        return result; // Returning null would skip the Faction-side update
    }
}
```

Full reference implementation: <https://github.com/factionsecurity/Faction-Jira-Extension>

## Build and Install

Build the JAR:

```bash
mvn clean compile assembly:single
```

This produces `target/<artifact>-<version>-jar-with-dependencies.jar`.

In Faction, go to **Admin → App Store → Install Extension** and upload the JAR. The extension installs in a disabled state. Click the extension in the list, fill in the settings defined by `config.json`, and enable it. The hook will fire on its registered events from that point on.

## Reference

- [App Store Extension API](https://docs.factionsecurity.com/APIS/App%20Store%20Extension%20API/) — hook interface reference.
- [JIRA App Integration Example](https://docs.factionsecurity.com/APIS/JIRA%20App%20Integration%20Example/) — full walkthrough.
- [Faction-Jira-Extension](https://github.com/factionsecurity/Faction-Jira-Extension) — example project source.
- [Faction-Vulnerability-Bar-Chart](https://github.com/factionsecurity/Faction-Vulnerability-Bar-Chart) — example `ReportManager` extension that injects HTML bar charts into reports.
