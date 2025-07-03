# **Guide for Detection IDs and Versioning**

## **Introduction**

A consistent system for identifying and versioning detection analytics is the foundation of an effective and scalable detection engineering program. This document establishes simple and clear rules to ensure that everyone on your team can easily track, manage, and reference any detection.

## **1\. Detection ID System**

Every detection rule must have a unique and immutable identifier (ID). The ID serves as a permanent reference to the detection, even if its name or logic changes over time.

We recommend using a structured ID format linked to the **MITRE ATT\&CK®** framework. This immediately provides context about the behavior the detection is trying to capture.

### **ID Format**

The ID will have the following structure:

**DE-\<Tactic\_Code\>-\<Technique\_Code\>-\<NNN\>**

* **DE:** An abbreviation for "Detection Engineering." A simple prefix that identifies the document type.  
* **Tactic\_Code:** The code for the MITRE ATT\&CK Tactic that the detection primarily relates to. For example, TA0006 for Credential Access.  
* **Technique\_Code:** The code for the MITRE ATT\&CK Technique or Sub-technique. Always use the most specific code available. For example, T1003.001 for LSASS Memory. *Keep the period in the sub-technique code.*  
* **NNN:** A three-digit sequential number (starting with 001). This number ensures uniqueness in case you have multiple detection rules for the same technique or sub-technique.

### **Example of ID Creation**

Let's imagine we are creating our first detection for dumping credentials from the LSASS process.

1. **Identify the technique in ATT\&CK:**  
   * Tactic: **Credential Access** \-\> Code: TA0006  
   * Technique: **OS Credential Dumping** \-\> Code: T1003  
   * Sub-technique: **LSASS Memory** \-\> Code: T1003.001  
2. **Assemble the ID:**  
   * This is the first rule for this sub-technique, so the sequential number will be 001\.  
   * Resulting ID: **DE-TA0006-T1003.001-001**

If you were to later create a *second, different* rule for detecting LSASS dumping, its ID would be DE-TA0006-T1003.001-002.

## **2\. Versioning System**

Versioning allows us to track the history of changes in a detection rule. We use a **Major.Minor** versioning system (e.g., 1.0, 1.1, 2.0). It is crucial to know when to use a major versus a minor version change.

### Minor Version (e.g., from 1.0 to 1.1)

A minor version change is used for modifications that **do not alter the core detection logic** or its behavioral target. These changes are backward-compatible. In essence, you are improving and tuning the existing rule.

**When to use a Minor change:**

* **Adding or modifying exclusions:** You are adding a new legitimate process to the filter to reduce false positives.  
* **Refining a filter:** You are slightly adjusting a condition to be more precise (e.g., changing contains to startswith to reduce FPs).  
* **Fixing errors:** You are correcting a typo in a variable name or a comment.  
* **Improving metadata:** You are updating the description, adding a reference, or modifying the triage steps.  
* **Changing the "Status":** You are moving the rule from Test to Production.

Example:  
You have a detection v1.0 that generates several false positives from a legitimate antivirus tool. You add MsMpEng.exe to the exclusion list. The new version will be 1.1.

### Major Version (e.g., from 1.1 to 2.0)

A major version change is used for **fundamental, backward-incompatible changes**. This means you have altered the core detection logic so significantly that it is essentially a "new" rule, even if it targets the same goal. Old tests and assumptions may no longer apply.

**When to use a Major change:**

* **Changing the primary data source:** You are switching the detection from Sysmon Event ID 10 to data from an EDR (e.g., DeviceEvents). The events have a different structure and require a complete rewrite of the query.  
* **Fundamental change in detection logic:** You are changing the approach from detecting command-line arguments to monitoring API calls. For example, instead of looking for whoami.exe (a Level 3 detection), you start monitoring calls to GetTokenInformation (a Level 4/5 detection).  
* **Change in "Summiting the Pyramid" level:** Your modifications have shifted the robustness of the detection to a different level (e.g., from Level 3 to Level 4). This is such a significant change in resilience that it deserves a new Major version.  
* **Significant query rewrite:** You have rewritten the query from scratch, even if it monitors the same events, because you found a much more efficient or reliable method.

Example:  
You have a detection v1.3 that looks for service creation using sc.exe create in the command line. You decide this is too brittle. You create new logic that detects direct writes to the registry in HKLM\\SYSTEM\\CurrentControlSet\\Services\\. Even though the goal is the same (detecting service creation), the logic is completely different and more robust. The new version will be 2.0.

### **Summary**

* **ID is permanent:** Once assigned, it never changes.  
* **Minor version (1.0 \-\> 1.1):** Tuning, fixes, refinements. The core logic remains the same.  
* **Major version (1.1 \-\> 2.0):** A fundamental rewrite of the logic, change of data source, or change in robustness.
