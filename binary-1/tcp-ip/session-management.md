# Session Management

## **Overview**

Clients initiate a TCP/IP session to the Binary EOS \(Order Entry Server\). Messages may be exchanged between the client and server after successful authentication. After one unsuccessful attempt, the server will automatically drop the connection. All binary data is sent in network Endian format \(Big Endian\). During periods of inactivity, the server and client use Heartbeat messages to ensure the connection is up and functioning properly. The client needs to send Heartbeat messages to remain connected. 

## **Disconnects**

TCP/IP connections can experience disconnections for a variety of reasons. Each order message has a message sequence number to ensure no messages are lost when the connection drops. Each client assigns sequence numbers to the messages they send to the server and the server tracks them.

## Logon/Logout to Admin Server \(AES\)

Clients use the Logon message to establish a connection, identify the message version profile they will use for the client session and identify the last response message they have processed. The server may accept or reject the client logon.

### BOClientLogon -- Client Sending

1. The BOClientLogon message must be sent to the AES in order to initiate the logon process \(please contact BO Representative for IP address and port\).
2. Please refer to the BOClientLogon with logon status and if logon was successful the IP Address and Port of the OES \(Order Entry Server\).
3. The AES will respond with a BOClientLogon with logon status and if logon was successful the IP Address and Port of the OES \(Order Entry Server\).
4. Only one login session is permited for a unique account ID and UserName.
5. Black Ocean requests that if they user is going to close the connec\)on a BOClientLogon message should be sent with the LogonType set to 2 prior to closing the connection in order to allow the OES to close the connec\)on gracefully.
6. BOClientLogon Example Message - Client Sending

| Field Name | Data Type | Data Length | Buffer Offset | Required Field | Required Value | Example Value | Notes |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Data1** | char | 1 | 0 | X | H | H | Header |
| **Data2** | char | 1 | 1 |  |  |  | Header |
| **Data3** | short | 2 | 2 | X | 143 | 143 | Header |
| **LogonType** | short | 2 | 4 | X |  | 1 | Note1 |
| **Account** | Int | 4 | 6 | X |  | 253336 | Note2 |
| **2FA** | char\[\] | 6 | 10 | X |  | 1F6A | Note4 |
| **UserName** | char\[\] | 6 | 16 | X |  | BOU1 | Note2 |
| **TradingSessionID** | Int | 4 | 22 | \* |  |  | Note2 |
| **PrimaryOESIP** | char\[\] | 24 | 26 | \* |  |  | Note3 |
| **SecondaryOESIP** | char\[\] | 24 | 50 | \* |  |  | Note3 |
| **PrimaryMDIP** | char\[\] | 24 | 74 |  |  |  | Note Used |
| **SecondaryIP** | char\[\] | 24 | 98 |  |  |  | Not Used |
| **SendingTime** | Long | 8 | 122 |  |  |  | Note 5 |
| **MsgSeqNum** | Int | 4 | 130 |  |  | 1500201 |  |
| **Key** | Int | 4 | 134 |  |  | 432451 |  |
| **LoginStatus** | short | 1 | 138 |  |  |  |  |
| **RejectReason** | short | 2 | 140 |  |  |  |  |
| **RiskMaster** | char | 1 | 142 |  |  |  |  |

#### Notes:

1. LogonType is a short enum, values: `Login 1`, `Logout 2`
   * If the value is not one of the values above, a logout message will be sent and the connection closed.
2. Value assigned by Black Ocean. If this is a logout, the TradingSessionID must be supplied.
3. IP address and port of the OES will be sent in the server response BOClientLogon message.
4. 2FA is disabled for the testing phase.
5. Sending times are in nanoseconds from the epoch, January 1, 1970.

### BOClientLogon/Logout -- Server Response

