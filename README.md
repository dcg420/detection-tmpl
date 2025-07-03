# **A Step-by-Step Guide to the Detection Engineering Template**

## **Introduction**

This guide will walk you through each section of the **Detection Analytic Template**. The goal of this template is not just to document a detection rule, but to guide a thought process that results in more robust, resilient, and effective detections. It's designed to make you think like an adversary and build defenses that are difficult and costly for them to bypass.

We will cover the core concepts behind each section, explaining *why* each piece of information is critical for creating, validating, and responding to a high-fidelity security alert.

## **Section 1: METADATA**

This section is for administrative tracking. It’s the first thing anyone sees and helps manage the lifecycle of the analytic.

* **Analytic Title:** Make this as clear and descriptive as possible. Instead of "LSASS Dump," use "LSASS Memory Access via OpenProcess." This immediately tells the reader *how* the technique is being detected.  
* **ID:** A unique identifier for tracking. Your team should establish a simple convention, like DE-YYYY-NNN or one based on MITRE ATT\&CK codes.  
* **Version:** Use semantic versioning (e.g., 1.0, 1.1, 2.0). Major changes (like a change in logic) get a new whole number. Minor changes (like adding an exclusion) increment the decimal.  
* **Status:** This tracks the maturity of the rule.  
  * \[x\] Experimental: Just an idea, not tested.  
  * \[ \] Test: Deployed in a non-alerting mode to gather data and assess false-positive rates.  
  * \[ \] Production: Fully deployed and generating alerts for the SOC.  
* **Author/Date Fields:** These are crucial for accountability and history. Who made it, when, and when was it last touched?

## **Section 2: DETECTION OVERVIEW**

This section sets the stage. It forces you to articulate exactly what you are trying to accomplish before you write a single line of code.

#### **Description**

* **What to write:** A 1-2 sentence summary. What is the bad thing? Why do we care?  
* **Example:** "This analytic detects attempts by a process to open a handle to and read the memory of the Local Security Authority Subsystem Service (LSASS). This is a common technique used by adversaries to dump credentials, such as password hashes and Kerberos tickets, from memory."

#### **Hypothesis**

* **Why it's important:** This is the scientific core of your detection. A strong hypothesis makes your detection testable and prevents you from just chasing random events. It defines the specific, observable attacker behavior you expect to see.  
* **How to write it:** Follow the structure: "We hypothesize that an adversary performing **\[Technique\]** will execute the **\[Procedure/Operation Chain\]**. This activity can be observed through **\[Specific Observables\]**..."  
* **Example:** "We hypothesize that an adversary performing **OS Credential Dumping (T1003.001)** will execute the **Process Enumerate \-\> Process Access \-\> Process Read** operation chain. This activity can be observed through a **non-standard process opening a handle to lsass.exe with PROCESS\_VM\_READ access rights**, which is anomalous behavior."

#### **MITRE ATT\&CK® Mapping**

