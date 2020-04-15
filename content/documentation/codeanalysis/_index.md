---
title: "Static Code Analysis"
date: 2018-09-03T19:48:11+02:00
---

CogniCrypt's static analysis CogniCrypt<sub>SAST</sub> automatically runs on the code within Eclipse. The static analysis is based on `CrySL rules` that specify the *correct* use of an application programming interface (API). `CrySL` is a domain-specific language that allows to specify usage patterns of APIs. The static analysis reports any deviations from the usage pattern defined within the rules. 

While the `CrySL` rules are adjustable, a user of CogniCrypt is not expected to change the rules of CogniCrypt. 

## Eclipse Error Markers and their Meanings

CogniCrypt generates errors markes when the analysis detects incorrect and insecure parts of code. CogniCrypt displays error markers within the Eclipse IDE to warn the developer about insecure code. The error markers are associated to the respective line in the editor the errors is located at. 

There are various different error types that CogniCrypt reports. Below, we distinguish the error types based on the warning the error marker reports.

* `"MD5 should be any of {SHA-256, SHA-384, SHA-512}"` is a **Constraint Error**: the static analysis detects an incorrect `String` (or `int`) to flow as argument to a method call. CogniCrypt automatically suggest alternatives to fix the issue. The error message describes that `MD5` should be replaced by and of the other `String` elements.  

* `"Unexpected call to method reset. Expect a call to one of the following methods digest,update"` marks a **Typestate Error**. The sequence of object calls made on an object is not according to its `CrySL` specification.

* `"Operation with Cipher object not completed. Expected call to update, doFinal."` marks an **Incomplete Operation Error**. An incomplete operation errors appears, when a call on an object is missing and the object is garbage collected without having properly used. A typically example for such an error is a missing call to `close` on a `FileWriter`.

* `"Variable keyBytes was not properly randomized"` is called a **Required Predicate Error**. Such an error is reported when the analysis infers that the combination of using *several* object is incorrect. The error message reports that the developer uses a variable `keyBytes` (containing some `byte[]`) for some cryptographic operation. The correct specification of the API requires that the `byte` array has been previously `randomized` (wich means correctly generate by some API).