1. Upon Receipt of a client logon, the server will respond with a BOClientLogon message.
2. The status of the logon will be in the LoginStatus ﬁeld, a value of 1 indicates the logon was successful, a value of 2 indicates the login failed and the server will then close the connection. The reason for the unsuccessful login will be in the RejectReason ﬁeld. Possible values for the reject will be detailed in a section below.
3. The IP address and port of the OES \(Order Entry Server\) will be supplied in the message in order to allow the user to make the connection to the OES. Login to the OES will be covered in a subsequent section.
4. The server may initiate logout messages in the following circumstances:
   1. Matching Engine switching to secondary server \(if it is deemed operationally more eﬃcient in the event of a loss of the primary Matching Engine, the OES may switch to the secondary OES where the new primary Matching Engine is located\).
   2. Malformed messages which cannot be processed.
   3. Black Ocean initiated shutdown of the OES
5. In the event the OES becomes unavailable, the user should reconnect to the secondary OES for which the IP Address and Port is supplied in the message.
6. In the event the client sends a logout message, the server will not respond with a logout message but will instead proceed to close the connection.

| Field Name | Data Type | Data Length | Buffer Offset | Required Field | Required Value | Example Value | Notes |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Data1** | char | 1 | 0 | X | H | H | Header |
| **Data2** | char | 1 | 1 |  |  |  | Header |
| **Data3** | short | 2 | 2 | X | 143 | 143 | Header |
| **LogonType** | short | 2 | 4 |  |  | 1 | Note 1 |
| **Account** | Int | 4 | 6 | \* |  | 253336 | Note 2 |
| **2FA** | char\[\] | 6 | 10 | X |  | 1F6A | Note 3 |
| **UserName** | char\[\] | 6 | 16 | X |  | BOU1 | Note 2 |
| **TradingSessionID** | Int | 4 | 22 | X |  | 505 | Note 2 |
| **PrimaryOESIP** | char\[\] | 24 | 26 | \* |  | 192.0.0.1:4300 5 | Note 4 |
| **SecondaryOESIP** | char\[\] | 24 | 50 | \* |  | 113.50.1.1:430 05 | Note 4 |
| **PrimaryMDIP** | char\[\] | 24 | 74 |  |  |  | Not Used |
| **SecondaryIP** | char\[\] | 24 | 98 |  |  |  | Not Used |
| **SendingTime** | Long | 8 | 122 |  |  |  | Note 5 |
| **MsgSeqNum** | Int | 4 | 130 | X |  | 1500201 |  |
| **Key** | Int | 4 | 134 |  |  | 432451 |  |
| **LoginStatus** | short | 1 | 138 |  |  |  | Not Used |
| **RejectReason** | short | 2 | 140 | \* |  |  | Note 6 |
| **RiskMaster** | char | 1 | 142 |  |  |  | Note 7 |
|  |  | **Total Length** | **143** |  |  |  |  |

#### Notes:

1. LogonType is a short enum \(see Client logon above for possible values\)
2. TradingSessionID, UserName and Account number included in logon response but may or may not be included in logout messages.
3. Disabled for testing.
4. Not included in logout messages \(LogonType = 2\)
5. Sending times are in nanoseconds from the epoch, January 1, 1970.
6. If the login was rejected, the reject reason will be in the ﬁeld RejectReason
7. Not currently used, for future expansion.

### Logon/Logout Reject Codes

UTPDirect sends a Logon Reject message only when logon validation fails and then closes the connection. If logon was successful, UTPDirect sends a Logon message back to the client.

| Error Code | Value |
| :--- | :--- |
| **USER\_NOT\_FOUND** | 2 |
| **ACCOUNT\_NOT\_FOUND** | 3 |
| **INVALID\_KEY** | 4 |
| **ACCOUNT\_DISABLED** | 5 |
| **RISK\_ACCOUNT\_NOT\_FOUND** | 7 |
| **MES\_NOT\_AVAILABLE\_TRADING\_DISABLED** | 9 |
| **OES\_NOT\_AVAILABLE\_TRADING\_DISABLED** | 10 |
| **MDS\_NOT\_AVAILABLE\_TRADING\_DISABLED** | 11 |
| **MSG\_TYPE\_INVALID** | 12 |
| **NO\_RISK\_DATA** | 44 |
| **MSG\_SEQ\_NUM\_INVALID** | 52 |
| **USER\_ALREADY\_LGGGED\_IN** | 53 |
| **MALFORMED\_MSG** | 54 |
| **SENDING\_TIME\_INVALID** | 21 |

## BOClientLogin to Order Entry Server \(OES\)

