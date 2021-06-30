# Session Management

## BOClientLogon -- Client Sending

DESCRIPTION

| Field Name | Data Type | Required | Notes |
| :--- | :--- | :---: | :---: |
| **Data1** | char | X |  |
| **LogonType** | short | ~~X~~ |  |
| **Account** | Int | X |  |
| **UserName** | char\[\] | X |  |
| **TradingSessionID** | Int | X |  |
| **SendingTime** | Long | X |  |
| **MsgSeqNum** | Int | X |  |
| **Key** | Int | X |  |

```text
{"msg1":"H","LogonType":1,"Account":100700,"UserName":""BOU7"","TradingSessionID":506,"SendingTime":1624785162815971526,"MsgSeqID":110434,"Key":123456,"LoginStatus":1,"RejectReason":50,"RiskMaster":"N"}
```

#### 



### BOClientLogon/Logout -- Server Response

DESCRIPTION

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

