CogniCrypt's static analysis automatically runs a static analysis on the code within Eclipse. The static analysis is based on `CrySL rules` that specify the *correct* use of an application programming interface (API). `CrySL` is a domain-specific language that allows to specify usage patterns of APIs. The static analysis reports any deviations from the usage pattern defined within the rules. 

While the `CrySL` rules are adjustable, a developer using CogniCrypt is not expected to change the rules of CogniCrypt. 

## Eclipse Error Markers and their Meanings

CogniCrypt generates errors markes when the analysis detects incorrect and insecure parts of code. CogniCrypt displays error markers within the Eclipse IDE to warn the developer about insecure code. The error markers are associated to the respective line in the editor the errors is located at. 

There are various different error types that CogniCrypt reports. Below, we distinguish the error types based on the warning the error marker reports.

* `"MD5 should be any of {SHA-256, SHA-384, SHA-512}"` is a **Constraint Error**: the static analysis detects an incorrect `String` (or `int`) to flow as argument to a method call. CogniCrypt automatically suggest alternatives to fix the issue. The error message describes that `MD5` should be replaced by and of the other `String` elements.  

* `"Unexpected call to method reset. Expect a call to one of the following methods digest,update"` marks a **Typestate Error**. The sequence of object calls made on an object is not according to its `CrySL` specification.

* `"Operation with Cipher object not completed. Expected call to update, doFinal."` marks an **Incomplete Operation Error**. An incomplete operation errors appears, when a call on an object is missing and the object is garbage collected without having properly used. A typically example for such an error is a missing call to `close` on a `FileWriter`.

* `"Variable keyBytes was not properly randomized"` is called a **Required Predicate Error**. Such an error is reported when the analysis infers that the combination of using *several* object is incorrect. The error message reports that the developer uses a variable `keyBytes` (containing some `byte[]`) for some cryptographic operation. The correct specification of the API requires that the `byte` array has been previously `randomized` (wich means correctly generate by some API).

## Syntax of the Domain-Specific Language CrySL