After successfully logging into the AES, the user should log into the OES using the IP address and port provided in the AES login response.

### BOClientLogon -- Client Sending 

1. The BOClientLogon message must be sent to the OES in order to initiate the logon process. The IP address and port were provided to the user during the AES login.
2. Please refer to the BOClientLogon excel spreadsheet for exact ﬁelds, datatypes and buﬀer oﬀsets.
3. Only one login session is permitted for a unique account ID and UserName.
4. Black Ocean requests that if they user is going to close the connection a BOClientLogon message should be sent with the LogonType set to 2 prior to closing the connection in order to allow the OES to close the connection gracefully.
5. BOClientLogon Example Message - Client Sending

| Field Name | Data Type | Data Length | Buffer Offset | Required Field | Required Value | Example Value | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Data1** | char | 1 | 0 | X | H | H | Header |
| **Data2** | char | 1 | 1 |  |  |  | Header |
| **Data3** | short | 2 | 2 | X | 143 | 143 | Header |
| **LogonType** | short | 2 | 4 | X |  | 1 | Note 1 |
| **Account** | Int | 4 | 6 | X |  | 25336 | Note 2 |
| **2FA** | char\[\] | 6 | 10 | X |  | 1F6A | Note 4 |
| **UserName** | char\[\] | 6 | 16 | X |  | BOU1 | Note 2 |
| **TradingSessionID** | Int | 4 | 22 | \* |  |  | Note 2 |
| **PrimaryOESIP** | char\[\] | 24 | 26 | \* |  |  | Note 3 |
| **SecondaryOESIP** | char\[\] | 24 | 50 | \* |  |  | Note 3 |
| **PrimaryMDIP** | char\[\] | 24 | 74 |  |  |  | Not Used |
| **SecondaryIP** | char\[\] | 24 | 89 |  |  |  | Not Used |
| **SendingTime** | Long | 8 | 122 |  |  |  | Note 5 |
| **MsgSeqNum** | Int | 4 | 130 |  |  | 1500201 |  |
| **Key** | Int | 4 | 134 |  |  | 432451 |  |
| **LoginStatus** | short | 1 | 138 |  |  |  |  |
| **RejectReason** | short | 2 | 140 |  |  |  |  |
| **RiskMaster** | char | 1 | 142 |  |  |  |  |

#### Notes:

1. LogonType is a short enum, values: `Login 1`, `Logout 2`
   * If the value is not one of the values above, a logout message will be sent and the connection closed.
2. Value assigned by Black Ocean. If this is a logout, the TradingSessionID must be supplied.
3. IP address and port of the OES will be sent in the server response BOClientLogon message.
4. 2FA is disabled for the testing phase.
5. Sending times are in nanoseconds from the epoch, January 1, 1970.

### BOClientLogon/Logout -- Server Response

1. Upon Receipt of a client logon, the server will respond with a BOClientLogon message.
2. The status of the logon will be in the LoginStatus ﬁeld, a value of 1 indicates the logon was successful, a value of 2 indicates the login failed and the server will then close the connection. The reason for the unsuccessful login will be in the RejectReason ﬁeld. Possible values for the reject will be detailed in a section below.
3. The IP address and port of the OES \(Order Entry Server\) will be supplied in the message in order to allow the user to make the connection to the OES. Login to the OES will be covered in a subsequent section.
4. The server may initiate logout messages in the following circumstances:
   1. Matching Engine switching to secondary server \(if it is deemed operationally more eﬃcient in the event of a loss of the primary Matching Engine, the OES may switch to the secondary OES where the new primary Matching Engine is located\).
   2. Malformed messages which cannot be processed.
   3. Black Ocean initiated shutdown of the OES.
5. In the event the OES becomes unavailable, the user should reconnect to the secondary OES for which the IP Address and Port is supplied in the message.
6. In the event the client sends a logout message, the server will not respond with a logout message but will instead proceed to close the connection.

