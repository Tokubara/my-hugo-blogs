+++
title = "a short tour of asymmetric cryptography from certificate"
author = ["Tian Zhuoyu"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [situation](#situation)
    - [The problem: Your connection is not private](#the-problem-your-connection-is-not-private)
    - [Certicate is not valid](#certicate-is-not-valid)
- [asymmetric encryption](#asymmetric-encryption)
    - [Why do we need encryption?](#why-do-we-need-encryption)
    - [symmetric encryption](#symmetric-encryption)
    - [Asymmetric encryption](#asymmetric-encryption)
- [Problem with asymmetric encryption: MITM](#problem-with-asymmetric-encryption-mitm)
    - [things are not perfect](#things-are-not-perfect)
    - [example](#example)
    - [the problem](#the-problem)
- [CA and digital certificates](#ca-and-digital-certificates)
    - [certificate](#certificate)
    - [now](#now)
- [back to the website](#back-to-the-website)

</div>
<!--endtoc-->


## situation {#situation}

<div class="NOTES">

Let's start with a situation.

</div>


### The problem: Your connection is not private {#the-problem-your-connection-is-not-private}

{{< figure src="/ox-hugo/Pasted_image_20230618190223.png" >}}

For example, [this site](https://138.2.115.37/buy/5).

<div class="NOTES">

I guess most of you have encountered a situation like this before: you enter a URL in your browser to open a website, but the browser tells you that the connection is not secure. Why does the browser think that the website is not secure?

</div>


### Certicate is not valid {#certicate-is-not-valid}

{{< figure src="/ox-hugo/Pasted_image_20230618192459.png" >}}

Certificate is not valid.

But what is a certificate?

<div class="NOTES">

If you click here, the browser will tell you that "Certificate is not valid." But what does "Certificate is not valid" mean? What is a certificate?

</div>

<div class="NOTES">

This is my topic. I'll try to explain what is a certificate, explain why some websites don't have certificates and what risks can be associated with accessing these sites.

</div>


## asymmetric encryption {#asymmetric-encryption}

<div class="NOTES">

Certificates is to solve some issues in asymmetric encryption. Therefore, I will first provide a brief introduction to asymmetric encryption.

</div>


### Why do we need encryption? {#why-do-we-need-encryption}

{{< figure src="/ox-hugo/Pasted_image_20230618201349.png" >}}

Why do we need encryption? Because the Internet is not secure at all.

Let's assume that the two parties communicating are Alice and Bob, and Eve is a malicious third party trying to eavesdrop on their communication.

Before Alice sends a message to Bob, she encrypts the message to create ciphertext. Bob decrypts the ciphertext to obtain the message after receiving it.

<div class="NOTES">

Why do we need to encrypt data? Because the internet is not safe at all. transmitting plain text poses a risk of being intercepted, tampered and discarded. For example, you probably don't want someone else to eavesdrop on your Alipay password, right?

For the purpose of this discussion, let's assume that the two parties communicating are Alice and Bob, and Eve is a malicious third party trying to eavesdrop on their communication.

Before Alice sends a message to Bob, she encrypts the message to create ciphertext. Bob decrypts the ciphertext to obtain the message after receiving it. just as illustrated in this picture.

</div>


### symmetric encryption {#symmetric-encryption}

{{< figure src="/ox-hugo/Pasted_image_20230618204353.png" width="45%" >}}

Symmetric encryption: encrypt and decrypt data with the same key, so the sender and receiver have the same key.

The problem of symmetric encryption: how to transfer a key safely? That's the motivation for asymmetric encryption.

<div class="NOTES">

Before asymmetric encryption was introduced, there was only symmetric encryption. Symmetric encryption uses a single key to encrypt and decrypt data. This key is shared between the sender and receiver. The problem is that, the secret key can be difficult to be distributed over a public network, such as the internet, because your secret key can be eavesdrop and become not secret any more. And that's the motivation of asymmetric encryption.

</div>


### Asymmetric encryption {#asymmetric-encryption}

Asymmetric encryption uses key pairs for encryption and decryption. key pair=(public key, private key).

The public key is used for encryption, and the private key is used for decryption. Public key should be shared with senders, secret key shouldn't be told anyone.

{{< figure src="/ox-hugo/Pasted_image_20230619093338.png" >}}

Even if Eve intercepts public keys of Alice and Bob, it doesn't matter because public keys are used for encryption not decryption. Eve cannot intercept private keys because private keys will never be sent.

<div class="NOTES">

Asymmetric encryption uses key pairs for encryption and decryption. A key pair consists of a public key and a private key. The public key is used for encryption, and the private key is used for decryption. Public key should be shared with senders, secret key shouldn't be told anyone.

As shown in this picture, if Alice wants to receive a message from Bob, she sends him her public key, but never her private key. Before Bob sends a message to Alice, he encrypts it using Alice's public key, and Alice decrypts it using her private key. Even if Eve eavesdrops on the encrypted message, she can't decrypt it without Alice's private key. Similarly, if Alice wants to send a message to Bob, she needs to obtain Bob's public key first.

Even if the public key is intercepted by Eve, it doesn't matter because public key can only be used for encryption and not decryption. Eve doesn't have Alice.

</div>

In asymmetric encryption, there's no need to distribute the private key, and there's no need to worry about the public key being intercepted.

<div class="NOTES">

Therefore, in asymmetric encryption, there's no need to distribute the private key, and there's no need to worry about the public key being intercepted.

</div>

<a id="figure--in asymmetric encryption, the keys used for encryption and decryption are different."></a>

{{< figure src="/ox-hugo/Pasted_image_20230618204948.png" width="45%" >}}

<div class="NOTES">

The picture emphasizes that in asymmetric encryption, the keys used for encryption and decryption are different.

</div>


## Problem with asymmetric encryption: MITM {#problem-with-asymmetric-encryption-mitm}

<div class="NOTES">

Sounds perfect, right? Not that perfect.

</div>


### things are not perfect {#things-are-not-perfect}

{{< figure src="/ox-hugo/Pasted_image_20220920204326.png" >}}

Asymmetric encryption still poses a potential risk of man-in-the-middle attacks.

<div class="NOTES">

Asymmetric encryption poses a potential risk of man-in-the-middle attacks. Let me give an example to illustrate this point.

</div>


### example {#example}

{{< figure src="/ox-hugo/Pasted_image_20230618212403.svg" >}}

<div class="NOTES">

It is supposed to be that Alice sends her public key to Bob, and Bob sends his public key to Alice.

</div>

{{< figure src="/ox-hugo/Pasted_image_20230618210943.svg" >}}

<div class="NOTES">

But now there is a man in the middle Eve. When Alice sends her public key to Bob, Eve intercepts Alice's message and obtains Alice's public key, then Eve sends her own public key to Bob. Similarly, when Bob sends his public key to Alice, Eve intercepts Bob's message, obtains Bob's public key, and sends her own public key to Alice. Alice thinks she is communicating with Bob, and Bob thinks he is communicating with Alice, but in fact, they are both communicating with Eve.

</div>


### the problem {#the-problem}

When you receive a public key claiming to be Alice's, how can you make sure it is actually Alice's public key and not the public key of a man-in-the-middle attacker?

That's when certificates come into play.

<div class="NOTES">

The problem is, when you receive a public key claiming to be Alice's, how can you make sure it is actually Alice's public key and not the public key of a man-in-the-middle attacker? That's when certificates come into play.

</div>


## CA and digital certificates {#ca-and-digital-certificates}


### certificate {#certificate}

certificate=subject's public key+digital signature of CA (Certificate Authority)

Digital signatures ensure the authenticity of the content of the certificate.

The certificate proves that Alice's public key is indeed hers.

Just like driver's license.

<div class="NOTES">

Technically, certificates consist of the subject's public key and the issuer's digital signature.

The digital signature is another important concept in asymmetric cryptography. Here we only need to know that with a digital signature, Bob can be sure that the certificate is issued by a trusted CA and the contents are trustworthy, rather than being forged by Eve.

The certificate proves that Alice's public key is indeed hers.

A very appropriate analogy is a driver's license. Just as the transportation department issues a driver's license to prove a driver's qualifications, a CA issues a certificate to ensure the authenticity of the public key.

</div>

{{< figure src="/ox-hugo/Pasted_image_20230618214452.png" width="45%" >}}

<div class="notes">

As shown in this image, the CA issues a certificate that contains Jane Doe's public key and a digital signature from the CA.

</div>


### now {#now}

Now instead of sending public keys, they send their own certficates.

{{< figure src="/ox-hugo/Pasted_image_20230618230110.svg" width="45%" >}}

Even if Eve intercepts the certificates, she can do nothing.

{{< figure src="/ox-hugo/Pasted_image_20230618230134.svg" width="45%" >}}

<div class="notes">

Now, Alice and Bob are sending their respective certificates to each other instead of public keys. Even if Eve intercepts the certificate, she can't do anything because If Eve also has a certificate, the certificate will only show that it's Eve's public key and not Alice's. Therefore, Eve cannot impersonate Alice or Bob anymore.

</div>


## back to the website {#back-to-the-website}

{{< figure src="/ox-hugo/Pasted_image_20230618192459.png" width="30%" >}}

If the website do not have a certificate issued by a CA, it could be a phishing website (钓鱼网站) or a legitimate website that hasn't yet applied for a certificate from the CA

A legitimate website without a certificate is still insecure. Your communication may be vulnerable to a man-in-the-middle attack. You should not send any sensitive information, such as passwords.

<div class="NOTES">

Now back to our example, what does "Certificate is not valid" mean? It means that the website doesn't have a certificate issued by a CA. it could be a phishing website, such as a fake Baidu website. Alternatively, it could be a legitimate website that hasn't yet applied for a certificate from the CA. However, even if it is a legitimate website, your communication is not secure. This puts you at risk of a man-in-the-middle attack. While you think you are communicating with the website server, it is possible that you are communicating with a man-in-the-middlebut. Therefore, even if you are certain that the website is legitimate, you should not send any sensitive information, such as passwords.

</div>
