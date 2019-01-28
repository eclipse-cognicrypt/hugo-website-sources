---
title: "The CrySL Language"
date: 2018-09-03T19:48:11+02:00
---

The [static analysis](/cognicrypt/documentation/codeanalysis) is based on *CrySL rules* that specify the *correct* use of an application programming interface (API). *CrySL* is a domain-specific language that allows to specify usage patterns of APIs. The static analysis reports any deviations from the usage pattern defined within the rules. 

## Syntax of the Domain-Specific Language CrySL

CogniCrypt's error markers are generated based on violations of *rules*. Rules in CogniCrypt are written in *CrySL*. *CrySL* is a domain-specific language for the specification of correct cryptograhy API uses in Java. The Eclipse plugin CogniCrypt ships with a XText editor that supports the *CrySL* syntax. *CrySL* encodes a white-list approach and specifies how to *correctly* use crypto APIs. We discuss some of the most important concepts of the rule language here, the [research paper](http://drops.dagstuhl.de/opus/volltexte/2018/9215/pdf/LIPIcs-ECOOP-2018-10.pdf) provides more detailed insides on the language. CogniCrypt ships with a default rule set for the [Java Cryptographic Architecture (JCA)](https://docs.oracle.com/javase/8/docs/technotes/guides/security/crypto/CryptoSpec.html). At the bottom of this page, you may find a description of this rule set.

Each *CrySL* rule is a specification of a single Java class. A short example of a *CrySL* rule for `javax.crypto.Cipher` is shown below. 

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

To cope with these object interactions, *CrySL* allows the specification of what we call *predicates* that are listed in the blocks `REQUIRES` and `ENSURES`. An object that is used in coherence with the rule receives the predicate listed in the `ENSURES` block. In turn, the block `REQUIRES` allows rules to force other objects to hold certain predicates.

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

Above is an excerpt of the rule for `SecretKeySpec`. The predicate `generatedKey` is listed within the `ENSURES` block of this rule. The static analysis labels any object of type `SecretKeySpec` by `generatedKey` when the analysis finds the object to be used correctly (with respect to its *CrySL* rule).