| Field Name | Data Type | Data Length | Buffer Offset | Required Field | Required Value | Example Value | Notes |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Data1** | char | 1 | 0 | X | H | H | Header |
| **Data2** | char | 1 | 1 |  |  |  | Header |
| **Data3** | short | 2 | 2 | X | 143 | 143 | Header |
| **LogonType** | short | 2 | 4 |  |  | 1 | Note 1 |
| **Account** | Int | 4 | 6 | \* |  | 25336 | Note 2 |
| **2FA** | char\[\] | 6 | 10 | X |  | 1F6A | Note 3 |
| **UserName** | char\[\] | 6 | 16 | X |  | BOU1 | Note 2 |
| **TradingSessionID** | Int | 4 | 22 | X |  | 505 | Note 2 |
| **PrimaryOESIP** | char\[\] | 24 | 26 | \* |  | 192.0.0.1:4300 5 | Note 4 |
| **SecondaryOESIP** | char\[\] | 24 | 50 | \* |  | 113.50.1.1:430 05 | Note 4 |
| **PrimaryMDIP** | char\[\] | 24 | 74 |  |  |  | Not Used |
| **SecondaryIP** | char\[\] | 24 | 98 |  |  |  | Not Used |
| **SendingTime** | Long | 8 | 122 |  |  |  | Note 5 |
| **MsgSeqNum** | Int | 4 | 130 | X |  | 1500201 |  |
| **Key** | Int | 4 | 134 |  |  | 432451 |  |
| **LoginStatus** | short | 1 | 138 |  |  |  | Not Used |
| **RejectReason** | short | 2 | 140 |  |  |  | Note 6 |
| **RiskMaster** | char | 1 | 142 |  |  |  | Note 7 |
|  |  | **Total Length** | **143** |  |  |  |  |

#### Notes:

1. LogonType is a short enum \(see Client logon above for possible values\)
2. TradingSessionID, UserName and Account number included in logon response but may or may not be included in logout messages.
3. Disabled for testing.
4. Not included in logout messages \(LogonType = 2\)
5. Sending times are in nanoseconds from the epoch, January 1, 1970.
6. If the login was rejected, the reject reason will be in the ﬁeld RejectReason, see section below for possible values.
7. Not currently used, for future expansion.

| Error Code | Value |
| :--- | :--- |
| **USER\_NOT\_FOUND** | 2 |
| **ACCOUNT\_NOT\_FOUND** | 3 |
| **INVALID\_KEY** | 4 |
| **ACCOUNT\_DISABLED** | 5 |
| **RISK\_ACCOUNT\_NOT\_FOUND** | 7 |
| **MES\_NOT\_AVAILABLE\_TRADING\_DISABLED** | 9 |
| **OES\_NOT\_AVAILABLE\_TRADING\_DISABLED** | 10 |
| **MDS\_NOT\_AVAILABLE\_TRADING\_DISABLED** | 11 |
| **MSG\_TYPE\_INVALID** | 12 |
| **NO\_RISK\_DATA** | 44 |
| **MSG\_SEQ\_NUM\_INVALID** | 52 |
| **USER\_ALREADY\_LGGGED\_IN** | 53 |
| **MALFORMED\_MSG** | 54 |
| **SENDING\_TIME\_INVALID** | 21 |

## Successful login to Order Entry Server \(OES\)

After a successful login to the OES the user should request the instrument data, risk information and outstanding orders.

### BOInstrumentRequest

The BOInsrumentRequest message should be sent to receive the parameters necessary to trade any instrument supported by Black Ocean. The OES will respond with a BOInstrument message.

| Field Name | Data Type | Data Length | Buffer Offset | Required Field | Required Value | Example Value | Notes |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Data1** | char | 1 | 0 | X | Y | Y | Header |
| **Data2** | char | 1 | 1 |  |  |  | Header |
| **Data3** | short | 2 | 2 | X | 62 | 62 | Header |
| **MessageType** | short | 2 | 4 |  |  |  | Not Used |
| **Reject Reason** | Int | 4 | 6 | \* |  |  | Note 7 |
| **Account** | Int | 4 | 8 | X |  | 100700 | Note 1 |
| **Request Type** | short | 2 | 12 | X |  | 2 | Note 2 |
| **Key** | Int | 4 | 14 | X |  |  | Note 2 |
| **SymbolName** | char\[\] | 24 | 18 |  |  |  | Note 3 |
| **SymbolType** | short | 2 | 42 |  |  |  | Note 4 |
| **SymbolEnum** | short | 2 | 44 |  |  |  | Note 5 |
| **TradingSessionID** | Int | 4 | 46 |  |  | 506 |  |
| **SendingTime** | Long | 8 | 50 | X |  |  | Note 6 |
| **MsgSeqNum** | Int | 4 | 58 | X |  | 1500201 |  |
|  |  | **Total Length** | **62** |  |  |  |  |

