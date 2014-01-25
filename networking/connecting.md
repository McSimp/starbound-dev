---
title: Connection Process
layout: base
---

# Connection Process

A client connects to a server through an exchange of packets, initiated by the server. The full process is documented in the following sections.

## Overview
There are five packets involved in the connection process. They are:

1. [Protocol Version](/networking/#0x01) 
2. [Client Connect](/networking/#0x07)
3. [Handshake Challenge](/networking/0x04)
4. [Handshake Response](/networking/0x09)
5. [Connection Response](/networking/0x02)

All five of these packets must be exchanged for a clean connection, even one that the server ends up refusing. **No** other packets may be sent during this period, or the connection will inevitably fail. 

The packets are covered below in the context of implementation. For a more in-depth look at the data structures, please view the [/networking/](Data Structures) page.

## Packets
Please note that all pseudo-code below will be operating on the extracted payload. The ID and Payload Size of each base packet is not factored in.

### 0x01: Protocol Version

This packet is sent by the server following a successful TCP handshake. The only data this contains is a protocol version, which is a 32-bit integer. The payload in hex from Angry Koala is `00000274`. This changes with each version.

After the client verifies the version, it sends a Client Connect packet.

### 0x07: Client Connect
The client connect packet is by far the most complex packet in terms of the handshake process. It is far larger than any of the others and contains the player info and a full shipworld file. The structure of the file is plotted in the Data Structures page, and the format of the files themselves is covered in the [File Formats](/file_formats/) section.

### 0x04: Handshake Challenge
The handshake challenge provides a claim message (currently unused), salt, and round count for the player to hash a password against. The handshake response is the password hash using these parameters. Due to the nature of this packet and the handshake response it is not possible to emulate a client without implementing the two packets dynamically, at a bare minimum.

The round count, in the current version, is set to 5000. Presumably this will be modifiable in the future, and indeed if you manually send the packets the client will respond with the correct hash. The salt is a randomly generated 32 character Base 64 string.

Example data for the handshake challenge packet:

* Salt: 5uAciRZkmwKUkek3krU+s2LTvPHE6v2P
* Claim Message: '' 
* Round Count: 5000

### 0x09: Handshake Response
The handshake response hash is generated using the SHA-256 algorithm, successively working on its own output. The format of the string to be hashed is the UTF-8 encoded string password+salt. 

Pseudo-code follows for an example implementation.

```
rounds = 5000
salt = "uAciRZkmwKUkek3krU+s2LTvPHE6v2P"
hash = "test_password"

while (rounds > 0) 
	message = hash+salt
	hash = sha256(message)
	rounds--

return hash
```

### 0x02: Connection Response
Assuming all previous packets have been sent, the client will receive a connection response. 
The first byte is a success flag. The following bytes are a client id, only applicable if the success flag is true, and then a rejection reason, only applicable if the success flag is false.