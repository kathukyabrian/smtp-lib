# SMTP Docs 
> objective - transfer mail reliably and efficiently

- independent of the transmission subsystem and requires only a reliable ordered data stream channel

## The SMTP Model
### Basic Structure
- when an SMTP client has a message to transmit, it establishes a 2-way transmission channel to the SMTP server
- a fully capable SMTP client is expected to support
    - queueing
    - retrying
    - alternate address functions
- an SMTP server may be either:
    - ultimate destination
    - intermediate relay
- the client generates SMTP commands and sends them to the server, the server sends an SMTP reply back to the client.
- message transfer can occur in a single connection between the original SMTP-sender and the final recipient or through a series of hops through intermediary systems.
- in either case, once the server has issued a success response at the end of the mail data, a formal handoff of responsibility for the message occurs
- the protocol requires that the server MUST accept responsibility for either
    - delivering the message
    - properly reporting the failure to do so.
- once the transmission channel is established, the SMTP client initiates a mail transaction
- such a transaction consists of a series of commands to specify the originator and destination of the mail and transmission of the message content
- the server responds to each command with a reply, the replies may indicate that:
    - command was accepted
    - additional commands are expected
    - temporary or permanent error condition exists
- once a given mail message has been transmitted, the client may either:
    - request that the connection be shut down
    - initiate other mail transactions
    - use connection for ancillary services such as verification of email addresses or retrieval of mailing list subscriber addresses.

### SMTP Terminology
#### Mail Objects
- SMTP transports mail objects
- consists of:
    - envelope
    - content
- envelope is sent as a series of SMTP protocol units. consists of:
    - originator address
    - one or more recipient addresses
    - optional protocol extension material
- content is sent in the SMTP DATA protocol unit and has 2 parts:
    - header
    - body
- header sections consists of a collection of header fields each consisting of a header name, a colon and data
- body is defined according to MIME. the content is textual expressed using the US-ASCII

#### Senders and Receivers
- the 2 parties participating in a SMTP transaction

#### Mail Agents and Message Stores
- Mail Transfer Agents(MTA) - SMTP servers and client - they provide a mail transport service

- Mail User Agents(MUA) - sources and targets of the mail 

- case:
    - MUA - MTA - MTA - (message store) - MUA

#### Host
- computer system attached to the internet
- known by name, not addresses.

#### Domain Names
- entire, fully qualified domain name(FQDNs)

#### Buffer and State Table
- SMTP sessions are stateful - both parties maintain a common view of the current state
- model - virtual 'buffer' and a 'state table' on the server that may be used by client to:
    - clear the buffer
    - reset the state table


#### Commands and Replies
- commands are transmitted from the sender to the receiver via the transmission line
- a reply is an ACK(negative or positive) sent in lines from receiver to sender in response to a command.
- general form of a reply is a numeric completion code usually followed by a text string

#### Lines
- contain zero or more characters terminated by CR(hex 0D) followed immediately by LF(hex 0A) - CRLF
- limits may be imposed on line lengths by servers
- CR or RF should not appear alone in mail bodies

#### Message Content and Mail Data
- describes the material transmitted after the DATA command is accepted and before the end of data indication is transmitted
- includes message header section and the possibly structured message body

#### Originator, Delivery, Relay and Gateway systems
- originating system(SMTP originator) introduces mail into the internet(transport service environment)
- delivery system is one that receives mail from a transport service environment and passes it to a mail user agent or deposits it into a message store that a mail user agent is expected to subsequently access
- relay system receives mail from an SMTP client and transmits it, without modification to the message data other than adding trace information, to another SMTP server for further relaying or delivery
- gateway system receives mail from a client system in one transport environment and transmits it to a server system in another transport environment

#### Mailbox and Address
- address is a character string that identifies a user to whom mail will be sent or a location into which mail will be deposited.
- mailbox refers to that depository.
- address consists of 
    - user 
    - domain

