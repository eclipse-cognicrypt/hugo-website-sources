---
title: ""
date: 2018-09-03T19:48:11+02:00
---


# General Contributions

* Eclipse stuff


# New Components by Crypto Developers


In general, crypto experts can contribute in two ways. First, they can contribute support for new primitives (e.g., encryption schemes, key agreement algorithms, digital signature algorithms). Second, they can integrate new cryptographic tasks that can be offered to users of OpenCCE. In this guide, the term cryptographic task refers to a more or less complex component that involves one or multiple cryptographic primitives. Examples for cryptographic tasks include file encryption, communication over secure channel, and user authentication mechanisms. Depending on how a task is integrated into CogniCrypt, it can use primitives that have been integrated before, but primitives are not directly exposed to the end user. For easier distinction, this guide separates contributing crypto experts into primitives developers and task developers.

In order to integrate a new primitive/task, four components need to be provided. 
* Implementation
* Usage example
* CrySL rules
* Model describing the involved algorithms

The implementation should encompass the full functionality of the primitive/task. The usage example in general represents the way the functionality of the primitives/task should be accessed. To prevent developers from misusing primitive/task, usage rules in the definition language CrySL describe how they are supposed to be used. These rules are translated into static analyses that are run when the application developer is writing the code, not only once it is run. This enables developers to correct their mistakes before deploying their application. These rules may also help application developers who do not use CogniCrypt's code-generation functionality, but want to implement a cryptographic task themselves as the static analyses might still alert them to API misuses. Finally, the model describes the used cryptographic algorithms and the attributes relevant to determine their security level. Throughout, the documentation shows only excerpts of the model. Contributors who are interested in a different part or the full model are referred to the CogniCrypt maintainers.

## Integrating Primitives

### Implementation

Primitives need to be implemented as Cryptographic Service Providers (CSP) in Java. The JDK does not simply provide a set of implementations of cryptographic primitives to its users. Instead, a set of standardized and algorithm-specific interfaces is provided by the Java Cryptography Architecture(JCA). These interfaces must then be implemented by CSPs in order to provide cryptographic primitives to an application developer. The goal of this design is to make the architecture around cryptography both independent of algorithms and implementations as well as easily extensible. It enables primitives developers to implement their own algorithms as CSPs and plug them into the JCA. One CSP may include multiple primitives of multiple algorithm types.

The JCA comes with a number of default providers that implement the most common cryptographic algorithms in several configurations. BouncyCastle, another cryptographic library for Java, can also be used as a CSP. It implements a wide range of cryptographic algorithms and configurations, including the ones implemented by the default CSPs.

During runtime, the Java Virtual Machine (JVM) maintains an ordered list of all plugged in CSPs. As illustrated in Figure~\ref{fig:JCAOverview}, when an algorithm in a certain configuration is requested by some application code, the JVM iterates through the list and asks each provider if it supports the requested algorithm and configuration. In case several CSPs support the same configuration, the implementation of the first CSP in the list to support the configuration is selected.

