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

| Value      | Description |
|------------|------------ |
| 0 (default)| no echo     |
| 1          | echo        |

## Data forwarding characters


| Value      | Description |
|------------|------------ |
| 0 (default) | No data forwarding characters |
| 1  | Alphanumeric |
| 2  | CR           |
| 16 | EXT, EOT     |
| 126 | All C0 controls + DEL |

## Idle timer delay

| Value      | Description |
|------------|------------ |
| 0   | infinite delay |
| 1 (default)  | 0.05 second |
| 20  | 1.00 second |
| 255 | 12.75 seconds |

## Binary speed

The product of the binary speed and the idle timer delay determines the number of characters that appear in each data packet sent.

In X.3, this is usually a read-only property of a client.

| Value       | Nominal data rate | True bytes per 1/20 sec |
|------------ |-------------- | ----------------- | 
| 0           | 110 bits/s    | 1
| 2 (default) | 300 bits/s    | 2
| 3           | 1200 bits/s   | 8
| 4           | 600 bits/s    | 4
| 5           | 75 bits/s     | ½
| 6           | 50 bits/s     | ⅓
| 7           | 800 bits/s    | 5
| 12          | 2400 bits/s   | 15
| 13          | 4800 bits/s   | 30
| 14          | 9600 bits/s   | 60
| 15          | 19,200 bits/s | 120
| 16          | 48,000 bits/s | 300
| 17          | 56,000 bits/s | 350
| 18          | 64,000 bits/s | 400 
| 19          | 14,400 bits/s | 90 

(I added 18 myself, because, the defaults are obsolete. )

## Size of input field

| Value      | Description |
|------------|------------ |
| 0 (default)| Undefined size |
| 1 to 255 | Number of graphic characters |

## End of frame

I'm not sure if I'm going to implement this.

| Value      | Description |
|------------|------------ |
| 0  | No end-of-frame signal |
| 32 (default) | A frame is a complete packet sequence |

## Extended data forwarding signals

| Value      | Description |
|------------|------------ |
| 0 | No extended data forwarding |

## Display interrupt

| Value      | Description |
|------------|------------ |
| 0 | No display interrupt |

## Diacritic character ending

Obsolete?

## Extended echo mast

| Value      | Description |
|------------|------------ |
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
* If the bit rate is 64,000 bits/s and the input buffer is not empty, 400 bytes or all the characters in the input buffer, whichever is less, are packetized and sent. 400 bytes = 64,000 bits/s * (0.125 bytes/bit) * (0.05 seconds).
* In the 300 baud case, 2 bytes per packet would be sent.
* In the 110 baud case, 1 byte would be sent.


# API

joha\_channel\_t \*joha\_open (joha\_broker\_t \*broker, const char \*name, uint16\_t window\_size, joha\_packet\_size\_t psize, joha\_channel\_speed\_t speed);

Open a virtual circuit between this worker and another worker. NAME is the identifier for the other worker.  Window size, packet size, and speed are requested parameters for the connection.

Returns NULL on failure, setting joha\_error\_cause and joha\_error\_diagnostic.  On success, returns a new joha\_channel\_t structure.

uint16\_t joha\_window\_size(joha\_channel\_t \* stream);

size\_t joha\_read (void \* ptr, size\_t size, size\_t count, joha\_channel\_t \*channel );

size\_t joha\_write (const void \* ptr, size\_t size, size\_t count, joha\_channel\_t \* stream);

int joha\_putc (int character, joha\_channel\_t stream );

int joha\_getc (joha\_channel\_t stream );

joha_printf (joha_channel_t *stream, const char *format, ...);
