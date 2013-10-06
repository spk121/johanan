johanan
=======

*johanan* is a packet assembly and disassembly library for the Switched Virtual Circuit protocol

It breaks a constant stream of input into packets sent out at regular intervals, and it receives packets and buffers them into a stream-like interface.

This functionality is intended for clients that wish to communicate using the *jozabad* broker using the Switched Virtual Circuit protocol.


The name "Johanan" refers to a talented archer that fought with the biblical King David.  The image the archer is a metaphor for hub-to-node information transmission.

*Johanan* is the lowest level of a three layer application stack.
* *Jozabad* handles low-level communication.
* *Johanan* is a packet assembler and disassembler, expressing a stream-like interface (fprintf, fgetc) for intra-worker communcation based on *Jozabad* messaging.
* *Jahaziel* is a text protocol with in-band escapes for colors, graphics, and audio, and a rendering engine for that protocol.

## References

* It is inspired by the packet assembly and disassembly described in the ITU X.3 standard.
