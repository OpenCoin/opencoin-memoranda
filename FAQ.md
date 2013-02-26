This document contains additional explanations to the [protocol specification](https://github.com/OpenCoin/opencoin-memoranda/blob/master/opencoin-v3.txt), it describes why certain design decisions have been made, which alternative options have been evaluated and their pros and cons.


What is OpenCoin?
-----------------
opencoin is a use case agnostic protocol which can be used for a digital 
cash system but also for other usages such as for balloting and vouchers. 
Hence, the vocabulary used in the protocol specification intends to be 
practical and easy to understand but at the same time not sound to 
describe a financial system. 


What is it about tokens, blinds, payloads, blanks, coins?
---------------------------------------------------------
In explaining opencoin in non-technical terms we say: "you put a blank 
in an envelope, stamp through the envelope (blind signature), open
the envelope (unblind) and you get the coin". 
As nice as the metapher is, it doesn't quite work when defining the 
protocol. As we have a data container that is never changed (unlike
the blank, which gets transformed by minting), and that data container
only gets combined with a signature into a coin, we used the term 
payload instead. Blank might give the impression that its a container 
that gets modfied in someway, but no modification takes place.

So, a Payload without signature is equal to a Blank, a Blind contains 
the blinded hash of the Payload, a Blind Signature object holds the 
blind signature, and a Coin consists of a Payload, and a Signature 
(which is the unblinded blind signature of the Payload).


There are optional fields on some of the messages.
--------------------------------------------------
The alternative would be to have more different messages. As we work
with a challenge response protocol, we wanted it to be clear what 
response object to expect for a message. Its a question of the right 
mixture of amount of messages, and number of optional fields.


Having field xyz in the protocol would be useful. Why isn't it there?
---------------------------------------------------------------------
We try to create the protocol with as few fields as possible, but as
many as necessary. Its tempting to add a field here and there, but
then we suddenly have information/data bloat. So the idea is to keep
it simple, and try to find out whats needed (instead of nice).


(Implementation) Why is the id in the DSDB not just the serial
--------------------------------------------------------------
An attacker could otherwise block arbitary serial numbers. Its an 
expensive attacks (the serials have to be signed to get into the 
DSDB), but we rather prevent this.


What is the Risk of Denial of Service (DoS) Attacks?
----------------------------------------------------
Because the issuer service performs computing-complex cryptographic 
operations and is accessible by every internet user, methods to prevent 
DoS have to be taken seriously. The fact that not the actual sender is 
requesting mint's service but a third party (the receiver in a peer-to-
peer transaction) makes network-based DoS protection methods more 
difficult to apply. For instance an attacker may send false coins to 
an online shop or another legitimate receiver, the receiver will ask 
the issuer to refresh the coins. In this scenario the attacker has no 
direct connection to the mint but receivers may be misused as proxies.


Does Opencoin Provide Anonymity
-------------------------------
Untracability for issuer of peer-to-peer transactions is important. 
But opencoin doesn't contain any active anonymization measures.

* Withdraw/validate and redeem might require authentication
* Exchange: we might log IPs and dont want to block TOR.


How Are Peer-to-Peer Transfers Been Performed?
----------------------------------------------
Opencoin doesn't care, it's an implementation issue.


What are the Reasons to Encrypt Coin Containers?
------------------------------------------------
* To prevent copying of coins ("theft")
* To protect user's privacy
* To prevent manipulation


What should be visible on the outside of encrypted containers?
--------------------------------------------------------------
* Whatever the recipient needs to make use of container
* The fact that its opencoin
* The key-ID used to encrypt the container
* Maybe a comment or subject or similar...


Isn't the usage of opencoin in desktop browser insecure?
--------------------------------------------------------
Its a known-to-be-untrusted environment, hence at users own risk.


How can a peer verify a coin's validity?
----------------------------------------
The issuer has no insight into the coin it signs because it's a 
blind signature. Hence a correct signature of the issuer over the 
coin's hash does not guarantee that the elements (denomination, 
validity period) are correct. In fact they could be falsified. A 
peer (e.g. receiver in a peer-to-peer transaction) must verify

* The coin's signature
* The denomination and validity period associated with the issuer 
key used to sign the coin. These should match the denomination and 
validity stated within the coin.
* The actual denomination results by dividing the denomination with 
the denomination_divisor.
* In addition to verify that a coin hasn't been (double) spent, the 
receiver needs to refresh the coin at the Renew service.


Why then perform the signature over the coin's meta data and not over the coin's random serial only?
----------------------------------------------------------------------------------------------------
Because in this case an attacker could easily choose a random value 
and calculate the signature backwards. The signature over the meta 
data's hash ensures that it has been performed by the secret mint 
key only.


How to find out the validity period of a coin?
----------------------------------------------
The key which has signed a coin is associated with a certain currency, 
denomination, and validity period. This validity period applies to the 
coin.


