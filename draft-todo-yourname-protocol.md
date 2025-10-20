---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "NTPv5 Timestamp Extension Field"
category: info

docname: draft-ietf-ntp-ntpv5-extension-fields-00
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: INT
workgroup: NTP
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Siyu Tang
    organization: Huawei Technologies
    email: siyutang@huawei.com
normative:

informative:

...

--- abstract
This document describes an alternative timestamp to the Monotonic Receive Timestamp Extension Field defined in NTP version 5 (NTPv5) when transferring frequency offset. The new extension field, named Monotonic RAW Receive Timestamp Extension Field uses a stable clock source that is not affected by NTP adjustment. It provides more accurate frequency-transfer offset between a remote server and local client, which further enhances the accuracy of time synchronization.


--- middle

# Introduction

NTP version 5 (NTPv5) [I-D.draft-ietf-ntp-ntpv5] introduces a Monotonic Receive Timestamp Extension Field to transfer frequency in addition to the time-transfer offset captured by the receive and transmit timestamps in the header. Separation of time and frequency transfer using different clocks shall enhance synchronization accuracy. 
It should be noted that when the system clock is slewed [RFC5905], the clock rate of the Monotonic Receive Timestamp Extension Field changes accordingly, i.e., clock rate diverges from the rate of the crystal. This introduces additional errors when performing frequency transfer, hence, negatively impact the accuracy of clock synchronization. 
This document proposes a stable clock source whose rate is not affected by NTP adjustment. The Monotonic RAW Receive Timestamp Extension Field is recommended to faithfully reflect the crystal rate, despite stepping or slewing a system clock. In case of link asymmetry, the Monotonic RAW Transmit Timestamp Extension Field is recommended in addition to the Monotonic RAW Receive Timestamp Extension Field. Measurements from the two extension fields can be used to identify link asymmetry and enhance time synchronization accuracy.  



# Conventions and Definitions

{::boilerplate bcp14-tagged}
Monotonic Raw Original Timestamps (raw_org): Time of the monotonic raw clock at the client when the request departed for the server, in NTP timestamp format.

Monotonic Raw Receive Timestamps (raw_rec): Time of the monotonic raw clock at the server when the request arrived from the client, in NTP timestamp format. 

Monotonic Raw Transmit Timestamps (raw_xmt): Time of the monotonic raw clock at the server when the response left for the client, in NTP timestamp format.

Monotonic Raw Destination Timestamps (raw_dst): Time of the monotonic raw clock at the client when the reply arrived from the server, in NTP timestamp format.

# Monotonic Receive Timestamp Extension Field in NTPv5

The Monotonic Receive Timestamp Extension Field defined in NTPv5 uses a different clock to transfer frequency between client and server. In NTP version 4 (NTPv4) [RFC5905], the clock discipline function defines two methods to adjust system clock, i.e., step and slew. In the step mode, the clock is stepped to the correct offset. In the slew mode, the clock rate is adjusted to achieve the desired offset during a certain amount of time. The clock rate used to measure the Monotonic Receive Timestamp remains unchanged if the system clock is stepped, but is subject to changes if the clock is slewed, see Figure 1. 

+-----------------------------------------------------+
| ClockID              |    STEP   |    SLEW          |
| in Linux             |           |                  |
+-----------------------------------------------------|
| CLOCk_REALTIME       |    YES    |     YES          |
|                                                     |
| CLOCK_MONOTONIC       |    NO     |     YES          |
|                                                     |
| CLOCK_MONOTONIC_RAW  |    NO     |     NO           |
+-----------------------------------------------------|
Figure 1 Impact of NTP clock adjustment on clock rate/frequency. 

In NTPv5 Use Cases and Requirements [draft-ietf-ntp-ntpv5-requirements-04], it is recommended to adopt a linear and monotonic timescale when communicating time between a number of computers. Stepping a clock may cause the system time to jump backward, making the timescale non-monotonic. When the system clock is slewed, the rate of the monotonic clock source moves at the same speed as the system clock. The frequency-transfer offset can no longer reflect the rate of the crystal, thus, introducing errors in frequency transfer. In a multi-hop scenario, this effect can be amplified over a number of hops. In some scenarios, it can increase time errors when synchronizing time, sometimes, result in a system that fails to converge, see Section 5. 

