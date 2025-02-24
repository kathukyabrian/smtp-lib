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