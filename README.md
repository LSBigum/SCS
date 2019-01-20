# SCS Wiki on Heartbleed

## Introduction

There was a devastating security vulnerability that was discovered in the OpenSSL implementation of the SSL/TLS protocol. If you don't know, TLS is a protocol that is commonly used to safeguard transactions you make on the web. Sometimes TLS reffered to as SSL and vice-versa. Technically SSL is the protocol that came first. TLS is more of a succesor, more or less, to SSL. You can more or less think of them as being interchangeable. E.g. when you see the letters HTTPs in your web-browser (typically you will see that next to a lock icon), it means that the web page you're viewing has been transmitted to you in encrypted form. In particular, that means that your data is supposed to protected from any kind of eavesdropping or prying eyes - and that goes both ways in this case. Any passwords you type in, the confidential data you provide to the website will in theory be encrypted, and be safe from someone trying to eavesdrop on you. In particular, what the 's' in HTTPs refers to, is that you're transmitting HTTP traffic that is encrypted by SSL. OpenSSL is an open-source implementation of the TLS protocol. It also happens to be the most widely deployed implementation. It is so widely spread that anyone who uses the web on a daily basis, is certain to interact with an OpenSSL implementation.

In OpenSSL versions 1.01 and up to 1.02 betas, there was a subtle but highly critical programming mistake that could lead to an attacker being able to gain access to the otherwise encrypted data. A system running one of the aforementioned versions of the OpenSSL implementation can be attacked quite easily. This wiki will describe both the attack and the flaw it exploits, as well as some of its ramifications. In particular, there's an extension to the TLS protocol known as Heartbeat. The Heartbeat allows you to keep a TLS session running even though no data has been communicated in a while, simply by sending a special message known as a heartbeat request. Before the introduction of Heartbeat, if there was a period without any communication between a client and server, the TLS session might have been terminated and would have to be reopened if any further data transfer was required. Heartbeat can also be used to check if the receiver of the signal is still present, and in that way keep the line of communication alive. The request itself consists of a payload containing the actual message, as well as some explicit information that specifies the size of the payload. The responding computer's answer will then contain the same payload and some padding. However, the vulnerability that went undiscovered for approximately two years, was that the computer receiving a heartbeat request would not check whether the information regarding size of the payload was actually accurate - it would blindly trust the information in the request. This allowed an attacker to send a very short message, for instance only a single byte of payload, while the message header informed that the payload actually had a size of 64KB. The code that handles the heartbeat extension within the OpenSSL library, simply copies the payload that the attacker provides into its memory. 
buffer exploit
This blind trust becomes a problem, since