# Monotonic RAW Timestamp Extension Fields
In the Linux system, CLOCK_MONOTONIC_RAW is a clock source that is not subject to NTP adjustment, despite stepping or slewing a clock, see Figure 1. It provides a stable source to calculate the frequency-transfer offset and reduces the error that has been introduced using the Monotonic Receive Timestamp extension field. 

## Monotonic Raw Receive Timestamp Extension Field
An NTPv5 message contains multiple optional extension fields. A Monotonic Raw Receive Timestamp Extension Field is recommended in addition to the Monotonic Receive Timestamp extension field. It is also recommended to derive the frequency-transfer offset from the Monotonic Raw Receive Timestamp Extension Field if CLOCK_MONOTONIC_RAW is available. 
The Monotonic Raw Receive Timestamp Extension Field has the same format of the Monotonic Receive Timestamp Extension Field. It complies to the constant length of 16 octets as defined in NTPv5. The counter and timestamp are set in response. This extension field enhances the accuracy of frequency-transfer function and further reduce synchronization time error. 

## Monotonic Raw Transmit Timestamp Extension Field 
The Monotonic Raw Receive Timestamp Extension Field appears to be insufficient when dealing with asymmetric packet delay variation (PDV) on the forward and backward paths. The same issue exists with the Monotonic Receive Timestamp Extension Field.
The Monotonic Raw Transmit Timestamp Extension Field is included to identify link asymmetry (i.e., different PDV on forward and backward paths) and reduce related errors. The Monotonic Raw Transmit Timestamp Extension Field has the same format of the Monotonic Receive Timestamp Extension Field. 

## Frequency to The Root Server Extension Field
In NTPv5, the frequency-transfer offset is computed as the offset of a client relative to its immediate preceding server. A client is able to synchronize with the primary server (i.e., the root server) only if its preceding server has synchronized its frequency with the primary server. The Frequency To The Root Server Extension Field is an optional field that can be used to expedite the convergence speed when synchronizing time. 
The Frequency To The Root Server Extension Field contains the frequency-transfer offset of a client relative to the Realtime clock of the primary server. This extension field has a fixed length of 12 octets. The 1-bit sign bit is a binary number indicates if the frequency of a client is faster (1) or slower (0) relative to the primary server. The absolute frequency-transfer offset relative to the primary server is carried by the remaining 31-bit.

0                     1                     2                     3                  
01234567890123456789012345678901 
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Type=[[TBD]] (draft 0xF508)     |         Length                 |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|s|               Frequency To The Root Server (31)                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
Figure 2 Format of a Frequency To The Root Server Extension Field.

Assume a multi-hop scenario with n stratum levels. The primary server determines the frequency offset between the Realtime clock and the Monotonic Raw clock, and include this value into the Frequency To The Root Server Extension Field. The frequency-transfer offset of a server at stratum level i (2<i<=n) relative to its immediate preceding server is determined using the Monotonic Raw Receive Timestamp Extension Field (or with the Monotonic Receive Timestamp Extension Field if the clock is stepped). When receiving an NTP message, a server at stratum level i (2<i<=n) reads the Frequency To The Root Server Extension Field, and adds the frequency-transfer offset that it computed locally to the existing value. This way, the frequency-transfer offset of a server relative to the primary server is captured and passed down to the succeeding nodes.

For simplicity, assume that all nodes (other than the primary server) have successfully learnt their frequency-transfer offset relative to the preceding server at time t_k. Without the Frequency To The Root Server Extension Field, synchronization of a server to the primary server happens sequentially in time, i.e., a server with stratum level i needs to wait for its immediate preceding server (with stratum level i-1) to complete the operation of setting frequency-transfer offset. Note the operation of setting frequency offset is decoupled with NTP message exchange. This can take longer for the server at the last stratum level to synchronize with the primary server. With the assist of the Frequency To The Root Server Extension Field, the frequency-transfer offset of a server relative to the primary server can be immediately passed down in an NTP message, allowing the succeeding nodes to synchronize their frequency relative to the primary server once receiving that message. 



# Security Considerations

As this document is intended to create discussion and consensus, it introduces no security considerations of its own.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
