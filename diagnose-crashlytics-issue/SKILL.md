---
name: diagnose-crashlytics-issue
description: >-
  Review a Firebase Crashlytics issue using the Firebase MCP to understand the crash report, identify the root cause within the codebase, and suggest possible solutions.
disable-model-invocation: true
metadata:
  version: "1.0"
---

# Process

1. Accept a Firebase Crashlytics issue URL or issue ID. Resolve the Firebase project and app from the supplied information or available context. If the issue or its project/app context is ambiguous, ask the user for the missing details before retrieving the repor
2. Using the Firebase MCP, review the crash report for the shared ID. Check associated logs and breadcrumbs for additional insight. If the MCP is not working, stop immediately and ask the user to resolve the issue. Let the user know what went wrong so they have enough information to resolve the issue on your behalf.
3. Identify the affected app versions and builds, and map them to source commits or release tags. Inspect the relevant revisions in a separate worktree to preserve existing work. If the mapping cannot be established, state this limitation and request the missing release information.
4. Compare the crash report against the affected app version to identify the most likley root cause. Distinguish confirmed findings from hypotheses, cite supporting evidence, and explain any uncertainty.
5. Use the commit history to investigate which changes may have introduced the issue. Identify a confirmed introducing commit only when supported by evidence; otherwise, report candidate changes or state that the introducing change could not be determined.
6. Complete the crash report template below and share it in the current conversation.
7. Identify different solutions to the problem and share them with the user in the current conversation. Explain how and why it addresses the likely cuase.

Use “Unknown” when a value cannot be determined and “Unavailable” when the supporting source cannot be accessed. Do not invent missing details. Mark inferred screens or app areas as inferred and explain the supporting evidence. For empty sections, briefly state whether nothing was identified, the section is not applicable, or the investigation was blocked.

If stack trace frames are obfuscated (e.g., com.example.a.b.c), inform the user that a ProGuard/R8 mapping file or Proguard mapping setup might be missing in Crashlytics.

Where applicable, include a unit or integration test scenario to verify that the proposed solution prevents the issue from reoccurring.

# Crash report template

<crash-report-template>

# Crash report

|             |                                                                 |
| ----------- | --------------------------------------------------------------- |
| **Issue**   | [**{{issue_id}}**]({{issue_url}})                                 |
| **App**     | `{{package_name}}` ({{app_name}})                                |
| **Type**    | {{severity}} `{{exception_type}}`                                |
| **Title**   | `{{crash_title}}`                                               |
| **Seen in** | {{affected_versions_and_builds bullet points}}; {{other_version_observations}} |
| **Volume**  | {{event_count}} events, {{install_count}} installs; {{device_and_os_observations}} |
| **Affected Screens** | {{bullet-point list of screens or areas of the app affected}} |

# Stack Trace

# Root Cause

Provide an outline of the root cause with supporting evidence. This may include code snippets.

# Contributing Factors

A bullet-point list of zero or more factors that contribute to the issue.

# Issue Tracker Links

Links to any relevant issues on project trackers. These may include the Android Open Source Project.

# Other Notes

</crash-report-template>