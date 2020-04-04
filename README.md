# Bunker
Peer to Peer Encypted File Storage with Lightning Micro-Payments

## Outline

The application will store files in multiple ways:
- Locally on the device
- Encrypted and stored on other devices in exchange for Bitcoin Lightning micro-payments
- Encrypted and stored on other devices in exchange for backing up some of their data.

Each file will have a configurable "redundancy level" (how many copies to keep with peers) and an option as to whether to keep a local copy on the device. Files that are not kept locally will be hashed to a merkel root, which will be used to verify ownership of the data by other peers.

The remaining allocated space will be used to store files for peers, prioritizing first exchanging for files stored on their server, then prioritizing for highest fees per byte.

The device will keep a manifest file that it also duplicates with high redundancy. This file will be at a fixed path (/MANIFEST.db) and it will contain a list of all files kept, their merkel proofs, redundancy levels, and which peers have which files. It is also recommended this file be backed up by other means.

On a short interval, the node will ask for a random segment of a file it has stored on another server, along with a merkel proof. Upon success, it will send a micro-payment to the counterparty over lightning. Upon too many successive failures, it will move the file to a new peer. (This means that if no local copies are kept, the minimum redundancy level should be 2).

On a longer interval, the node will request the full file from the counterparty, and check it against the merkel root or decrypt it and verify its checksum. Once again if it is correct, it will send a micro-payment. This is necessary to prevent peers from hiking prices in the event that file recovery is needed. This kind of verification request will be indistinguishable from an attempt to recover data.

If a file is requested by the client, and a local copy is not available, it will be requested from a peer as above, decrypted, and streamed to the client.

If the device is corrupted, destroyed, or lost, and it needs a full recovery, it will do so as follows:
- Reach out to as many nodes as possible and ask for its manifest file. If the manifest file cannot be recovered, then it will try to recover all files in the same way. Nodes that recieve this kind of request can forward the request to their peers, and take a cut from the resulting fee. (this is not a desirable scenario)
- If the manifest is found, it will grab each file from the peers they are stored with.

## Open Questions
- How can we design things to prevent nodes from detecting and price gouging those in recovery scenarios?
- What other mechanisms can we use to back up the MANIFEST.db or a more minimal version of it (just the peer list). OP_RETURN?
