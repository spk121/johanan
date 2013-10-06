# Notes

In ITU-T T.105, it makes the following definitions:

The *input buffer* receives the user input, and which may be used in parallel with the echo handler.

The *output buffer* receives display data from received data packets and from the echo handler.

The *echo handler* control the echoplex procedure.

In the state ECHO_ACTIVE, all user input shall be echoed immediately on a character-by-character basis.

If parameter 24 is set, end-of-frame toggles echoing.

In ITU-T T.105, it suggest that the following X.3 parameters be used

| Number         | Description        | Required Values  | Default |
| -------------- | ------------------ | ---------------- | ------- |
| 2 (mandatory)  | Echo               | default          | 0       |
| 3 (mandatory)  | Data forwarding    | default + 1, 16  | 0       |
| 4 (mandatory)  | Idle timer delay   | default + 1      | 1       |
| 11 (mandatory) | Binary speed       |  default         |         |
| 23 (optional)  | Size of input field | default         | 0       |
| 24 (optional)  | End of frame       | default          | 32      |
| 25 (optional)  | Extended data forwarding signals | default | 0  |
| 26 (optional)  | Display interrupt  | default          | 0       |
| 28 (optional)  | Diacritic character editing | default | 0       |
| 29 (optional)  | Extended echo mast | default          | 0       |

If you combine the above table with the minimum requirements in X.3, you end up with the following list.

## Echo

|---|---------|
| 0 | no echo |
| 1 | echo    |

## Data forwarding characters

| 0  | No data forwarding characters |
| 1  | Alphanumeric |
| 2  | CR           |
| 16 | EXT, EOT     |
| 126 | All C0 controls + DEL |

## Idle timer delay
| 0   | infinite delay |
| 1   | 0.05 second |
| 20  | 1.00 second |
| 255 | 12.75 seconds |

## Binary speed

| 0  | 110 bits/s    |
| 2  | 300 bits/s    |
| 18 | 64,000 bits/s |

(I added 18 myself, because, the defaults are obsolete. )

## Size of input field

| 0        | Undefined size |
| 1 to 255 | Number of graphic characters |

## End of frame
| 0  | No end-of-frame signal |
| 32 | A frame is a complete packet sequence |

## Extended data forwarding signals

| 0 | No extended data forwarding |

## Display interrupt

| 0 | No display interrupt |

## Diacritic character ending

Obsolete?

## Extended echo mast

| 0 | No extended echo mask |

## Forwarding

Data in the input buffer is forwarded when of of the following occurs.
* a X.29 PAD (metadata) message is received;
* when the X.3 timer expires, as described in parameter 4;
* when the X.3 standard forwarding condition is received as described in parameter 3;
* when the X.3 extended data forwarding signal is received.

The default parameters are 0.05 seconds, no data forwarding character, and a 64,000 bits/s
binary speed.  This has the following effect.
* Every 0.05 seconds, the input buffer is checked for characters.
* If the input buffer is not empty, 400 bytes or all the characters in the input buffer, whichever is less, are packetized and sent. 400 bytes = 64,000 bits/s * (0.125 bytes/bit) * (0.05 seconds).
* In the 300 baud case, 2 bytes per packet would be sent.
* In the 110 baud case, 1 byte would be sent.