The error markers are generated based on violations of *rules*. Rules in CogniCrypt are written in `CrySL`. `CrySL` is a domain-specific language for the specification of correct cryptograhy API uses in Java. The Eclipse plugin CogniCrypt ships with a XText editor that supports the `CrySL` syntax. `CrySL` encodes a white-list approach and specifies how to *correctly* use crypto APIs. We discuss some of the most important concepts of the rule language here, the [research paper](http://drops.dagstuhl.de/opus/volltexte/2018/9215/pdf/LIPIcs-ECOOP-2018-10.pdf) provides more detailed insides on the language. 

Each `CrySL` rule is a specification of a single Java class. A short example of a `CrySL` rule for `javax.crypto.Cipher` is shown below. 

```
SPEC 	javax.crypto.Cipher
OBJECTS 
	java.lang.String trans;
	byte[] plainText; 
	java.security.Key key;
	byte[] cipherText;
EVENTS 
	Get: getInstance(trans); 
	Init: init(encmode, key); 
	doFinal: cipherText = doFinal(plainText); 
ORDER 
	Get, Init, (doFinal)+ 
CONSTRAINTS  
	encmode in {1,2,3,4};
	part(0, "/", trans) in {"AES", "Blowfish", "DESede", ..., "RSA"};
	part(0, "/", trans) in {"AES"} => part(1, "/", trans) in {"CBC"};
REQUIRES 
	generatedKey[key, part(0, "/", trans)];
ENSURES 
	encrypted[cipherText, plainText]; 
```
Each rule has a `SPEC` clause that lists the fully qualified class name the following specification holds for (in this case `javax.crypto.Cipher`)
The `SPEC` clause is followed by the blocks `OBJECTS`, `EVENTS`, `ORDER`, `CONSTRAINTS`, `REQUIRES` and `ENSURES`. 
Within the `CONSTRAINTS` block each rule lists `Integer` and `String` constraints.  The `OBJECTS` clause lists all variable names that can be used within all blocks of the rule. The `EVENTS` block lists API method calls that can be made on each `Cipher` object. When an event is encountered, the actual values of the events parameters are assigned to respective variable name listed in the rule. These parameter values can then be constrained by `CONSTRAINTS`. 

### The CONSTRAINTS section

The `Cipher` rule lists `encmode in {1,2,3,4};` within its `CONSTRAINTS` block. The value `encmode` that is passed to method `init(encmode, cert)` is restricted to be one of the four integers. In other terms, whenever the function `init` is called, the value passed in as first parameter must be in the respective set.  The constraint `part(0, "/", trans) in {"AES", "Blowfish", "DESede", ..., "RSA"}`  refers to the fact that at the call to `Cipher.getInstance(trans)` the `String trans` must be correctly formed. The function `part(0, "/", trans)` splits the `String` at the character `"/"` and returns the first part. Hence the constraint restricts the first part prior of any `"/"` to be either `"AES"` or `"RSA"`. The third constraint (`part(0, "/", trans) in {"AES"} => part(1, "/", trans) in {"CBC"};`) is a conditional constraint: If the first part of `trans` is `"AES"`, then the second part of `trans` must be `"CBC"`. For example, this conditional rule warns a developer writing `Cipher.getInstance("AES/ECB/PKCS5Padding")` instead of `Cipher.getInstance("AES/CBC/PKCS5Padding")`. 

### The ORDER section

The `ORDER` section of a rule specifies a regular-expression like description of the excepted events to occur on each individual object. For the `Cipher` rule the order is `Get, Init, (doFinal)+`. The terms `Get`, `Init` and `doFinal` are labels and group a set of API methods that are defined within the `EVENTS` block. The regular expression stated in the `ORDER` section enforces the following order on a `Cipher` object: The object must be create by a `Get`, i.e., `Cipher.getInstance`, call, then `Init` must be called before eventually any (and at least one) times the method `doFinal` is called. A programmer who writes the program below contradicts the `ORDER` section of the `CrySL` rule: A call to `init` on the `cipher` object is missing between the `getInstance` and `doFinal` call (the missing call is commented out).

```
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
//cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);
cipher.doFinal(plainText);
```

### The ENSURES and the REQUIRES section

Cryptographic tasks are more complex and involve interaction of multiple object instances at runtime. For example for an encryption task with a `Cipher` instance, the `Cipher` object must be initialized with a securely generated key. The API of the `Cipher` object has a method `init(encmode,key)` where the second parameter is the respective key and is of type `SecretKeySpec`. For a correct use of the `Cipher` object, the key must be used correctly as well.

To cope with these object interactions, `CrySL` allows the specification of what we call *predicates* that are listed in the blocks `REQUIRES` and `ENSURES`. An object that is used in coherence with the rule receives the predicate listed in the `ENSURES` block. In turn, the block `REQUIRES` allows rules to force other objects to hold certain predicates.

The specification of the `Cipher` rule lists a predicate `generatedKey[key,...]` within its `REQUIRES` block. The variable name `key` refers to the same object that is used within the event `Init: init(encmode, key);` of the  `EVENTS` block. Hence, the key object must receive this predicate which is listed in the rule for `javax.crypto.SecretKeySpec`. 

```
SPEC javax.crypto.spec.SecretKeySpec
OBJECTS
	java.lang.String alg;
	byte[] keyMaterial;
		
EVENTS
	c1: SecretKeySpec(keyMaterial, alg);
...
REQUIRES
	randomized[keyMaterial]; 
ENSURES
	generatedKey[this, alg];
```

Above is an excerpt of the rule for `SecretKeySpec`. The predicate `generatedKey` is listed within the `ENSURES` block of this rule. The static analysis labels any object of type `SecretKeySpec` by `generatedKey` when the analysis finds the object to be used correctly (with respect to its `CrySL` rule).


## CrySL Rules for the JCA
CogniCrypt ships with a pre-defined set of `CrySL` rules. The standard rule set covers the correct specification of most classes of the [Java Cryptographic Architecture (JCA)](https://docs.oracle.com/javase/8/docs/technotes/guides/security/crypto/CryptoSpec.html). The JCA offers various cryptographic services. In the following, we describe these services with their respective classes and briefly summarize important usage constraints. All mentioned classes are defined in the packages `javax.crypto` and `java.security` of the JCA. 

The rule set is also [publicly available](https://github.com/CROSSINGTUD/Crypto-API-Rules) .The definition of the `CrySL` rules are found in the files ending in `.cryptsl` named with the respective class name.

* **Asymmetric Key Generation**: 
Asymmetric and symmetric cryptography requires different key formats. Asymmetric cryptography uses pairs of public and private keys. While one of the keys encrypts plaintexts to ciphertexts, the second key decrypts the ciphertext. The JCA models a key pair as class `KeyPair` and are generated by `KeyPairGenerator`. 

* **Symmetric Key Generation**:
Symmetric cryptography uses the same key for encryption and decryption. The JCA models symmetric keys as type `SecretKey`, generated by a `SecretKeyFactory` or `KeyGenerator`. The `SecretKeyFactory` also enables password-based cryptography using `PBEParameterSpec` or `PBEKeySpec`. 

* **Signing and Verification of Data**:
The class `Signature` of the JCA allows one to digitally sign data and verify a signature based on a private/public key pair. A `Signature` requires the key pair to be correctly generated, hence the rule for `Signature` requires a predicate from the asymmetric-key generation task.

* **Generation of Initialization Vectors**:
Initialization vectors (IVs) are used to add entropy to ciphertexts of encryptions. An IV must have enough randomness and must be properly generated. The JCA class `IvParameterSpec` wraps a byte array as an IV and it is required for the array to be randomized by `SecureRandom`. The `CrySL` rule for `IvParameterSpec` requires a predicate `randomized`.

* **Encryption and Decryption**
The key component of the JCA is represented by the class `Cipher`, which implements functionality to encrypt or decrypt data. Depending on the used algorithms, modes and paddings must be selected and keys and initialization vectors must be properly generated. Hence, the complete `CrySL` rule for `Cipher` requires many other cryptographic services to be executed securely earlier and list them in its respective `REQUIRES` clause.

* **Hashing & MACs**´:
There are two forms of cryptographic hash functions. A MAC is an authenticated hash that requires a symmetric keys, but there are also keyless hash functions such as MD5 or SHA-256. The JCA's class `Mac` implements functionality for mac-ing, while keyless hashes are computed by `MessageDigest`. 

* **Persisting Keys**:
Securely storing key material is an important cryptographic task for confidentiality and integrity of the encrypted data. The JCA class `KeyStore` supports  developers in this task and stores the key material.

* **Cryptographically Secure Random-Number Generation**: 
Randomness is vital in all aspects of cryptography. Java offers cryptographically secure pseudo-random number generators through `SecureRandom`. As discussed for `PBEKeySpec`, `SecureRandom` often acts as a helper and therefore many rules list the `randomized` predicate in their own `REQUIRES` section.
