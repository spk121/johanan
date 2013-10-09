# Terms

*out-band* - anything with no analog in X.29.  For us, *out-band* is anything beyond connection between a client and server.

*in-band* - anything with an analog in X.29: creating a virtual circuit between a client and server.



# API

Johanan is a client/server messaging scheme that it built on top of Jozabad primitives.  This messaging is similar to that described in ETS 300 223 or ITU T.105.

Clients and servers will register themselves with the broker using the Jozabad `CONNECT` messages.

Basic Jozabad workflows are tailored for this case. The basic functionality of a Johanan-based client will be
* The client sends an `SBV_Establish` to a server
* The server will respond with an `SBV_Establish` indication.
* The server may query and/or set packet assembly parameters on the client
* The server may set function key definitions for the client
* The client and server exchange `SBV_VTX_DATA` packets
* One side disconnects with a `SBV_Release` request

| Service                | Jozabad Msg  | Confirmed | Client/Server | Function |
| ---------------------- | ------------ | ----------| ------------- | -------- |
| `SBV_Establish` request       | `CALL_REQUEST`  | yes       | Client        | Connection establishment |
| `SBV_Establish` indication    | `CALL_ACCEPTED` | ...        | Client        | Connection establishment |
| `SBV_Release`          | `CLEAR_REQUEST` | no        | Both   | Connection release |
| `SBV_Reset` request    | `RESET_REQUEST` | yes       | Server | Reset to basic state |
| `SBV_Reset` indication | `RESET_CONFIRMATION` | ...       | Server | Reset to basic state |
| `SBV_VTX_DATA`         | `DATA` (Q=0)      | no        | Both   | Videotex data packets |
| `SBV_Set_Param`        | `DATA` (Q=1)      | no        | Server | Set packet assembly parameters |
| `SBV_Read_Param` request | `DATA` (Q=2)    | yes        | Server | Request packet assembly parameters |
| `SBV_Read_Param` indication | `DATA` (Q=3) | ...        | Client | Send packet assembly parameters |
| `SBV_TFI` request      | `DATA` (Q=4) | yes       | Server | Terminal facility indicator |
| `SBV_TFI` indication   | `DATA` (Q=5) | ...       | Server | Terminal facility indicator |
| `SBV_TC_Error`         | `DATA` (Q=6) | no        | Both   | Error indication |
| `SBV_Define_Function_Keys` | DATA (Q=7) | no        | Server | Definition of function keys |
| `SBV_Reset_Function_Keys` | DATA (Q=8) | no        | Server | Clearing the definition of function keys |

## `SBV_Establish`

Same as Jozabad `CALL_REQUEST` and `CALL_ACCEPTED`

## `SBV_Release`

Same as Jozabad `CLEAR_REQUEST` with the caveat that `CLEAR_CONFIRMATION` messages are ignored.

## `SBV_Reset`

Same as Jozabad `RESET_REQUEST` and `RESET_CONFIRMATION`

## `SBV_VTX_Data`

Send in a Jozabad `DATA` message with `Q` = 0.

## `SBV_Set_Param`, `SBV_Read_Param` request, `SBV_Read_Param` indication

Send in a Jozabad `DATA` message with `Q` = 1, 2 or 3 respectively.

The payload contains an X.3 Parameter list: one or more PAD parameter ID / value pairs.

In the `SBV_Read_Param` case, the value part of the pair is ignored.

The parameter list is in `X.3_Parameter_List` format as described in X.29.

## `SBV_Define_Function_Keys`, `SBV_Reset_Function_Keys`

This is sent by the server to the client to inform the client about the behavior of the function keys.  This message will inform the client of a string that it may send back to the server to inform it that a function key has been pressed.  (This string is information the client would send to the server as part of the text sent in-band as part of a `SBV_VTX_Data` message.)

This is as described in T.105.

# Notes on Packet Assembly

Packet assembly is mostly inspired by ITU X.3 and ITU-T T.105.

In ITU-T T.105, it makes the following definitions:

The *input buffer* receives the user input, and which may be used in parallel with the echo handler.

The *output buffer* receives display data from received data packets and from the echo handler.

Through `SBV_Set_Param` messages, a server may tell a client when to send messages from its input buffer.  The contents of the input buffer may be sent at regular intervals or when a specific character occurs.

The *echo handler* control the echoplex procedure.

The server may also tell the client if client input is supposed to be echoed automatically to the client output.

In the state ECHO_ACTIVE, all user input shall be echoed immediately on a character-by-character basis, except for function keys.

In ITU-T T.105, it suggest that the following X.3 parameters be used

| Number         | Description        | Required Values  | Default |
| -------------- | ------------------ | ---------------- | ------- |
| 2 (mandatory)  | Echo               | default          | 0       |
| 3 (mandatory)  | Data forwarding    | default + 1, 16  | 0       |
| 4 (mandatory)  | Idle timer delay   | default + 1      | 1       |
| 11 (mandatory) | Binary speed       |  default         |         |
| 23             | Size of input field | all             | 0       |
| 29 (optional)  | Extended echo mask | default          | 0       |

