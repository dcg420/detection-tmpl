### **1\. METADATA**

* **Analytic Title:** \[Clear, descriptive title of the detection rule\]  
* **ID:** \[e.g., DE-TA0000-T0000.000-001\]  
* **Version:** 1.0  
* **Status:** \[ \] Experimental \[ \] Test \[ \] Production  
* **Author:** \[Your Name/Team\]  
* **Date Created:** YYYY-MM-DD  
* **Date Modified:** YYYY-MM-DD

### **2\. DETECTION OVERVIEW**

**Description:**

\[A brief, high-level summary of the detection's purpose. What threat or behavior is this designed to catch? Why is it important?\]

**Hypothesis:**

\[e.g., We hypothesize that an adversary performing **\[MITRE ATT\&CK Technique\]** will execute the **\[Procedure/Operation Chain\]**. This activity can be observed through **\[Specific Observables\]**...\]

**MITRE ATT\&CK® Mapping:**

* **Tactic(s):** \[e.g., Credential Access (TA0006)\]  
* **Technique(s):** \[e.g., OS Credential Dumping (T1003)\]  
* **Sub-technique(s):** \[e.g., LSASS Memory (T1003.001)\]

### **3\. ANALYTIC DETAILS (Summiting the Pyramid)**

**Analytic Robustness Level (Select One & Justify):**

*Justification:* \[Explain why this level was chosen. Is the detection based on an invariant behavior, a specific tool artifact, or something *in between?\]*

* \[ \] Level 1: Ephemeral  
* \[ \] Level 2: Core to Adversary-Brought Tool  
* \[ \] Level 3: Core to Pre-Existing Tool  
* \[ \] Level 4: Core to Some Implementations of a (Sub-)Technique  
* \[ \] Level 5: Core to a (Sub-)Technique (Invariant Behavior)

**Data Source & Event Robustness:**

* **Platform:** \[e.g., Windows, Linux, Network\]  
* **Source:** \[e.g., EDR, Sysmon, Zeek, Windows Event Log\]  
* **Event(s):** \[e.g., Sysmon Event ID 10, DeviceEvents ActionType\]  
* **Event Robustness Column (Select One):**  
  * **Host-Based:** \[ \] Application (A) \[ \] User-Mode (U) \[ \] Kernel-Mode (K)

  *Justification:* \[Explain why this column was chosen, e.g., *"The event is generated by the OS Kernel..."\]*

  * **Network-Based:** \[ \] Protocol Payload (P) \[ \] Protocol Header (H)

**Final Summiting Score:** \[e.g., 4K, 3U\]

### **4\. LOGIC & IMPLEMENTATION**

**Detection Logic (Pseudocode or SIEM Query):**

\-- Provide the query here.  
\-- Comment heavily to explain each logical step of the query.

// Example Structure:  
// 1\. Define the primary data source and core event type.  
// 2\. Add filters for the primary behavioral indicators.  
// 3\. Add filters to reduce noise (this is the classification step).  
// 4\. Implement the exclusion strategy for known good activity.

**Known False Positives:**

* \[List any legitimate activities or tools that may trigger this alert.\]

**Exclusion Strategy:**

\[Describe the strategy for filtering out false positives. Focus on robust, context-rich attributes (e.g., signed binaries from specific paths) rather than easily-spoofed values (e.g., process names alone).\]

### **5\. VALIDATION & RESPONSE (Funnel of Fidelity)**

**Testing Procedures:**

\[Describe how the rule was validated. The goal is to confirm behavioral coverage.\]

* **Test** Case **1 (Functional Synonym):**  
  * **Tool/Procedure:** \[e.g., Execute Tool A\]  
  * **Result:** \[ \] Detected \[ \] Not Detected  
* **Test Case 2 (Procedural Synonym):**  
  * **Tool/Procedure:** \[e.g., Execute Tool B, which uses different API calls for the same operations\]  
  * **Result:** \[ \] Detected \[ \] Not Detected  
* **Test** Case **3 (Sub-Technical Synonym):**  
  * **Tool/Procedure:** \[e.g., Execute Tool C, which uses a different procedure for the same sub-technique\]  
  * **Result:** \[ \] Detected \[ \] Not Detected

**Triage & Investigation Steps (Manual Playbook):**

\[Provide a clear, step-by-step checklist for the responding analyst.\]

1. **Initial Triage (Is this a true positive?):**  
   * \[Check 1: e.g., Identify the user and host.\]  
   * \[Check 2: e.g., Cross-reference with change management or helpdesk tickets.\]  
2. **Investigation (What is the context?):**  
   * \[Step 1: e.g., Review command-line history on the host.\]  
   * \[Step 2: e.g., Look for related network or file events.\]

**Response & Remediation:**

* \[List the immediate, standard response actions if the activity is confirmed malicious, e.g., Isolate host, reset credentials.\]

### **6\. AUTOMATION & RESPONSE PLAYBOOK (SOAR)**

This section provides a structured playbook for automated systems to perform enrichment, containment, and notification actions.

**Alert Trigger Condition:**

* IF 'detection\_logic' RETURNS 'true'

**Alert Severity:**

* Default: \[High, Medium, or Low\]

**Enrichment Steps:**

* **Action:** \[e.g., Get-UserDetails\]  
  * **Input:** \[e.g., event.AccountName\]  
  * **Output:** \[e.g., user.title, user.department\]  
* **Action:** \[e.g., Get-HostDetails\]  
  * **Input:** \[e.g., event.DeviceName\]  
  * **Output:** \[e.g., host.os, host.tags\]  
* **Action:** \[e.g., Lookup-Hash\]  
  * **Input:** \[e.g., event.SHA256\]  
  * **Source:** \[e.g., VirusTotal\]  
  * **Output:** \[e.g., hash.reputation\]

**Triage Logic (Automated):**

* IF \[condition\_1\] \-\> SET 'Severity' to '\[New Severity\]' AND '\[Action\]'  
* IF \[condition\_2\] \-\> SET 'Severity' to '\[New Severity\]' AND '\[Action\]'

**Containment** Steps (Requires **'Critical' Severity):**

* **Action:** \[e.g., Isolate-Host\]  
  * **Input:** \[e.g., event.DeviceName\]  
  * **Execute:** \[true or false (requires approval)\]  
* **Action:** \[e.g., Disable-User\]  
  * **Input:** \[e.g., event.AccountName\]  
  * **Execute:** \[true or false (requires approval)\]

**Notification Steps:**

* **Action:** \[e.g., Create-Ticket\]  
  * **System:** \[e.g., ServiceNow / Jira\]  
  * **Assignee:** \[e.g., SOC Tier 2\]  
  * **Content:** \[Summary of enriched alert and actions taken.\]  
* **Action:** \[e.g., Send-Email\]  
  * **Recipient:** \[e.g., soc-alerts@yourcompany.com\]  
  * **Subject:** \[CRITICAL/HIGH Alert: {{Analytic Title}} on {{host.name}}\]  
  * **Content:** \[Summary of enriched alert.\]

### **7\. FRAMEWORK MAPPINGS**

**MITRE Engage™ Mapping:**

* **Goal:** \[e.g., Disrupt (G0009)\]  
* **Approach:** \[e.g., Detect (A0001)\]

**D3FEND Mapping:**

* **Tactic:** \[e.g., Detect (D3-DET)\]  
* **Technique:** \[e.g., Process Spawn Analysis (D3-PSA)\
