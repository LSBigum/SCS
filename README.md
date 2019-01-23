# SCS Wiki on Heartbleed

## Introduction

In 2014 a devastating security vulnerability was discovered in the OpenSSL implementation of the SSL/TLS protocol. TLS is a protocol that is commonly used to safeguard transactions being made on the web while it is also being used for e-mails, VPNs, voice over IP (VoIP), and more. In particular, when a website uses the HTTPs protocol, it means that the web page you're viewing has been transmitted to you in encrypted form, meaning that all data is supposed to protected from any kind of eavesdropping or prying eyes - and that goes both ways in this case. The confidential data being provided to the website - e.g. any passwords typed in - will in theory be encrypted, and be safe from someone trying to eavesdrop on you. In particular, what the 's' in HTTPs refers to, is that you're transmitting HTTP traffic that is encrypted by SSL. OpenSSL is an open-source implementation of the TLS protocol. It also happens to be the most widely deployed implementation. It is being used on notable open source webservers like Apache and nginx which, when combined, had a market share of more than 66% of the active sites in 2014. Anyone who uses the web on a daily basis is therefore certain to be interacting with an OpenSSL implementation, which makes this security flaw all the more severe. This wiki will describe both the attack and the flaw it exploits, as well as some of its ramifications. 

## The Heartbeat extension

In OpenSSL versions 1.01 and up to 1.02 betas, there was a subtle but highly critical programming mistake that could lead to an attacker being able to gain access to the otherwise encrypted data. A system running one of the aforementioned versions of the OpenSSL implementation can be attacked quite easily. In particular, there's an extension to the TLS protocol known as Heartbeat. The Heartbeat allows you to keep a TLS session running even though no data has been communicated in a while, simply by sending a special message known as a heartbeat request. Before the introduction of Heartbeat, if there was a period without any communication between a client and server, the TLS session might have been terminated and would have to be reopened if any further data transfer was required. Heartbeat can also be used to check if the receiver of the signal is still present, and in that way keep the line of communication alive. The request itself consists of a payload containing the actual message, as well as some explicit information that specifies the size of the payload. The response from the server will then contain the same payload and some padding. 

## Heartbleed bug

The vulnerability that went undiscovered for approximately two years, was that a computer receiving a heartbeat request would not check whether the information regarding size of the payload was actually accurate - it would blindly trust the information in the request. In the end, this was a rather simple programming mistake that allowed an attacker to send a very short message, for instance only a single byte of payload, while the message header could be manifactured to inform that the payload actually had a size of 64KB. When the response is being processed, the responder will simply look at its memory address for the request message payload, and send back a message with a payload of the size specified in the initial heartbeat request, thus passing along whatever is stored in the memory afterwards as well. In the case of a heartbeat request with an initial payload of a single byte, but with a manifactured size indicating the payload is 64KB long, the response will then contain that same initial payload of one byte, as well as whatever happens to be stored in the responders memory next to the initial payload. This is known as a buffer over-read. The information being sent in the response could contain usernames, passwords, specific requests, credit card numbers, or even the keys that SSL was using to encrypt/decrypt data to and from its clients, thus allowing the attacker to spoof the identity of the website and thereby having easy access to any further data being transmitted. 

### Leaked information
More specifically regarding the contents of the leaked information, this can be separated into four categories:

- Primary key material
- Secondary key material
- Protected content
- Collateral


## References

[1] http://heartbleed.com/, General information on Heartbleed by Synopsys
[2] https://www.csoonline.com/article/3223203/vulnerabilities/what-is-the-heartbleed-bug-how-does-it-work-and-how-was-it-fixed.html