If you combine the above table with the minimum requirements in X.3, you end up with the following list.

## Echo

When echo is enabled all characters except those in the extended echo mask are echoed from the input buffer to the output buffer.

| Value      | Description |
|------------|------------ |
| 0 (default)| no echo     |
| 1          | echo        |


## Data forwarding characters

When data forwarding characters are set, a packet is sent immediately once of the indicated characters is encountered.

| Value      | Description |
|------------|------------ |
| 0 (default) | No data forwarding characters |
| 1  | Alphanumeric |
| 2  | CR           |
| 16 | EXT, EOT     |
| 126 | All C0 controls + DEL |

## Idle timer delay

When the idle timer delay is set, a packet is sent at regular intervals if a character is ready.

| Value      | Description |
|------------|------------ |
| 0   | infinite delay |
| 1 (default)  | 0.05 second |
| 20  | 1.00 second |
| 255 | 12.75 seconds |

## Binary speed

The product of the binary speed and the idle timer delay determines the number of characters that appear in each data packet sent for those packets that are sent as a consequence of the idle timer.

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

## Size of input field

The input field, when set, indicates that a packet is sent once it contains the indicated number of characters.

| Value      | Description |
|------------|------------ |
| 0 (default)| Undefined size |
| 1 to 255   | Number of graphic characters |


## Extended echo mask

This is a bitfield that describes the characters that are note echoed from the input buffer to the output buffer when echoing is enabled.

| Value      | Description |
|------------|------------ |
| 0 | No extended echo mask |
| 1 | No echo of CR |
| 2 | No echo of LF |
| 4 | No echo of VT, HT, FF |
| 8 | No echo of BEL, BS |
| 16 | No echo of ESC, ENQ |
| 32 | No echo of ACK, NAK, STX, SOH, EOT, ETB, ETX |
| 64 | (n/a) |
| 128 | No echo of remaining C1 controls and DEL |

# Data Formats

## Coding of the X.3 Parameter List

When a `DATA` packet contains an X.3 parameter list, it is 2 to 12 bytes.  Each pair of bytes is an X.3 parameter.  The first byte is the parameter ID: 2 = Echo, 3 = Data Forwarding, 4 = Idle Timer Delay, etc.  The second byte in the pair is the value for the parameter as described above.

## Coding of the Define Function Keys parameter

Quoting and adapting from T.105

The Define Function Keys parameter is a structured data type. It carries a list of function key definitions. Each
function key consists of an identification, an optional user visible name, an optional code sequence and an optional
“do-not-forward” indication.

This table defines the coding of the type indicators
| Type indicator | Data element |
| -------------- | ------------- |
| 0x60           | Function key |
| 0x61           | Identification |
| 0x62           | User-visible name |
| 0x63           | Code sequence |
| 0x64           | Do not forward |

The user visible name may be used by the client to inform the user about the purpose of a function key. The
code sequence is a sequence of octets which has to be sent to the server when the function key is depressed.

