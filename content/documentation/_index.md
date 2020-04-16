---
title: "Documentation"
date: 2018-09-03T19:48:11+02:00
---

CogniCrypt comprises two features to assist in the usage of cryptographic APIs. First, its code generator CogniCrypt<sub>GEN</sub> may generate code wrappers around cryptographic APIs that implement programming tasks involving cryptography. Currently, CogniCrypt<sub>GEN</sub> supports code generation for five such tasks. CogniCrypt also employs a suite of static code analyses CogniCrypt<sub>SAST</sub> constantly running the background and checking for misuses of cryptographic APIs. Thanks to its tight integration with Eclipse, developers are being alerted of misuses by means of regular Eclipse error markers. Both CogniCrypt<sub>GEN</sub> and CogniCrypt<sub>SAST</sub> are parameterized and configured by rules in the specification languge CrySL. For more details on all three, please refer to their corresponding tutorial pages:

* [CrySL - Usage Specifications for Cryptographic APIs](code-generation)

* [CogniCrypt<sub>GEN</sub> - Generator for Code Examples for Cryptographic APIs](code-generation)

* [CogniCrypt<sub>SAST</sub> - Analysis for Misuses of Cryptographic APIs](code-analysis)

## Configuration

Through its preference menu, CogniCrypt may be configured in several different ways. The preferences are depicted below.

<div class="imgbox">
    <img class="center-fit" src='./preferences.png' alt="Preferences">
</div>

* `Source of CrySL Rules` : Users may select in this table which CrySL rules CogniCrypt should include in its analyis and code generation. By default, there are three rule sets, one for the JCA, one for BouncyCastle, and one for Google Tink. However, users may add new ruleset through the button below the table.

* `Select Custom Rules` : As explained [here](crysl), users may write their own custom CrySL rules in CogniCrypt directly if they do not want to specify a complete rule set. For CogniCrypt to use these custom rules, a user has to enable this option here.

* `Enable Automated Analysis when Saving` : When this option is enabled, CogniCrypt<sub>SAST</sub> executes whenever a source-code file is saved. Otherwise, the user has to trigger CogniCrypt<sub>SAST</sub> manually

* `Enable Provider-Detection Analysis` : If enabled, CogniCrypt<sub>SAST</sub> determines automatically which JCA provider is used primarily in the application under analysis and subsequently selects the appropriate  rule set. 

* `Show Secure objects` : If users wish CogniCrypt<sub>SAST</sub> to inform them not just about insecure objects, but also about secure ones, they can enable this option. Secure objects are then reported in the gutter on the left in the source-code editor as info messages. 

* `Include Dependencies to Analysis` : When enabled, CogniCrypt<sub>SAST</sub> not only checks the application code directly, but any library code the application code depends on.

* `Call-graph construction algorithm` : Users may select which algorithm CogniCrypt<sub>SAST</sub> uses for call-graph construction.

* `Error-Warning Types` : By means of these options, users may set the severity CogniCrypt<sub>SAST</sub>'s error types are reported with. They may chosse between Error, Warning, Info, or Ignore.

* `Persist Code-generation Configuration` : If enabled, CogniCrypt<sub>GEN</sub> stores the configuration it uses to generate code for a cryptographic use case into the root folder of the user's project.
