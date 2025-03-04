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