How is the coin serialized/normalized before it's hash is calculated?
---------------------------------------------------------------------
Using [Bencode](https://en.wikipedia.org/wiki/Bencode).


Why are big integers (e.g. hashes, keys, signatures) encoded in hex?
--------------------------------------------------------------------
We were considering Base64 as an alternative to hex:

*Pro hex:*

* Hex is supported out of the box by big integer libraries and other 
crypto tools.
* The message size is reduced by compressing the entire message, so 
that the larger hex size isn't that significant anymore.

*Pro Base64:*

* Base64 is more efficient (smaller) than Hex.
* Consistent to have only one encoding instead of two different (hex 
and Base64)
* (A Base64 converter is required for the envelope anyway, so no 
additional effort to implement a Base64 converter.)


Why is a message-based API used instead of a RESTful approach?
--------------------------------------------------------------
The opencoin specification follows the message-based approach but a 
RESTful issuer implementation is available as well.

*Pro RESTful:*

* Low entry barrier and easy integration to other web-based applications
* Humans can directly interact, test and verify just by using a web 
browser
* Simpler implementation. For instance the implementation of message containers would be reduced from 106 lines to 23 lines in Scala.
* Simpler protocol specification: The container section describing RESTful requires only 40 lines vs. 108 lines for message-based approach.
* It's the right fit to the current use cases which all rely on HTTP and don't use any other transport protocol. No additional complexity for speculative future requirements.
* Easily evolves over long term. Upgrades can be incorporated easily without braking backward compatibility by specifying the API version as part of the URL. As opposed the message-based approach requires a version field in each message.

*Pro message-based:*

* Independence of transport protocol (HTTP) allows using other protocols in the future
* Allows transport-protocol-independent end-to-end encryption and signing of messages
* Allows easier censorship-circumvention measures through proxying


What is the Average Amount of Coins Used?
-----------------------------------------
The following overview shows the average amount of coins used for a particular transaction amount by using the denominations 1, 2, 5, 10, 20, 50, 100, 200, 500, and 1000.

range 1 to 200
{1: 1000, 2: 2522, 3: 3900, 4: 4089, 5: 3162, 6: 2063, 7: 1324, 8: 817, 9: 473, 10: 272, 11: 164, 12: 84, 13: 28, 14: 2}
Average: 4 coins

range 200 to 500
{1: 1013, 2: 2763, 3: 5219, 4: 7977, 5: 10395, 6: 11171, 7: 9513, 8: 6561, 9: 4168, 10: 2647, 11: 1615, 12: 954, 13: 556, 14: 318, 15: 142, 16: 38}
Average: 6 coins

range 500 to 1000
{1: 1013, 2: 2763, 3: 5645, 4: 10846, 5: 19748, 6: 29878, 7: 34619, 8: 30350, 9: 21215, 10: 13306, 11: 8376, 12: 5305, 13: 3237, 14: 1907, 15: 1100, 16: 626, 17: 282, 18: 82, 19: 2}
Average: 7 coins

range 1000 to 1500
{1: 1013, 2: 2763, 3: 5645, 4: 11272, 5: 22567, 6: 38911, 7: 52436, 8: 54116, 9: 43924, 10: 30158, 11: 19610, 12: 12949, 13: 8411, 14: 5206, 15: 3067, 16: 1782, 17: 1054, 18: 510, 19: 154, 20: 2}
Average: 8 coins

range 1500 to 2000
{1: 1013, 2: 2763, 3: 5645, 4: 11272, 5: 22993, 6: 41730, 7: 61469, 8: 71933, 9: 67690, 10: 52867, 11: 36462, 12: 24183, 13: 16055, 14: 10380, 15: 6366, 16: 3749, 17: 2210, 18: 1282, 19: 582, 20: 154, 21: 2}
Average: 8 coins

range 2000 to 5000
{1: 1013, 2: 4376, 3: 21475, 4: 86196, 5: 246278, 6: 505195, 7: 766216, 8: 882479, 9: 795939, 10: 591134, 11: 391463, 12: 249573, 13: 156228, 14: 95697, 15: 58763, 16: 36802, 17: 22967, 18: 13867, 19: 8135, 20: 4668, 21: 2522, 22: 1050, 23: 262, 24: 2}
Average: 8 coins

range 5000 to 10000 {1: 1013, 2: 4376, 3: 21475, 4: 86196, 5: 249891, 6: 541025, 7: 938753, 8: 1409281, 9: 1925824, 10: 2384863, 11: 2569686, 12: 2332616, 13: 1786294, 14: 1202304, 15: 759514, 16: 474163, 17: 296182, 18: 182843, 19: 111121, 20: 67666, 21: 41781, 22: 25589, 23: 15125, 24: 8607, 25: 4776, 26: 2522, 27: 1050, 28: 262, 29: 2}
Average: 11 coins 