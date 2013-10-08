# API

Johanan is a client/server messaging scheme that it built on top of Jozabad primitives.  This messaging is similar to that described in ETS 300 223.

The basic functionality of a Johanan-based client will be
* The client sends an `SBV_Establish` to a server
* The server may query and/or set packet assembly parameters on the client
* The server may set function key definitions for the client
* The client and server exchange `SBV_VTX_DATA` packets
* One side disconnects with a `SBV_Release` request

| Service                | Confirmed | Client/Server | Function |
| ---------------------- | --------- | ------------- | -------- |
| `SBV_Establish`        | yes       | Client | Connection establishment |
| `SBV_Release`          | no        | Both   | Connection release |
| `SBV_Reset`            | yes       | Server | Reset to basic state |
| `SBV_VTX_DATA`         | no        | Both   | Videotex data packets |
| `SBV_Set_Param`        | no        | Server | Set packet assembly parameters |
| `SBV_Read_Param`       | no        | Server | Request packet assembly parameters |
| `SBV_Param_Indication` | no        | Client | Send packet assembly parameters |
| `SBV_TFI`              | yes       | Server | Terminal facility indicator |
| `SBV_TC_Error`         | no        | Both   | Error indication |
| `SBV_DFK`              | no        | Server | Definition of function keys |

## `SBV_Establish`

`SBV_Establish` is a message sent from client to server. 

It shall include the following parameters

| Parameter                  | Format |  Function |
| -------------------------- | -------| --------- |
| `OB_Called_Address`        | Called Party Number| the network address of the broker to be reached |
| `OB_Application_Selection` | string | the mnemonic of the a Videotex Application on the broker |
| `OB_User_Data`             | free   | binary data passed transparently to the broker |
| `IB_Called_Address`        | Called DTE Address | the address of the server | 
| `IB_Application_Selection` | string | the mnemonic of the a Videotex Application on the server |
| `IB_User_Data`             | free | binary data passed transparently to the server |

The `OB_Called_Address` will get packed into a `Joza_Setup` message that connects to the broker.

The `IB_Called_Address` will get packed into a `Joza_Call_Request` message

## `SBV_Release`

`SBV_Release` is a message sent by either client or server to close the connection

| Parameter           | Required | C->B | B->S | S->B | B->C | Function |
| ------------------- | -------- | ---- | ---- | ---- | ---- |--------- |
| `IB_Cause`          | yes      |     |     |  X    |  X   | the TCP address of the broker |
| `IB_Diagnostic`     | yes      |     |     |  X    |  X    | the name of the desired application |


## `SBV_Reset`

`SBV_Reset` is sent by the server to the client to request that it reset the connection.

It has no parameters

## `SBV_VTX_Data`

`SBV_VTX_Data` is a packet that contains Videotex message data.

It contains one field, which is `VTX_Data`

## `SBV_Set_Param`

`SBV_Set_Param` is sent by the server to the client to set one or more of the parameters of its packet assembler and disassembler.

It has one parameter: `X3_Parameter_List`.

## `SBV_Read_Param`

`SBV_Read_Param` is sent by the server to the client to request information about one or more of the parameters of its packet assembler and disassembler.

It has one parameter: `X3_Parameter_List`.

## `SBV_Param_Ind`

`SBV_Param_Ind` is sent by the client to the server in response to an `SBV_Read_Param` message.  It contains information about one or more of the parameters of its packet assembler and disassembler.

It has one parameter: `X3_Parameter_List`.

## `SBV_TFI`

## `SBV_TC_Error`

This is sent by the server to the client to inform the client that its last message was erroneous.  It has two parameters.

| Parameter           | Required | C->B | B->S | S->B | B->C | Function |
| ------------------- | -------- | ---- | ---- | ---- | ---- |--------- |
| `IB_Cause`          | yes      |     |     |  X    |  X   | the TCP address of the broker |
| `IB_Diagnostic`     | yes      |     |     |  X    |  X    | the name of the desired application |

## `SBV_DFK`

This is sent by the server to the client to inform the client about the behavior of the function keys.  This message will inform the client of a string that it may send back to the server to inform it that a function key has been pressed.  (This string is information the client would send to the server as part of the text sent in-band as part of a `SBV_VTX_Data` message.)

It has two parameters

| Parameter           | Required | C->B | B->S | S->B | B->C | Function |
| ------------------- | -------- | ---- | ---- | ---- | ---- |--------- |
| `Function_Keys`          | no      |     |     |  X    |  X   | a list of function keys with their associated strings |
| `Reset_Keys`     | no      |     |     |  X    |  X    | a list of function keys to be reset |

# Notes on Packet Assembly

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

## Size of input field

| Value      | Description |
|------------|------------ |
| 0 (default)| Undefined size |
| 1 to 255   | Number of graphic characters |

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

# Formats

## Called Party Number

ETSI 300 079 says `OB_Called_Address` maps to  *Called Party Number*

ETS 300 102-1 4.5.8 describes the format of a *Called Party Number*

The max length of a Called Party Number sub-message is 23 bytes.

| Byte | Bits |Value |
| ----- | ----- |
| 1    | 1 - 7 | Called party number information element identifier = 112
| 1   | 8       | 0 |
| 2    | 1 - 8 | Length of called party number contents |
| 3    | 1 - 4 | Numbering plan identification | 
| 3    | 5 - 7 | Type of number | 
| 3    | 8     | 1 |
| 4+   | 8    | ISO 646 characters |

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