#### Notes:

Note 1: TradingSessionID, UserName and Account number supplied by Black Ocean to the user.

Note 2: RequestType is an enum with the following values:

| Enum Name | Enum Value |  |
| :--- | :--- | :--- |
| **ALL** | 1 | Request all instruments |
| **SYMBOL\_ENUM** | 2 | Individual instrument |

Note 3: Character string name representation an individual instrument if known, if not known the name will be provided in the BOInstrument response.

Note 4: SymbolType is a short integer enum with the following possible values: 

| Enum Name: SymbolType | Enum Value | Enum Data Type: short |
| :--- | :--- | :--- |
| **SPOT** | 1 |  |
| **FUTURES** | 2 |  |
| **DERIVATIVE** |  |  |

 Note 5: Symbol Enum is a short integer with the following possible values:

| Enum name: SymbolEnum | Enum value |
| :--- | :--- |
| **BTCUSD** | 1 |
| **USDUSDT** | 2 |
| **FLYUSDT** | 3 |
| **BTCUSDT** | 4 |

Note 6: Sending times are in nanoseconds from the epoch, January 1, 1970.

Note 7: If the request was rejected, the reject reason will be in the ﬁeld RejectReason, see section below for possible values.

### Possible BOInstrumentRequest Reject Codes

| Error Code | Value |
| :--- | :--- |
| **USER\_NOT\_FOUND** | 2 |
| **ACCOUNT\_NOT\_FOUND** | 3 |
| **INVALID\_KEY** | 4 |
| **MSG\_SEQ\_NUM\_INVALID** | 52 |
| **SENDING\_TIME\_INVALID** | 21 |

### BOInstrument - Server Response to BOInstrumentRequest 

In response to the BOInstrumentRequest, the server will send a BOInstrument message for each instrument.

* BOInstrument message

| Field Name | Data Type | Data Length | Buffer Offset | Required Field | Required Value | Example Value | Notes |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Data1** | char | 1 | 0 | X | Y | Y | Header |
| **Data2** | char | 1 | 1 |  |  |  | Header |
| **Data3** | short | 2 | 2 | X | 62 | 62 | Header |
| **MessageType** | short | 2 | 4 | \* | Y | Y | Note 6 |
| **Padding** | short | 2 | 6 | \* |  |  | Not Used |
| **ResponseType** | short | 2 | 8 | X |  | 2 | Note 1 |
| **SymbolEnum** | short | 2 | 10 | X |  | 1 | Note 2 |
| **SymbolName** | char\[\] | 24 | 12 | X |  | BTCUSD | Note 3 |
| **SymbolType** | short | 2 | 36 |  |  | 1 | Note 2 |
| **PriceIncrement** | double | 2 | 38 |  |  | .5 | Note 4 |
| **MinSize** | double | 2 | 46 |  |  | .00001 |  |
| **MaxSize** | double | 4 | 54 |  |  | 1000 |  |
| **SendingTime** | long | 8 | 62 | X |  |  | Note 5 |
| **MsgSeqNum** | Int | 4 | 70 | X |  | 1500201 |  |
|  |  | **Total Length** | **74** |  |  |  |  |

#### Notes:

Note 1: ResponseType is a short enum with the following possible values. If the ResponseType is set to SYMBOL\_ENUM, the SymbolEnum ﬁeld must be set to the value of the symbol enum being requested.

| ResponseType Enum | Enum Value \(short int\) |  |
| :--- | :--- | :--- |
| **SNAPSHOT** | 1 |  |
| **SNAPSHOT\_START** | 2 | Multi message responses |
| **SNAPSHOT\_CONTINUATION** | 3 | Continuation of response |
| **SNAPSHOT\_FINISH** | 4 | Last message of the response |
| **UPDATE** | 5 |  |
| **UNSOLICITED** | 6 |  |
| **POSS\_DUP** | 7 |  |