* **Why it's important:** This maps your specific detection to the globally recognized framework of adversary behaviors. It helps you track your defensive coverage and understand the *tactical goal* of the behavior you are detecting.  
* **How to fill it out:** Go to the [MITRE ATT\&CK Website](https://attack.mitre.org/). Find the technique that matches your hypothesis. Include the Tactic, Technique, and Sub-technique codes.  
* **Example:**  
  * **Tactic(s):** Credential Access (TA0006)  
  * **Technique(s):** OS Credential Dumping (T1003)  
  * **Sub-technique(s):** LSASS Memory (T1003.001)

## **Section 3: ANALYTIC DETAILS (The "Summiting the Pyramid" Section)**

This is the most crucial section for building *resilient* detections. The goal is to create detections that are painful for an adversary to bypass. Think of a pyramid: at the bottom are things that are trivial for an attacker to change (like a file hash). At the top is the core behavior (the TTP), which is very difficult to change without abandoning the technique entirely. **Your goal is to build your detection as high up this pyramid as possible.**

#### **Analytic Robustness Level**

* **Why it's important:** This forces you to evaluate *how easy* your detection is to evade. You are scoring the primary observable your logic relies on.  
* **How to choose a level:**  
  * **Level 1: Ephemeral:** Are you detecting an IP address, domain name, or file hash? An attacker can change these in seconds. This is a weak, brittle detection. **Example:** Image hash \== "abc..."  
  * **Level 2: Core to Adversary-Brought Tool:** Are you detecting a specific artifact of a known hacking tool, like a default command-line flag or a unique named pipe? The attacker can often recompile or reconfigure their tool to change this. **Example:** Detecting Cobalt Strike's default pipe name \\\\.\\pipe\\msagent\_...  
  * **Level 3: Core to Pre-Existing Tool:** Are you detecting an artifact of a legitimate tool that lives on the system (a "living-off-the-land" binary or LOLBin), like powershell.exe or reg.exe? An attacker can't easily change how reg.exe works, but they can use a *different* tool or API calls to achieve the same goal. **Example:** ProcessName \== "reg.exe" AND CommandLine contains "save hklm\\sam"  
  * **Level 4: Core to Some Implementations of a (Sub-)Technique:** Are you detecting a behavior that is fundamental to *some*, but not all, ways of performing a technique? This is a strong detection. **Example:** Detecting PROCESS\_VM\_READ access to lsass.exe. An attacker *could* use a different method (like handle duplication) that doesn't require this specific access right, but it significantly raises the bar for them.  
  * **Level 5: Core to a (Sub-)Technique (Invariant Behavior):** Are you detecting a "choke point"—a behavior that is fundamental to *every* known implementation of a technique? This is the ultimate goal. **Example:** Detecting the creation of a registry key in HKLM\\...\\Schedule\\TaskCache\\Tree\\ for Scheduled Task creation (T1053.005), as this happens regardless of whether you use the GUI, schtasks.exe, or PowerShell.

#### **Data Source & Event Robustness**

This looks at *where* your data is coming from and how hard it is for an attacker to manipulate that data source.

* **Platform/Source/Event:** Be specific. Windows \-\> Sysmon \-\> Event ID 10\. This helps others understand the data requirements.  
* **Event Robustness Column (Host-Based):**  
  * **Application (A):** The log comes from the application itself (e.g., PowerShell Script Block Logging). This is useful but can sometimes be bypassed if the attacker avoids that application.  
  * **User-Mode (U):** The log comes from a standard user-mode API call. This is good, but a sophisticated attacker can sometimes use lower-level calls (syscalls) to bypass user-mode hooks.  
  * **Kernel-Mode (K):** The log comes from deep within the OS kernel. This is the most robust and hardest source for an attacker to tamper with without crashing the system. **Kernel-Mode (K) is your goal.**  
* **Event Robustness Column (Network-Based):**  
  * **Protocol Payload (P):** You're looking at the data *inside* the packets. This is weak because attackers love encryption. If the payload is encrypted, your detection is blind.  
  * **Protocol Header (H):** You're looking at the metadata of the communication (like source/destination IP, ports, protocol flags). This is often unencrypted and much harder for an attacker to change. **Header (H) is your goal.**

**Final Summiting Score:** Combine the Level and the Column, e.g., 4K. This gives you a quick, at-a-glance measure of your analytic's resilience.

## **Section 4: LOGIC & IMPLEMENTATION**

This is where the rubber meets the road.

#### **Detection Logic**

* **What to write:** The actual query for your SIEM (Splunk, Sentinel, etc.).  
* **CRITICAL:** Comment your code\! Explain *each line* of your logic. Why are you looking for that ActionType? Why that TargetProcessFileName? This turns your query from a block of text into a teaching tool for the next analyst.

#### **Known False Positives & Exclusion Strategy**

* **Why it's important:** No detection is perfect. Listing known FPs helps the SOC tune the rule and avoid alert fatigue.  
* **Exclusion Strategy:** This is key to a high-fidelity alert. Your goal is to be **surgically precise**.  
  * **BAD:** where InitiatingProcessFileName \!= "svchost.exe". (An attacker can rename their tool to svchost.exe).  
  * **GOOD:** where InitiatingProcessFolderPath \!= "C:\\Windows\\System32\\wbem\\WmiPrvSE.exe" and InitiatingProcessSigner \!= "Microsoft Windows". This is much harder to spoof.  
  * Think about the "Summiting the Pyramid" levels for your exclusions, too. An exclusion based on a signed, trusted file path is a Level 3 exclusion.

## **Section 5: VALIDATION & RESPONSE (The "Funnel of Fidelity" Section)**

A detection is useless if it hasn't been tested and if the SOC doesn't know what to do with the alert. This section is about ensuring the analytic works and is actionable. The "Funnel of Fidelity" concept is about efficiently moving from millions of raw events to a handful of confirmed, high-value incidents without getting overwhelmed.

#### **Testing Procedures**

* **Why it's important:** You MUST prove your detection is behavioral, not just a signature for one tool.  
* **How to Test:** The template lists three types of "synonyms" (a concept from Jared Atkinson's research). This just means "tools that do the same thing but work differently under the hood."  
  * **Test Case 1 (Functional Synonym):** Test with two tools that are nearly identical (e.g., Mimikatz and a simple C\# port of it). Your rule should catch both. This is a basic check.  
  * **Test Case 2 (Procedural Synonym):** Test with a tool that uses a *different underlying function* to do the same thing. For LSASS dumping, you might test Mimikatz (which uses ReadProcessMemory) against Dumpert (which uses a direct syscall). If your rule catches both, it's getting stronger.  
  * **Test Case 3 (Sub-Technical Synonym):** Test with a tool that uses a completely *different procedure* or chain of operations. For LSASS dumping, test Mimikatz against a tool that uses the handle duplication trick. If your rule can still provide context or a signal, it's very robust.  
  * **The Goal:** The more *different* types of tools you can detect, the more confident you can be that you are detecting the *behavior*, not just the tool.

#### **Triage & Investigation Steps (Manual Playbook)**

* **Why it's important:** This is the playbook for the Tier 1 SOC analyst. It tells them exactly what to do to quickly decide if an alert is worth escalating. It's how you keep the "funnel" from getting clogged with false positives.  
* **What to write:** Create a simple, numbered checklist.  
  1. **Initial Triage:** The first 60 seconds. What should they check to make a quick "good vs. evil" decision?  
  2. **Investigation:** The next 5-10 minutes. If it looks suspicious, what contextual queries should they run? Who should they talk to?

#### **Response & Remediation**

* **What to write:** If the alert is confirmed malicious, what are the immediate actions? This is for the incident responder. Isolate host, reset creds, preserve evidence.

## **Section 6: AUTOMATION & RESPONSE PLAYBOOK (SOAR)**

This section operationalizes your detection. It defines a structured, machine-readable plan for automated systems (like a SOAR platform) to perform initial enrichment and response, saving valuable analyst time.

#### **Alert Trigger Condition & Severity**

* **Trigger:** This is the entry point for the playbook. Simply state that the playbook runs when the detection logic is met.  
* **Severity:** Define a default severity (High, Medium, Low). This can be dynamically changed by the automated triage logic.

#### **Enrichment Steps**

* **What to do:** List the automated data gathering steps. The goal is to collect all relevant context and present it to the analyst in one place.  
* **Action:** The name of the automation function (e.g., Get-UserDetails).  
* **Input:** The entity from the alert to enrich (e.g., event.AccountName).  
* **Output:** The new information you want to add to the ticket (e.g., user.title).

#### **Triage Logic (Automated)**

* **Why it's important:** This is where you automate the initial decision-making. You can automatically lower the priority of known benign events or raise the priority of high-risk events.  
* **How to write it:** Use simple IF \[condition\] \-\> THEN \[action\] logic. The condition should be based on the enriched data. The action can be changing the alert severity or flagging it for a specific team.

#### **Containment Steps**

* **What to do:** Define high-impact, automated response actions.  
* **CRITICAL:** These actions (like isolating a host or disabling a user) should almost always require manual approval (Execute: false). Set Execute: true only for the highest-confidence, highest-impact detections where the risk of a false positive is extremely low and the risk of attacker dwell time is extremely high.

#### **Notification Steps**

* **What to do:** Define how the alert and its enriched data are formally communicated. This typically involves creating a ticket in a system like Jira or ServiceNow and sending an email or chat message to the response team.

## **Section 7: FRAMEWORK MAPPINGS**

This section helps place your detection into broader strategic defense and engagement frameworks.

* **MITRE Engage™:** A framework for adversary engagement and deception. How could you use this detection to actively engage an adversary? You can find codes on the [Engage website](https://engage.mitre.org/).  
* **D3FEND:** A knowledge base of defensive countermeasures. It helps map your specific detection to a standardized defensive technique. You can find codes on the [D3FEND website](https://d3fend.mitre.org/).

By following this guide, you will not only document your detections thoroughly but also improve the quality, robustness, and effectiveness of your entire detection engineering program.

Based on: [Jared Atkinson](https://medium.com/@jaredcatkinson) and [MITRE Summiting the Pyramid](https://ctid.mitre.org/projects/summiting-the-pyramid)