## Addition or Modification of CrySL Rules
All the *CrySL* rules currently used by the crypto assistant CogniCrypt are present in the repo named [Crypto-API-Rules](https://github.com/CROSSINGTUD/Crypto-API-Rules). You need to clone this repo and import it as a maven project in the eclipse where you have already installed CogniCrypt & *CrySL* plugins. These plugins lets you update the *CrySL* rules on the fly. Please note that the target project must been in the same eclipse instance where the *CrySL-Examples* project is imported. In the imported project you can find separate packages for every service provider namely JCA, Google Tink & BouncyCastle in which you can find a list of `.cryptsl` files which specifies the rules of different classes corresponding to specific provider. You can edit them or even add a new rule and save all your changes. Now you simply have to re-run CogniCrypt on your target project and the plugin would use the previously modified ruleset which has been saved.

<div class="imgbox">
    <img class="center-fit" src='/images/1_correct_code.png' alt="An example code without any misuse">
</div>

<div class="imgbox">
    <img class="center-fit" src='/images/2_misuse_code.png' alt="Misuse of key size">
</div>

<div class="imgbox">
    <img class="center-fit" src='/images/3_error_markers.png' alt="Static Analyzer reports error markers">
</div>

<div class="imgbox">
    <img class="center-fit" src='/images/4_original_rule.png' alt="Original crySL rule for KeyGenerator class">
</div>

<div class="imgbox">
    <img class="center-fit" src='/images/5_modified_rule.png' alt="Modified crySL rule for KeyGenerator class">
</div>

<div class="imgbox">
    <img class="center-fit" src='/images/6_error_markers_disappear.png' alt="Static Analyzer doesn't report error markers">
</div>


## CrySL Rules for the JCA

CogniCrypt ships with a pre-defined set of *CrySL* rules. The standard rule set covers the correct specification of most classes of the [Java Cryptographic Architecture (JCA)](https://docs.oracle.com/javase/8/docs/technotes/guides/security/crypto/CryptoSpec.html). The JCA offers various cryptographic services. In the following, we describe these services with their respective classes and briefly summarize important usage constraints. All mentioned classes are defined in the packages `javax.crypto` and `java.security` of the JCA. 

The rule set is also [publicly available](https://github.com/CROSSINGTUD/Crypto-API-Rules) .The definition of the *CrySL* rules are found in the files ending in `.cryptsl` named with the respective class name.

- **Asymmetric Key Generation**: 
  Asymmetric and symmetric cryptography requires different key formats. Asymmetric cryptography uses pairs of public and private keys. While one of the keys encrypts plaintexts to ciphertexts, the second key decrypts the ciphertext. The JCA models a key pair as class `KeyPair` and are generated by `KeyPairGenerator`. 
- **Symmetric Key Generation**:
  Symmetric cryptography uses the same key for encryption and decryption. The JCA models symmetric keys as type `SecretKey`, generated by a `SecretKeyFactory` or `KeyGenerator`. The `SecretKeyFactory` also enables password-based cryptography using `PBEParameterSpec` or `PBEKeySpec`. 
- **Signing and Verification of Data**:
  The class `Signature` of the JCA allows one to digitally sign data and verify a signature based on a private/public key pair. A `Signature` requires the key pair to be correctly generated, hence the rule for `Signature` requires a predicate from the asymmetric-key generation task.
- **Generation of Initialization Vectors**:
  Initialization vectors (IVs) are used to add entropy to ciphertexts of encryptions. An IV must have enough randomness and must be properly generated. The JCA class `IvParameterSpec` wraps a byte array as an IV and it is required for the array to be randomized by `SecureRandom`. The *CrySL* rule for `IvParameterSpec` requires a predicate `randomized`.
- **Encryption and Decryption**
  The key component of the JCA is represented by the class `Cipher`, which implements functionality to encrypt or decrypt data. Depending on the used algorithms, modes and paddings must be selected and keys and initialization vectors must be properly generated. Hence, the complete *CrySL* rule for `Cipher` requires many other cryptographic services to be executed securely earlier and list them in its respective `REQUIRES` clause.
- **Hashing & MACs**´:
  There are two forms of cryptographic hash functions. A MAC is an authenticated hash that requires a symmetric keys, but there are also keyless hash functions such as MD5 or SHA-256. The JCA's class `Mac` implements functionality for mac-ing, while keyless hashes are computed by `MessageDigest`. 
- **Persisting Keys**:
  Securely storing key material is an important cryptographic task for confidentiality and integrity of the encrypted data. The JCA class `KeyStore` supports  developers in this task and stores the key material.
- **Cryptographically Secure Random-Number Generation**: 
  Randomness is vital in all aspects of cryptography. Java offers cryptographically secure pseudo-random number generators through `SecureRandom`. As discussed for `PBEKeySpec`, `SecureRandom` often acts as a helper and therefore many rules list the `randomized` predicate in their own `REQUIRES` section.

## Cryptographic Service Providers

JCA framework is based on provider architecture. This means that the implementation of the above cryptographic services are suppied by various providers. Apart from the default Sun provider which comes bundled with JDK we also have specific providers such as Bouncy Castle. Any program can get implementations either from one of installed providers or from a specific provider by refering to its name. Hence *CrySL* rules are being developed for third party providers like Bouncy Castle in order to extend the capabilities of Cognicrypt.

## CrySL Rules for the Bouncy Castle

The below rule set covers the specifications of most classes in the [Bouncy Castle (BC)](https://github.com/bcgit/bc-java/tree/master/core/src/main/java/org/bouncycastle/crypto). In the following, we describe all the services with their respective classes and briefly summarize important usage constraints. All mentioned classes are defined in the lightweight crypto packages `org.bouncycastle.crypto.*` of the BC.

The rule set is also [publicly available](https://github.com/CROSSINGTUD/Crypto-API-Rules/tree/bouncy_castle)

- **Asymmetric Key Generation**: 
  In BC every asymmetric cryptography has a separate key pair generator. For example RSA has `RSAKeyPairGenerator`, DSA has `DSAKeyPairGenerator` and so on. These asymmetric or public/private, cipher key pair generators should conform to an interface `AsymmetricCipherKeyPairGenerator`. Every key pair generator has its corresponding key generation parameters which specify the keys being generated. For example, RSA has `RSAKeyGenerationParameters` and DSA has `DSAKeyGenerationParameters` both of which conforms to its base class `KeyGenerationParameters`.
- **Symmetric Key Generation**:
  The BC has a base class named `CipherKeyGenerator` for symmetric or secret, cipher key generators. Every symmetric algorithm has specific key generator class which extends this base class. For example DES has `DESKeyGenerator` which extends the base class to specify the parameters.
- **Encryption and Decryption**:
  There are two variants of `Cipher` equivalent in Bouncy Castle. One is `BlockCipher` and the other one is `AsymmetricBlockCipher` both of which are interfaces. All the symmetric engines & modes should conform to the former interface and all the asymmetric counterparts should adhere to the latter. BC also provides classes named `BufferedBlockCipher` and `BufferedAsymmetricBlockCipher` which are buffer wrappers for block cipher and asymmetric block cipher respectively, allowing the input to be accumulated in a piecemeal fashion until final processing.
- **Hashing & MACs**´:
  The BC has a base interface named `Mac` for implementations of message authentication codes (MACs) and `Digest` for implementations of hashing.
- **Cryptographically Secure Random-Number Generation**:
  The BC uses Java offered cryptographically secure pseudo-random number generator `SecureRandom` for randomness.