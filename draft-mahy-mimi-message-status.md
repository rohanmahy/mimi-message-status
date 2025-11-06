---
title: "A Message Status format for the More Instant Messaging Interoperability (MIMI) content format"
abbrev: "MIMI Message Status"
category: info

docname: draft-mahy-mimi-message-status-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "More Instant Messaging Interoperability"
keyword:
 - mimi content
 - status
 - read receipts
 - message delivery
venue:
  group: "More Instant Messaging Interoperability"
  type: "Working Group"
  mail: "mimi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mimi/"
  github: "rohanmahy/mimi-message-status"
  latest: "https://rohanmahy.github.io/mimi-message-status/draft-mahy-mimi-message-status.html"

author:
 -
    fullname: Rohan Mahy
    organization:
    email: rohan.mahy@gmail.com

normative:

informative:


--- abstract

The More Instant Messaging Interoperability (MIMI) content format describes
a message format for instant messaging. This specification defines a concise,
efficient format for communicating status of messages sent using MIMI content.

--- middle

# Introduction


This document describes the semantics of a status report of MIMI content format {{!I-D.ietf-mimi-content}} messages.
Because some messaging systems deliver messages in batches and allow a user to mark several messages read at a time, the report format allows a single report to convey the read/delivered status of multiple messages (by message ID) within the same MIMI room at a time.
This specification defines a concise, efficient format for communicating status of messages sent using MIMI content.
It could also represent messages sent using other messaging formats that have  similar per-message unique message IDs and security characteristics.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the term "room" as defined in {{!I-D.ietf-mimi-arch}}.

# Delivery Reporting and Read Receipts {#reporting}

In instant messaging systems, read receipts typically generate a distinct
indicator for each message. In some systems, the number of users in a group
who have read the message is subtly displayed and the list of users who
read the message is available on further inspection.

Of course, Internet mail has support for read receipts as well, but
the existing message disposition notification mechanism defined for email
in {{?RFC8098}} is completely inappropriate in this context:

* notifications can be sent by intermediaries
* only one notification can be sent about a single message per recipient
* a human-readable version of the notification is expected
* each notification can refer to only one message
* it is extremely verbose

Instead, we would like to be able to include status changes about multiple
messages in each report, the ability to mark a message delivered, then read, then unread, then expired
for example.

The format, like the MIMI content format, uses Common Binary Object Representation (CBOR) {{!RFC8949}} encoding.
It has the media type `application/mimi-message-status`.
It is sent by individual members of a chat room and can refer to multiple messages sent in the same room in a single message.
The format contains a list of message ID / status pairs.
As the status at the recipient changes, the status can be updated in subsequent notification.

The status of each message can be one of the following values:

- 0 (unread) indicates that the message was not yet read by the sender of the
report.
- 1 (delivered) indicates that a messaging client of the sender of the report
received the message.
- 2 (read) indicates that the sender of the report read the message.
- 3 (expired) indicates that the message expired and is not available for
reading. In the case of absolute expiration, it does not indicate if the message
was read before its expiry.
- 4 (deleted) indicates that the message was deleted, either by the local
client, or by another member of the room with the power to retract messages.
- 5 (hidden) indicates that the message was hidden by the local
client (for example archived).
- 6 (error) indicates that the sender client is aware of the message ID, but
that there was an unspecified error with the reception of the message.

Not every state is relevant for every type of message, and it does not make sense to transition from any one state to any other state.
For example, a transition from `deleted` to `delivered` does not make sense.
The implementer of this format needs to decide which state transitions are meaningful given their implementation and its available policy options.

Depending on the policy of the room and a potential sender of delivery reports,
sending delivery receipts and/or read receipt messages might be required,
optional, or forbidden.
Clients might also have policies about specific status values that are shared
and others that are not.
Some status values might only be shared among the reporting user's own clients, for example.

# Formal Data Definition

Below is the Concise Data Definition Language (CDDL) {{!RFC8610}} definition for the message status format.

~~~ cddl
{::include status.cddl}
~~~
{: title="CDDL for MIMI message status format"}


# Message Status Format Example

The following example message assumes the sender user handle URL is `mimi://example.com/u/bob-jones`.

It uses the example message names and message IDs from Section 5 of the MIMI content {{!I-D.ietf-mimi-content}}.


~~~ cbor-diag
{::include status.edn}
~~~
{: title="Example message report"}

A CBOR pretty printed hexadecimal version is shown below:

~~~
84                                      # array(4)
   82                                   # array(2)
      58 20                             # bytes(32)
         017ce54837404c3696e0c747b985cb17
         2716d0ed0a3d249ca63ace7d82a096f4
      02                                # unsigned(2)
   82                                   # array(2)
      58 20                             # bytes(32)
         015354973c2b65ca937bf1e035ae53a5
         ab80e947afa43d46920d4202e5cc0b27
      02                                # unsigned(2)
   82                                   # array(2)
      58 20                             # bytes(32)
         018d825adf9f6be00dcafc5704c4102f
         5022e74219d0b603e4ba7622654042af
      00                                # unsigned(0)
   82                                   # array(2)
      58 20                             # bytes(32)
         01e59db8173939facc2c8a4a0f0ae8d0
         c7a11a81239626630c9464a8d6717a03
      03                                # unsigned(3)
~~~


# Security Considerations

Delivery and Read Receipts can provide useful information inside a group,
or they can reveal sensitive private information. In many IM systems
there are per-group policies for read receipts (and/or delivery notifications):

* they are required
* they are permitted, but optional
* they are forbidden

In the first case, everyone in the group would have to claim to support
read receipts to be in the group and agree to the policy of sending them
whenever a message was read. A user who did not wish to send read receipts could
review the policy (automatically or manually) and choose not to join
the group. Of course, requiring read receipts is a cooperative effort
just like using self-deleting messages. A malicious client could obviously
read a message and not send a read receipt, or send a read receipt for a
message that was never rendered. However, cooperating clients have a way to
agree that they will send read receipts when a message is read in a specific
group.

In the second case, sending a read receipt would be at the discretion
of each receiver of the message (via local preferences).


# IANA Considerations

RFC EDITOR: Please replace XXXX throughout with the RFC number assigned to this document.

## MIME subtype registration of application/mimi-message-status

This document proposes registration of a media subtype with IANA.

~~~~~~~
Type name: application

Subtype name: mimi-message-status

Required parameters: none

Optional parameters: none

Encoding considerations:
   This message type should be encoded as binary data

Security considerations:
   See Section 6 of RFC XXXX

Interoperability considerations:
   See Section 3 of RFC XXXX

Published specification: RFC XXXX

Applications that use this media type:
   Instant Messaging Applications

Fragment identifier considerations: N/A

Additional information:

   Deprecated alias names for this type: N/A
   Magic number(s): N/A
   File extension(s): N/A
   Macintosh file type code(s): N/A

Person & email address to contact for further information:
   IETF MIMI Working Group mimi@ietf.org
~~~~~~~

--- back
