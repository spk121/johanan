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

Data is forwarded when
* a X.29 PAD (metadata) message is received
* when the X.3 timer expires
* when the X.3 standard forwarding condition is received
* when the X.3 extended data forwarding signal is received.

## Echo

