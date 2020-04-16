---
title: "CogniCrypt - Secure Integration of Cryptographic Software"
---

A large number of recent studies have shown that most software applications that use cryptographic procedures misuse them. The VeraCode Report <a href="https://www.veracode.com/state-software-security-2017" target="_blank">State of the Software Security 2017</a> lists the insecure use of cryptography as the second most common cause of software vulnerabilities, right after data leakage.

Eclipse CogniCrypt was developed within the collaborative research center CROSSING of Technische Universität Darmstadt. It allows developers to quickly identify and fix security-critical misuses of cryptographic libraries.

The plugin Eclipse CogniCrypt ships in two main components: A wizard for **code generation** that supports a developer in generating secure code for common cryptographic tasks and a **static code analysis** that continuously checks the (generated and non-generated) code of the developer directly within Eclipse.

![Overview over CogniCrypt](images/home_codegen_codeanalysis.png)

            
# Jobs

<div class="alert alert-info" role="alert">

We currently have several openings for full-time research staff and software developers who will help us bring CogniCrypt to the next level. The openings are located both at Paderborn and Darmstadt. Please contact <a href="mailto:eric.bodden@upb.de?subject=CogniCrypt%20Job%20Offering">Eric Bodden</a> for further information.

</div>


# Code Generation

The code-generation feature CogniCrypt<sub>GEN</sub> is designed as a wizard that guides developers to select the correct cryptographic algorithms for their cryptographic use case at hand. The wizard asks high-level questions related the use case in order to tailor the solution to the user's needs. The [user documentation](./documentation/codegen) discusses the wizard in more detail.

<p align="center">

<video src="videos/codegen.mp4" controls width=800px>
  Ihr Browser kann dieses Video nicht wiedergeben.<br/>
  Dieser Film zeigt eine Demonstration des video-Elements. 
  Sie können ihn unter <a href="#">Link-Addresse</a> abrufen.
</video>

</p>

# Static Code Analysis

The static code analysis CogniCrypt<sub>SAST</sub>  continuously checks the developer's code for correct implementations. Upon saving the code in the editor, a static analysis is triggered in the background and reports warning when a cryptographic API is used incorrectly.

The video below shows a minimal example demonstrating the static code analysis within Eclipse.

<p align="center">
<video src="videos/staticanalysis.mp4" controls width=800px>
  Ihr Browser kann dieses Video nicht wiedergeben.<br/>
  Dieser Film zeigt eine Demonstration des video-Elements. 
  Sie können ihn unter <a href="#">Link-Addresse</a> abrufen.
</video> 
</p>
In the example, the developer creates a `Cipher` object and supplies the `String "AES"` as argument to configure using the encryption algorithm `AES`. They save their code and are warned instantaneously by CogniCrypt. By default, the algorithm `AES` encrypts with the insecure block mode `ECB`. The developer changes the `Cipher` object to be configured in a secure way (using the `String "AES/CBC/PKCS5Padding"` which requests from the provider a secure block mode `"CBC"` and a correct padding mode; assuming [padding oracle attacks](https://en.wikipedia.org/w/index.php?title=Padding_oracle_attack&oldid=881516766) are prevented, e.g. by adidtional integrity checks). CogniCrypt's error message disappears. For a more in-depth explanation, please check out the [user documentation](./documentation/codeanalysis).