By default, the function key shall be associated with a forwarding condition (causing an input data packet to be sent immediately once a function key is pressed. The presence of a do-not-forward indication defines that the function key shall not be associated with a forwarding condition.

The client shall process the conditions in the received order. If the server sends an erroneous message that sets a specific function key twice in one command, the value of the last received definition shall be used.

The list of function keys is coded as a structure. Each element of this structure is coded in a TLV (type-length-value)
form.

The identification is coded as an integer. For clients supporting the soft function keys service, at least values 1 to 8 shall be supported. The user visible name shall be encoded as a string of the indicated length. The code sequence shall be encoded as a sequence of octets of the indicated length. The values of these octets are not limited to a specific data syntax. 

The do-not-forward indication shall be encoded as a void type.

The following formal specification gives the syntax of the Function_Keys data-structure:
```
Function_Keys ::= Function_Key_List
Function_Key_List ::= Function_Key Function_Key_List 
                      | /* empty */ 
Function_Key ::= 0x60 Length Identification
                      User_visible_name
                      Code_sequence
                      Do_not_forward
Identification ::= 0x61 Length Integer
User_visible_name ::= 0x62 Length String
                  | /* empty */ 
Code_sequence ::= 0x63 Length Octet_sequence
              | /* empty */ 
Do_not_forward ::= 0x64 0x00
              | /* empty */
Length ::= 1-byte unsigned integer
Integer ::= 1-byte unsigned integer
String ::= ASCII-encoded string of 255 bytes or less
Octet_sequence ::= list of 8-bit unsigned integers of 255 bytes or less
```

The following example illustrate the use of the define funciton key service, showing the complete coding for a function key.

EXAMPLE: Key 1 with the user visible name “F1” and the code string
“Code1”, and key 10 with the user visible name “F10” and the code sequence “Code2”.

| code | meaning |
| ----- | ------ |
| 0x60 | function key begins |
| 0x0E | length = 14 |
| 0x61 | identification begins |
| 0x01 | identification length = 1 |
| 0x01 | function key #1 |
| 0x62 | function key name begins |
| 0x02 | function key name length = 2 |
| "F"  | first letter is "F" |
| "1"  | second letter is "1" |
| 0x63 | function key code sequence begins |
| 0x05 | function key code length = 5 |
| "C"  | function key code first letter is "C" |
| "o"  | function key code second letter is "o" |
| "d"  | function key code third letter is "d" |
| "e"  | function key code fourth letter is "e" |
| "1"  | function key code fifth letter is "1" |
| 0x60 | function key begins |
| 0x0E | length = 15 |
| 0x61 | identification begins |
| 0x01 | identification length = 1 |
| 0x0A | function key #10 |
| 0x62 | function key name begin |
| 0x03 | length = 3 |
| "F"  | first letter is "F" |
| "1"  | second letter is "1" |
| "0"  | third letter is "0" |
| 0x63 | function key code sequence begins |
| 0x05 | code sequence length = 5 |
| "C"  | code sequence first letter is "C" |
| "o"  | code sequence second letter is "o" |
| "d"  | code sequence third letter is "d" |
| "e"  | code sequence fourth letter is "e" |
| "2"  | code sequence fifth letter is "2" |

## Called Party Number

ETS 300 079 says `OB_Called_Address` maps to  *Called Party Number*

ETS 300 102-1 4.5.8 describes the format of a *Called Party Number*

The max length of a Called Party Number sub-message is 23 bytes.

| Byte | Bits | Value |
| ---- | ---- | ------- |
| 1    | 1 - 7 | Called party number information element identifier = 112 |
| 1    | 8     | 0 |
| 2    | 1 - 8 | Length of called party number contents |
| 3    | 1 - 4 | Numbering plan identification | 
| 3    | 5 - 7 | Type of number | 
| 3    | 8     | 1 |
| 4+   | 1 - 8 | ISO 646 characters |

Numering plan identification is an enumerated type

| Value | Description |
| ------ | ------- |
| 0 | unknown |
| 1 | ISDN/Telephony numbering plan (E.164)
| 3 | Data numbering plan (X.121)
| 4 | telex numbering plan (F.79)
| 8 | national standard numbering plan
| 9 | private numbering plan
| 15 | reserved for extension

Type of number is an enumerated type

| Value | Description |
| ----- | ----- |
| 0 | unknown
| 1 | international number
| 2 | national number
| 3 | networke specific number
| 4 | subscriber number
| 6 | abbreviated number
| 7 | reserved for extension

This is all obsolete and not worth implementing.

## Called DTE Address

ETS 300 079 says `IB_Called_Address` maps to  *Called DTE Address*

ISO 8208 §12.2.1.1.3 supposedly describes the format of a *Called DTE Address*, but, I couldn't find a copy of ISO 8208.  It is supposed to be similar to X.25.

X.25 §5.2.1.1 describes a super-complicated meta-format that covers a dozen or so different obsolete address formats.

Not worth implementing.

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


# Mapping of Johanan messages onto Jozabad primitives

With Jozabad, the most complicated operation is establishing a connection.  First a client connects to a broker, declaring its directionality.  Second, the client connects to another named worker, declaring window size, packet size, and connection speed.

The Johanan method `SBV_Establish` is supposed to set up a complete connection.  It goes like this...

* On the calling client, Johanan converts the `SBV_Establish` into a `Joza_Call_Request` with a calling address, called address, window size, packet size, connection speed.  The connection speed is a read-only property of the client set at launch.  It will usually be 64,000 bits/sec or 300 bits/sec by default.  The packet size is based on a lookup table of the connection speed.  The window size is usually two seconds worth, which would be 40 for most speeds, 20 for 75 bits/sec, and 14 for 50 bits/sec.
* The `Joza_Call_Accepted` message comes back, with updated window size, packet size, and connection speed.   Johanan splits this call accepted into an `SBV_Establish` response and `SBV_Set_Param` response to the client.  For the latter, Johanan is updating this client's packet assembly parameters to acceptable defaults for this packet size and connection speed.
* The main loop is active at this point.

The `SBV_VTX_Data` method is when the client sends data.
* On the calling client, Johanan takes the contents of `SBV_VTX_Data` and puts it into a buffer. Every 1/20 second (or 2/20 for 75 baud or 3/20 for 50 baud) Johahan runs the data forwarding check, and if there is data to forward, it ships  `Joza_Data` message to the broker if its window is open.  If it can't send the data, and the buffer fills up, it sends a `SBV_TC_Error` back to the client.
* On the calling client, if a `Joza_Data` message is received, its contents are put into a buffer.  A `Joza_RR` message is sent in response if the buffer has empty space for an entire window's worth of data.  Every 1/20 seconds (or less for 75 bps or 50bps) Johanan does the data forwarding check and if there is data, it sends `SBV_VTX_Data` to the client.