Note 2: Please see preceding section for possible values for the SymbolEnum and SymbolType.

Note 3: Character representation of the instrument name.

Note 4: PriceIncrement is the minimum price movement of the instrument.

Note 5: Sending times are in nanoseconds from the epoch, January 1, 1970.

Note 6: MessageType is not really required since a message of this type is deﬁned in the ﬁrst byte of the header.

### BORiskUpdateRequest -- Client Sending

BORiskUpdateRequest is sent retrieve the current risk parameter values.

| Field Name | Data Type | Data Length | Buffer Offset | Required Field | Required Value | Example Value | Notes |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Data1** | char | 1 | 0 | X | W | W | Header |
| **Data2** | char | 1 | 1 |  |  |  | Header |
| **Data3** | short | 2 | 2 | X | 34 | 34 | Header |
| **MessageType** | short | 2 | 4 | \* | W | W | Note 6 |
| **ResponseType** | short | 2 | 6 | X |  | 2 | Note 1 |
| **Account** | Int | 4 | 8 | X |  | 100700 | Note 2 |
| **TradingSessionID** | Int | 4 | 12 | X |  | 506 | Note 3 |
| **SymbolEnum** | short | 2 | 16 |  |  | 1 | Note 2 |
| **Key** | Int | 2 | 18 |  |  |  | Note 4 |
| **MsgSeqNum** | Int | 4 | 22 | X |  | 1005231 |  |
| **SendingTime** | uint64\_t | 4 | 26 | X |  |  |  |
|  |  | **Total Length** | **34** |  |  |  |  |

#### Notes:

Note 1: If the value of the RequestType in the BOInstrumentRequest message was set to SYMBOL\_ENUM, the value of this ﬁeld will be set to SNAPSHOT to signify it is the only response to the request. If the BOInstrument was set to ALL, then the value of this ﬁeld will be set corresponding to the current position of the response messages, SNAPSHOT\_START, SNAPSHOT\_CONTINUATION, SNAPSHOT\_FINISH.

Note 2: Please see preceding sections for possible values of SymbolEnum Note 3: Session ID returned in the BOClientLogon when logging into the OES Note 4: Disabled for testing.

### BORiskUserSymbol - Server Response to BORiskUpdate

In response to a BORiskUpdateRequest the server will send one or multiple BORiskSymbolUpdate messages depending on the ResponseType requested in the BORiskUpdateRequest.

| Field Name | Data Type | Data Length | Buffer Offset | Required Field | Required Value | Example Value | Notes |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Data1** | char | 1 | 0 | X | N | N | Header |
| **Data2** | char | 1 | 1 |  |  |  | Header |
| **Data3** | short | 2 | 2 | X | 34 | 34 | Header |
| **MessageType** | short | 2 | 4 | \* | N | N | Note 1 |
| **Padding** | short | 2 | 6 |  |  |  | Not Used |
| **UserName** | char\[\] | 6 | 8 | X |  | 2 |  |
| **Account** | Int | 4 | 14 | X |  | 100700 |  |
| **SymbolEnum** | short | 2 | 18 | X |  | 1 | Note 2 |
| **Leverage** | double | 8 | 20 |  |  |  | Note 3 |
| **LongPosition** | double | 8 | 28 | \* |  | 3.0 | Note 4 |
| **ShortPosition** | double | 8 | 36 | \* |  | 5.0 | Note 4 |
| **LongCash** | double | 8 | 44 | \* |  | 51000 | Note 4 |
| **ShortCash** | double | 8 | 52 | \* |  | 52000 | Note 4 |
| **SymbolDisabled** | char | 1 | 60 |  |  | 0 | Note 5 |
| **AccountEquity** | double | 8 | 61 |  |  |  | Not Used |
| **InstrumentEquity** | double | 8 | 69 |  |  |  | Not Used |
| **ExecutedLongCash** | double | 8 | 77 | \* |  | 150000 | Note 4 |
| **ExecutedLongPosition** | double | 8 | 85 | \* |  | 3 | Note 4 |
| **ExecutedShortCash** | double | 8 | 93 | \* |  | 102000 | Note 4 |
| **Executed Short Position** | double | 8 | 101 | \* |  | 2 | Note 4 |
| **BTCEquity** | double | 8 | 109 |  |  | 25 | Note 4 |
| **USDTEquity** | double | 8 | 117 |  |  | 500000 | Note 6 |
| **ETHEquity** | double | 8 | 125 |  |  | 1000 | Note 6 |
| **USDEquity** | double  | 8 | 133 |  |  | 1000000 | Note 6 |
| **FLYEquity** | double | 8 | 141 |  |  | 400000 | Note 6 |
| **OpenOrderRequest Limit** | Int | 4 | 149 |  |  | 1000 | Note 7 |
| **TradingSessionID** | Int | 4 | 153 |  |  | 506 |  |
| **MsgSeqNum** | Int | 4 | 157 |  |  | 42341 |  |
|  |  | **Total Length** | **161** |  |  |  |  |

