---
title: "Contributors Guide"
date: 2018-09-03T19:48:11+02:00
---

There are several opportunities for non-project-members to contribute to CogniCrypt:
* CogniCrypt users may contribute **Bug Reports** and **Feature Requests**,
* Regular developers who take interest in CogniCrypt may provide **bug fixes** and other kind of **code contributions**,
* Lastly, cryptography experts may share their own **cryptographic components** so that they can be used by CogniCrypt users.

# Bug Reports and Feature Requests <a name="bugs"></a>

In case you find a bug or have an idea for a feature, please check out [CogniCrypt's issue tracker](https://github.com/CROSSINGTUD/CogniCrypt/issues). If your problem or request is not yet being addressed there, please file a [new issue](https://github.com/CROSSINGTUD/CogniCrypt/issues/new/choose).

# Code Contributions <a name="code"></a>

Any contributions - no matter whether they are bug fixes or new features - are very welcome. That being said, since CogniCrypt is an official Eclipse project, only official committers may contribute directly to the repository without any review. Commits from non-committers are possible, but must be reviewed by someone with committer status and come in a certain form, described by the [Eclipse manual](https://www.eclipse.org/projects/handbook/#resources-commit). In short, apart from the usual author tag, each commit needs to come with a signed-off-by tag at the bottom of the commit message. Pull requests that contain commits from non-committers not following this structure may not be accepted.
For that matter the following steps must be done:
1.    Create an account on [Eclipse Foundation](https://accounts.eclipse.org/) with the same E-mail address as your Git account.
2.    Complete the form and apply for an ECA license for your account.
3.    Sign each commit with your e-mail address “git commit -s -m “….”.

For more information please check [this](https://wiki.eclipse.org/Development_Resources/Contributing_via_Git).

Note: Regular long-time contributors who wish to continue contributing may apply to project-committer status.


# Cryptographic Components

In general, crypto experts can contribute in two ways. First, they can contribute support for new cryptographic primitives (e.g., encryption schemes, key agreement algorithms, digital signature algorithms). Second, they can integrate new cryptographic use cases for CogniCrypt to offer to the users for code generation.


In the context of CogniCrypt, these use cases are called **tasks**. Examples for cryptographic tasks that CogniCrypt already supports  include:  **file encryption**, **communication over secure channel**, and **user authentication mechanisms**. Depending on how a task is integrated, it can use primitives that have been integrated before. However primitives are not directly exposed to the end user. For easier distinction, this guide separates:
* Contributing crypto experts into: **primitives developers** and **task developers**. 
* CogniCrypt users are being refered to as **developers**.

In order to integrate a new primitive/task, two components need to be provided: 

* Implementation
* Usage rules

The implementation should encompass the full functionality of the primitive/task. To both [prevent developers from misusing a primitive/task](../documentation/code-analysis) and [facilitate the code generation for tasks](../documentation/code-generation), usage rules describe how a task and/or a primitive is supposed to be used. 

## Integrating Primitives <a name="prim"></a>

### Implementation:

Primitives need to be implemented as [Cryptographic Service Providers (CSP)](https://docs.oracle.com/javase/9/security/howtoimplaprovider.htm#JSSEC-GUID-C485394F-08C9-4D35-A245-1B82CDDBC031) in Java. The JDK does not simply provide a set of implementations of cryptographic primitives to its users. Instead, a set of standardized and algorithm-specific interfaces is provided by the [Java Cryptography Architecture(JCA)](https://docs.oracle.com/javase/9/security/java-cryptography-architecture-jca-reference-guide.htm#JSSEC-GUID-2BCFDD85-D533-4E6C-8CE9-29990DEB0190). These interfaces must then be implemented by CSPs in order to provide cryptographic primitives to an application developer. With this design, the architecture around cryptography is made both independent of algorithms and implementations, as well as easily extensible. It enables primitives developers to implement their own algorithms as CSPs and plug them into the JCA. One CSP may include multiple primitives of multiple algorithm types.

The JCA comes with a [number of default providers](https://docs.oracle.com/javase/9/security/oracleproviders.htm#JSSEC-GUID-FE2D2E28-C991-4EF9-9DBE-2A4982726313) that implement the most common cryptographic algorithms in several configurations. [BouncyCastle](https://www.bouncycastle.org/java.html), another cryptographic library for Java, can also be used [as a CSP](https://www.bouncycastle.org/wiki/display/JA1/Provider+Installation). It implements a wide range of cryptographic algorithms and configurations, including the ones implemented by the default CSPs.

During runtime, the Java Virtual Machine (JVM) maintains an ordered list of all plugged-in CSPs. As illustrated in the figure below, when an algorithm in a certain configuration is requested by some application code, the JVM iterates through the list and asks each provider if it supports the requested algorithm and configuration. In case several CSPs support the same configuration, the implementation of the first CSP in the list to support the configuration is selected.

![JCA plug and play](https://docs.oracle.com/javase/9/security/img/architecture-service-provider-interface.gif)

### Usage Rules:
Furthermore, developers of primitives should provide usage rules for their CSP. Any component may be misused and each and every such misuse within a security context leads to a potential security vulnerability. To illustrate the necessity for such rules, consider the following two examples. The code snippet in the Listing below is a simple example of a symmetric encryption using the JCA. The call in *Line 2* instantiates an object of class `Cipher`, which is using the symmetric block cipher **AES**. Although not obvious at first glance, method `getInstance()` does in fact not expect a single algorithm, but instead a`transformation` consisting of **cipher**, **mode of operation** and **padding scheme**. If the latter two are not provided by the developer (as in the code snippet below), the exact behaviour depends on the selected CSP. The default CSP of the JCA selects ECB as mode of operation by default. Unfortunately, in ECB mode, identical plaintext blocks are encrypted to identical ciphertext blocks. Consequently, any encryption of more than one block using this configuration is deemed insecure. To prevent such misuses, contributors may specify how to use their CSP correctly and securely.

    public byte[] encrypt(byte[] secret, SecretKey key) {
        Cipher ciph = new Cipher.getInstance("AES"); 
        ciph.init(Cipher.Encrypt_Mode, key);
        return ciph.doFinal(secret);
    }

The usage rules must be provided in the definition language CrySL. CrySL alows a primitives developer to specify how their CSP should be used. Please refer to the [CrySL documentation](../documentation/crysl) for further reference. 

## Integrating Tasks <a name="tasks"></a>

Tasks are the main way application developers using CogniCrypt interact with the tool. When a user wants to implement a cryptographic task (e.g., communicate over a secure channel, encrypt data using a password), they can open the tool and select the respective task. CogniCrypt then guides them through a set of questions, lets them select one combination of algorithms and algorithm configurations, and generates the appropriate code.

For this to work properly, task developers are required to contribute three components. In addition to the two general components - implementation and CrySL rules - they also have to provide the high-level questions that CogniCrypt asks the user in the beginning to configure the generated code. To simplify the extension of the model and the development of the questions, these steps may be conducted collaboratively with the CogniCrypt maintainers. Shape and form of all of these components depend on the exact way the task is being integrated.

### Implementation

The implementation for a task may be provided to CogniCrypt as source code or a jar file. In this case, application developers get direct access to the code. If task developers do not wish to share the source code, they may provide access to the task's functionality indirectly through an API. Suppose a task developer wants to offer long-term document archiving as a cryptographic task in CogniCrypt. An archive requires the documents to be stored on a hard-disk. To not require the user to manage these files on their own, the task developer offers their archive as a web service that stores the documents on their server. In this case, the task developer does not have to provide any implementation directly to CogniCrypt. A task developer may choose to let the application developer configure the task in terms of used algorithms among other things. This configuration is done through the dialogue system after the CogniCrypt user selects a task they want to implement.


### Usage Rules 

Task developers must further provide usage rules for this component. CogniCrypt then automatically and continuously applies static analyses based on these rules in the background to ensure the code stays secure. These usage rules have to be provided in the specification language [CrySL](../documentation/crysl).

###  Configuration Questions

Lastly, if developers want to allow the end user of CogniCrypt to configure their task implementation, they are also required to provide the questions CogniCrypt can ask the user. These questions may influence the generated code as well as the involved usage rules.