### General Syntax Principles and Transaction Model
- commands begin with a command verb
- replies begin with a 3 digit numeric code
- in some commands and replies, arguments are required following the verb or reply code
- local part of mail box must be treated as case sensitive
- mailbox domains are not case sensitive
- argument clause consists of a variable-length character string ending with CRLF. receiver will not take action until this sequence is received.

## The SMTP Procedures: Overview
### Session Initiation
- a session is initiated when a client opens a connection to a server and the server responds with an opening message
- SMTP server implementations MAY include identification of their software and version information in the connection greeting reply after the __220__ code
- implementations MAY make provision for SMTP servers to disable the software and version announcement where it causes security concern
- SMTP allows a server to formally reject a mail session while still allowing the initial connection
  - a __554__ response may be given in the initial connection opening message instead of the 220
  - a server taking this approach MUST still wait for the client to send a QUIT before closing the connection and SHOULD respond to any intervening commands with __503 bad sequence of commands__

### Client Initiation
- once the server has sent the greeting, welcoming message and the client has received it, the client sends an __EHLO__ command to the server indicating the client's identity
- EHLO also indicates that the client is able to process service extensions and request that the server provides a list of the extensions it supports
- older SMTP systems that are unable to support service extensions and contemporary clients that do not require service extensions may use the __HELO__ command instead
- for a particular connection attempt, if the server returns a command not recognized response to EHLO, the client should be able to fallback to send HELO
- in the EHLO command, the host sending the command identifies itself; can be interpreted as saying *Hello I am <domain> and (in the case of EHLO) I support service extension requests*

### Mail Transactions
- 3 steps:
  -  __MAIL__ command - gives sender identification
  - a series of 1 or more __RCPT__ commands - gives receiver information
  - __DATA__ command - initiates transfer of the mail data, terminated by *end of mail* indicator

#### MAIL Command
> ```MAIL FROM:<reverse-path> [ SP <mail-parameters>] <CRLF>```
- this command tells the SMTP receiver that a new mail transaction is starting and that it resets all its state tables and buffers
- the __reverse-path__ contains the source mailbox in between < and >
- if accepted, the SMTP server responds with a __250 OK__ reply
- if the mailbox specification is not acceptable the server MUST return a reply indicating whether the failure is permanent or temporary
- there are certain circumstances in which the acceptability of the reverse path may not be determined until one or more forward paths can be examined
- in such a case, the server MAY accept the reverse path with a 250 REPLY and then report the problems after the forward paths are received and examined - failures produce __550__ or __553__ replies
- the optional mail parameters are associated with negotiated SMTP service extensions

#### RCPT Command
> ```RCPT TO:<forward-path> [ SP <rcpt-parameters>] <CRLF>```
- the first or only argument to this command - forward-path, is a mailbox and domain always sorrounded by < and >. it identifies a recipient
- if accepted, the server returns a __250 OK REPLY__ and stores the forward path
- if the recipient is known not to be a deliverable address, the SMTP server returns a __550 REPLY__ typically with a string such as "no such user" and the mailbox name
- the <forward-path> can contain more than just a mailbox
- servers must be prepared to encounter a list of source routes in the forward path
- if a RCPT command appears without a previous MAIL command the server must return a __503 Bad Sequence of Commands REPLY__
- the optional <rcpt-parameters> are associated with negotiated SMTP service extensions


- spaces are NOT permitted on either side of the colon following FROM in the mail command and TO in RCPT command

#### DATA command
> ```DATA <CRLF>```
- if accepted the server returns a __354 Intermediate REPLY__ and considers all succeeding lines up to but not including the end of the mail data indicator to be the message text
- when the end of the message is successfully received and stored, the SMTP receiver sends a 250 OK reply
- since the mail data is sent on the transmission line, the end of the mail data must be indicated so that the command and reply dialog can be resumed.
- SMTP indicates the end of the mail data by sending a line containing only a "."
- end of mail indicator also confirms the mail transaction and tells the server to now process the stored recipients and mail data
- if accepted the server returns a __250 OK REPLY__
- DATA command can fail at only 2 points:
  - if there was no MAIL or no RCPT command
  - all such commands were rejected


### Forwarding for address correction or updating