#### Notes:

Note 1: MsgType not required in this type of message.

Note 2: Refer to previous sections for possible values.

Note 3: Leverage is not available in spot markets.

Note 4: LongPosition, ShortPosition, LongCash and ShortCash are the values of the current outstanding position sizes and cash amounts of those sizes. ExecutedLongCash, ExecutedLongPosition, ExecutedShortCash, ExecutedShortPosition are the values of previously executed positions.

Note 5: There may be circumstances where an account needs to be disabled, a value of 1 in this ﬁeld indicates the account trading has been disabled.

Note 6: Current balances of collateral held by the account, either through deposits or as the result of trading activity.

Note 7: Black Ocean determines the maximum number of orders requests \(OrdType = NEW or OrdType = CANCEL\_REPLACE\) which can be submitted without receiving a response. Example, if the value of the OpenOrderRequestLimit is set at 1000, if the user submits 1000 order requests, they must wait until they receive at least one response before submitting another order request. When the user receives one response they can then submit another request. Once an order has been submitted and acknowledged by the Matching Engine and returned to the user the open order number is decremented by 1 until such time as the user submits another order or the Matching Engine acknowledges more open orders from the user.

### BOOpenOrderRequest - Client Sending

The BOOpenOrderRequest message should be sent to receive the parameters necessary to trade any instrument supported by Black Ocean. The OES will respond with a BOInstrument message.

| Field Name | Data Type | Data Length | Buffer Offset | Required Field | Required Value | Example Value | Notes |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Data1** | char | 1 | 0 | X | E | E | Header |
| **Data2** | char | 1 | 1 |  |  |  | Header |
| **Data3** | short | 2 | 2 | X | 40 | 40 | Header |
| **MessageType** | short | 2 | 4 |  |  |  | Not Used |
| **Account** | Int | 4 | 6 | X |  | 100700 | Note 1 |
| **SymbolEnum** | short | 2 | 10 |  |  |  | Note 2 |
| **SymbolName** | char\[\] | 12 | 12 |  |  |  | Note 3 |
| **TradingSessionID** | Int | 4 | 24 |  |  | 506 |  |
| **SendingTime** | long | 8 | 28 | X |  |  | Note 4 |
| **MsgSeqNum** | Int | 4 | 36 | X |  | 1500201 |  |
|  |  | **Total Length** | **40** |  |  |  |  |

#### Notes:

Note 1: TradingSessionID, UserName and Account number supplied by Black Ocean to the user.

Note 2: SymbolType is a short integer enum with the following possible values:

| Enum Name: SymbolType | Enum Value | Enum Data Type: short |
| :--- | :--- | :--- |
| **SPOT** | 1 |  |
| **FUTURES** | 2 |  |
| **DERIVATIVE** | \*\*\*\* |  |

Note 3: Character string name representation an individual instrument if known, if not known the name will be provided in the BOInstrument response.

Note 4: Sending times are in nanoseconds from the epoch, January 1, 1970.

### OES Response to BOOpenOrderRequest

The server response to the BOOpenOrderRequest is each order on the book for the requested instrument with a BOTransaction message. See next section \(Application Messages\) for a description of the BOTransaction messages.