Oracle provides extensive documentation on both [JCA and CSPs](https://docs.oracle.com/javase/8/docs/technotes/guides/security/crypto/CryptoSpec.html) as well as on [how to implement a Cryptographic Service provider](https://docs.oracle.com/javase/8/docs/technotes/guides/security/crypto/HowToImplAProvider.html). Readers who wish to implement their primitives in C/C++ are referred to Chapter TODO: Add Link.

### Usage Example
During the code generation, CogniCrypt  generates code into the application developer's project, that uses a CSP to perform a cryptographic operation (e.g. encryption, computation of a MAC). To ensure a secure usage of the implemented CSP, primitives developers should first provide a code snippet as a secure and correct usage example. OpenCCE can then use this snippet as the code it generates into the project as is shown in TODO: Fix Listing. The Listing illustrates a simple encryption with Java Crypto APIs that avoids usual pitfalls (as described below).

    public byte[] encrypt(byte[] secret, SecretKey key) {
        byte [] ivb = new byte[16];
        SecureRandom.getInstanceStrong().nextBytes(ivb);
        IvParameterSpec iv = new IvParameterSpec(ivb);
    
        Cipher c = Cipher.getInstance("AES/CBC/PKCS5Padding");
        c.init(Cipher.ENCRYPT_MODE, key, iv);
        return c.doFinal(secret);
    }

### CrySL Rules
Furthermore, developers of primitives should provide usage rules for their CSP, which CogniCrypt transforms into static analyses. The static analyses then makes sure the application developer's code is not breaking any of the specified usage rules.

To illustrate the necessity for such rules, consider the following two examples. The code snippet in Listing~\ref{lst:enc} is a simple example of a symmetric encryption using the JCA. The call in Line \ref{line:AES} instantiates an object of class `Cipher`, which is using the symmetric block cipher AES. Although not obvious at first glance, method `getInstance()` does in fact not expect a single algorithm, but instead a`transformation` consisting of cipher, mode of operation and padding scheme. If the latter two are not provided by the developer (as in the code snippet below), the exact behaviour depends on the selected CSP. The default CSP of the JCA selects ECB as mode of operation by default. Unfortunately, in ECB mode, identical plaintext blocks are encrypted to identical ciphertext blocks. Consequently, any encryption of more than one block using this configuration is deemed insecure. To prevent such misuses, contributors may specify how to use their CSP correctly and securely.  TODO: Add paragraph on CrySL

    public byte[] encrypt(byte[] secret, SecretKey key) {
        Cipher ciph = new Cipher.getInstance("AES"); 
        ciph.init(Cipher.Encrypt_Mode, key);
        return ciph.doFinal(secret);
    }

### Model

In addition to implementation and usage rules, the algorithms that are to be integrated into CogniCrypt need to be modelled in the variability modelling language Clafer. This is necessary since developers of tasks might not implement their tasks by using algorithms directly (e.g., an encryption using RSA) but may only specify certain requirements an algorithm needs to comply to (e.g., encryption with an asymmetric cipher with a minimal key size of 4096 bit) and leave the selection of the actual algorithm to the user of CogniCrypt. The algorithms hence work as basic building blocks for the supported tasks. In this case, CogniCrypt determines a list of potential algorithms and configurations based on the variability model. 

In general, the model consists of all algorithms and their configurations CogniCrypt supports at a given point in time.  An abbreviated model is shown in TODO: Fix Listing. Lines TODO to TODO  define the algorithm classes cipher and asymmetric cipher and their attributes. To integrate lattice-based schemes into this model, all lines in green need to be added. This extension can be separated into three different kinds of changes. First, class Cipher gets an additional attribute `quantum` (see line TODO) to specify whether an algorithm is pre- or post-quantum. Second, a new class `LatticeBasedCipher` and its attributes are added (see lines TODO to TODO). This class is a subclass of `AsymmetricCipher` and inherits all its attributes. Third, from Line TODO to line TODO, an actual lattice-based encryption scheme is modelled. This algorithm is modelled as an instance of the `LatticeBasedCipher` class and all its attributes (e.g. `messageSize`, `cipherSize`) as well as the attributes it inherited from `AsymmetricCipher` and `Cipher` are defined.

    abstract Cipher
        name -> string
        description -> string
        security -> Security
        performance -> Performance
        secProperty -> AttackModel
        quantum -> XQuantum
    
    abstract AsymmetricCipher : Cipher
        keySizePub -> integer
        keySizeSec -> integer
        performanceEnc -> Performance
        performanceDec -> Performance
    
    abstract LatticeBasedCipher: AsymmetricCipher
        msgSize ->integer
        cipherSize -> integer
        n -> integer
        q -> integer
    
    LP: LatticeBasedCipher
        [ name = "LP" ]
        [ description = "Linder-Peikert Scheme" ]
        [ msgSize = 1 || msgSize = 2 || msgSize = 3 ]
        [ n = 1 || n = 2 || n =€ 3 ]
        [ q = 20 || q = 40 ]
        s -> integer
        [ quantum = post ]
        [ n = 1 -> q = 20 && s = 6 && security = Broken && cipherSize = 22 * msgSize ]
        [ n = 2 -> q = 40 && s = 9 && security = Weak && cipherSize = 30 * msgSize ]
        [ n = 3 -> q = 40 && s = 7 && security =€ Medium && cipherSize = 36 * msgSize ]
        [ keySizePub = n * n * 25]
        [ keySizeSec = log * 26 ]
        [ performanceEnc = Slow ]
        [ performanceDec = Fast ]

## Integrating Tasks

Tasks are the main way users of CogniCrypt  interact with the tool. When a user wants to implement a cryptographic task (e.g., communicate over a secure channel, encrypt data using a password), they can open the tool and select the respective task. CogniCrypt then guides them through a set of questions, lets them select one combination of algorithms and algorithm configurations, and generates the necessary code.

For this to work properly, task developers are required to contribute four components. In addition to the three general components - implementation, usage example, CrySL rules, and variability model - they also have to provide the high-level questions that CogniCrypt asks the user in the beginning to configure the generated code. To simplify the extension of the model and the development of the questions, these steps may be conducted collaboratively with the CogniCrypt developers. Shape and form of all of these components depend on the exact way the task is being integrated.

### Implementation

The implementation for a task may be provided to CogniCrypt as source code or a jar file. In this case, end-users get direct access to the code. If task developers do not wish to share the source code, they may provide access to the task's functionality indirectly through an API. Suppose a task developer wants to offer long-term document archiving as a cryptographic task in CogniCrypt. An archive requires the documents to stored on a hard-disk. To not require the user to manage these files on their own, the task developer offers their archive as a web service that stores the documents on their server. In this case, the task developer does not have to provide any implementation directly to CogniCrypt. 

A task developer may choose to let the application developer configure the task in terms of used algorithms among other things. This configuration is done through the dialogue system after the CogniCrypt user selects a task they want to implement.

### Usage Example 

Regardless of how CogniCrypt users access a task's functionality, task developers need to provide a usage example for the implementation. If the implementation is provided as source code or a jar file, the usage example simply illustrates how one may use the implementation. If the end-user has to access the functionality through an API, the generated usage code should reflect how the API should best be used. In the above example of a document archiving web service, the usage example should showcase how to use important calls like`addDocumentToArchive()`, `retrieveFileFromArchive()`, and `verifyDocument()`.

During the code generation, CogniCrypt generates the usage example into the class the application developer has currently opened. A usage example for a symmetric encryption task is shown in the listing below. At first, a secret key is generated using the `KeyGenerator` class. Then, the key and the plain text are passed as parameters to the actual encryption method, which is shown in Figure TODO.

    public void performEncryption(byte[] plaintext) {
        KeyGenerator keyGen = KeyGenerator.getInstance("AES");
        keyGen.init(256);
        SecretKey key = keyGen.generateKey();	
        	
        Enc enc = new Enc();
        return enc.encrypt(plaintext, key);
    }

### CrySL Rules 

On top of that, task developers must provide usage rules for their usage example as well as the implementation if the user is not only accessing it through an API. CogniCrypt  then automatically runs static analyses based on these rules in the background to ensure the code stays secure. 
TODO CrySL

Consider the usage example for a symmetric encryption task from Listing TODO again. Since this code is generated directly into the application developer's project they might alter the code. They could, for instance, generate the key as it is shown in Listing TODO. In that listing, the key material is a fixed hard-coded string (see Line TODO, while the class `KeyGenerator` is using a CSPRG for the key material. Since the key would no longer be random and could be retrieved by decompiling the respective Java class file, the encryption cannot be deemed secure anymore.

    public void performEncryption(byte[] plaintext) {
        SecretKey secretKeySpec = new SecretKeySpec("key".getBytes(), "AES");
        Enc enc = new Enc();
        return enc.encrypt(plaintext, key);
    }

In terms of usage rules for the actual task implementation, consider Listing TODO. It shows a possible implementation of method `encrypt()` that is being called in the previous code snippet and illustrates how to potentially implement an encryption in Java. This implementation of the symmetric encryption task, however, contains a misuse of the underlying API. For a more detailed description of the misuse, readers are referred to Section TODO. 

### Model and Configuration Questions
In general, task developers should let the the end user configure their implementation. For an appropriate configuration point, assume that for a task an asymmetric encryption needs to be performed. Instead of implementing an encryption using, say, RSA with a fixed key length, the developer may outsource this decision to the CogniCrypt  user. To ensure that CogniCrypt presents the end-user with only appropriate algorithms to pick from, the requirements are specified in the model in the variability modeling language Clafer.

To accomplish this, the given task must be modelled top-down from the task to the algorithms, which serve as basic building blocks. Consider the symmetric encryption example in Listing TODO. In Lines TODO to TODO, cipher related algorithm classes are defined. The following lines define two instances of symmetric block ciphers, namely AES and DES. Finally, in Lines TODO to TODO, the task element is defined as using one algorithm of type `SymmetricBlockCipher`. All components of the task must be modelled until they are mapped to one of the algorithm classes. In the case of the symmetric encryption task, this goal is already reached as it directly uses a symmetric block cipher. When defining more complex tasks, several layers between the task element and the basic building blocks might be necessary.

    abstract Cipher 
        name -> string 
        description -> string
        security -> Security
        performance -> Performance
    
    abstract SymmetricCipher : Cipher
        keySize -> integer
    
    abstract SymmetricBlockCipher : SymmetricCipher
        mode -> Mode
        padding -> Padding
        [mode != ECB]
        [mode = CBC => padding != NoPadding]
    
    AES: SymmetricBlockCipher €\label{line:AESex}€
        [name = "AES"]
        [description = "Advanced Encryption Standard"]
        [keySize = 128 || keySize = 192 || keySize = 256]
        [keySize = 128 -> performance = VeryFast && security = Medium]
        [keySize > 128 -> performance = Fast && security = Strong]
    
    DES: SymmetricBlockCipher €\label{line:DESex}€
        [name = "DES"]
        [description = "Data Encryption Standard"]
        [performance = VeryFast ]
        [security = Broken ]
        [keySize = 56 ]
    
    SymmetricEncryption : Task €\label{line:symmBlockTask}€
        [description = "Encrypt data using a secret key"]
        cipher -> SymmetricBlockCipher 
        [cipher.security > Medium]

Lastly, if developers want to allow the end user of CogniCrypt to configure their task implementation, they are also required to provide the questions CogniCrypt can ask the user. These questions can include a precise specification of the expected security level, as is the case in Figure TODO, and their answers may influence the variability model as well as the generated code.