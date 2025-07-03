# **Evolving Detection: Why We're Moving Beyond ADS to a New Engineering Template**

For years, the **Alerting and Detection Strategy (ADS)** framework, pioneered by Palantir, has been a cornerstone of mature detection programs. It brought a much-needed structure to a field that often relied on ad-hoc scripts and institutional knowledge. It taught us the importance of documenting our goals, our technical context, and our response procedures. For that, it deserves immense credit.

However, the threat landscape and our understanding of detection have evolved. Adversaries are more sophisticated, our data sources are richer, and our response capabilities are increasingly automated. Simple documentation is no longer enough. We need a framework that doesn't just describe a detection but actively *engineers* it for resilience, actionability, and scale.

That is why we are adopting a new **Detection Analytic Template**, a framework designed for the modern Security Operations Center (SOC). This isn't just a replacement for ADS; it's an evolution. It's a shift from documentation to engineering.

We are also thrilled to announce that the **MISP Project** will be adopting this new structure to replace the existing "ADS" object in a future release, standardizing this next-generation approach for threat intelligence sharing across the community.

You can find the template on our GitHub repository here: [**dcg420/detection-tmpl**](https://github.com/dcg420/detection-tmpl)

## **Built on a Modern Philosophical Foundation**

Our template is not an arbitrary collection of fields. It is the practical application of two of the most important concepts in modern detection engineering:

1. **Capability Abstraction (Jared Atkinson, SpecterOps):** The idea that we must detect the adversary's underlying *operations* (like Process Access or Service Create), not just the specific tools or functions they use. An attacker can easily rename mimikatz.exe, but they cannot easily perform credential dumping without accessing the LSASS process memory. Our template forces us to think at this deeper, more abstract level. You can find Jared's essential work on his [Medium page](https://medium.com/@jaredcatkinson).  
2. **Summiting the Pyramid (MITRE CTID):** This framework provides a standardized, quantifiable way to measure the *robustness* of a detection. It asks, "How much pain does this cause the adversary?" By integrating this scoring system directly into our template, we make resilience a mandatory design consideration, not an afterthought. Learn more at the [Center for Threat-Informed Defense](https://ctid.mitre.org/projects/summiting-the-pyramid).

## **New Template vs. ADS: A Head-to-Head Comparison**

Let's break down why this new template represents a significant leap forward from the traditional ADS framework.

### **1\. From Narrative Context to Quantifiable Robustness**

* **ADS:** The "Technical Context" section is a free-form narrative. It relies on the engineer to explain, in prose, why the detection is effective. This is subjective and difficult to standardize or compare across different rules.  
* **New Template:** We've replaced this with the **Analytic Details (Summiting the Pyramid)** section. This is not optional. The engineer *must* score their detection on a scale from 1 (brittle) to 5 (invariant) and justify their choice. They must also score the robustness of the data source itself (Application, User-Mode, or Kernel-Mode).  
  * **The Result:** A detection with a score of 3K is immediately understood to be more resilient than one scored at 1A. This creates a culture of accountability and a clear target for engineers to aim for.

### **2\. From General Response to Actionable, Automated Playbooks**

* **ADS:** The "Response" section is a single, manual guide for an analyst. It often mixes enrichment, triage, and remediation steps, and is not designed for machines.  
* **New Template:** We split this into two distinct, highly structured sections:  
  * **Manual Playbook:** A clear, numbered checklist for the human analyst, focusing on triage and investigation.  
  * **SOAR Playbook:** A machine-readable blueprint for automation. It defines triggers, enrichment actions (Get-UserDetails), automated triage logic (IF user.risk\_score \== 'High'), and containment steps. This directly bridges the gap between detection and automated response.

### **3\. From Vague Goals to Testable Hypotheses**

* **ADS:** The "Goal" section describes the detection's purpose. While useful, it doesn't enforce a scientific methodology.  
* **New Template:** We lead with a **Hypothesis**. This small change has a massive impact. It forces the engineer to frame their detection as a testable statement about adversary behavior (e.g., "We hypothesize an adversary will execute the Process Access \-\> Process Read operation chain..."). This makes the detection's logic clearer and its validation process more rigorous.

### **4\. From Basic Validation to Evasion-Aware Testing**

* **ADS:** The "Validation" section simply asks the engineer to confirm the rule works. This often leads to a single test case: running the exact tool the rule was designed to catch.  
* **New Template:** Our "Testing Procedures" section demands more. Inspired by the concept of "synonyms" in attacker tradecraft, it requires testing against different implementations of the same behavior. Can your rule detect not just procdump.exe, but also a custom tool that uses a different API to achieve the same outcome? This validates that you are detecting the *behavior*, not just a signature of a single tool.

### **5\. Built for Scale and a Shared Future**

* **ADS:** A collection of ADS documents can become an unmanageable "data swamp" of text files. It's difficult to query, report on, or manage programmatically.  
* **New Template:** The structured format, combined with a standardized ID system (e.g., DE-TA0006-T1003.001-001), is built for scale. You can programmatically parse your entire library of detections to answer questions like, "What is our average robustness score for the Credential Access tactic?"  
* **MISP Integration:** The future of threat intelligence is shared, structured data. The fact that the MISP Project will be replacing its ADS object with this new template structure is a testament to its utility. By adopting this template now, you are aligning your detection program with the future of collaborative defense.

## **It's Time to Engineer Our Detections**

The ADS framework helped us start the journey. It taught us to document. Now, it's time to take the next step. It's time to engineer.

This new template provides the framework to do just that. It pushes us to build more resilient, context-rich, and automated defenses that are truly painful for our adversaries to bypass. It transforms detection from an art into a science.

We invite you to explore the template on [**GitHub at th3r3d/bm-template**](https://www.google.com/search?q=https://github.com/th3r3d/bm-template), use it, critique it, and help us build the next generation of detection engineering.
