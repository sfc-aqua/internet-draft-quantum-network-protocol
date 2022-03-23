---
title: The Quantum Network Protocol
abbrev: QNP
docname: draft-van-meter-qnp-00
date: 2022-03-22
category: info

ipr: trust200902
area: General
workgroup: AQUA
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: R. Van Meter
    name: Rodney Van Meter
    organization: Keio University
    email: rdv@sfc.wide.ad.jp

normative:
  RFC2119:
  RFC3986:
  RFC4086:
  RFC4648:

informative:
  RFC5389:
  I-D.ietf-behave-turn:
  STUNT:
    target: http://deusty.blogspot.com/2007/09/stunt-out-of-band-channels.html
    title: STUNT & out-of-band channels
    author:
      name: Robbie Hanson
      ins: R. Hanson
    date: 2007-09-17
  I-D.meyer-xmpp-e2e-encryption:
  I-D.ietf-xmpp-3920bis:



--- abstract

The Quantum Network Protocol.

--- middle

Introduction        {#problems}
============


Basic Protocol Operation   {#ops}
========================


~~~~~~~~~~


        STuPiD   ```````````````````````````````,
        Script   <----------------------------. ,
                                              | ,
          ^ ,                                 | ,
          | ,                                 | ,
    (1)   | ,                                 | ,  (3)
    POST  | ,                                 | ,  GET
          | ,                                 | ,
          | v                                 | v

        Peer A   ----------------------->   Peer B
                           (2)
                       out-of-band
                       Notification
~~~~~~~~~~
{: #figops title="STuPiD Protocol Operation"}


Data Types
===================
  - node address
  - entangled state identifier

Condition Clause
===================

Action Clause
===================

Rule
===================

Stages
===================

RuleSet
===================


Protocol Definition
===================

Terminology          {#Terminology}
-----------
In this document, the key words "MUST", "MUST NOT", "REQUIRED",
"SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" are to be interpreted as described in BCP 14, RFC 2119
{{RFC2119}} and indicate requirement levels for compliant STuPiD
implementations.




Security Considerations
=======================


The security implications are extensive.

--- back


Examples  {#xmp}
========

This appendix provides some examples of the STuPiD protocol operation.

~~~~~~~~~~
   Request:

      GET /stupid.php HTTP/1.0
      User-Agent: Example/1.11.4
      Accept: */*
      Host: example.org
      Connection: Keep-Alive

   Response:

      HTTP/1.1 200 OK
      Date: Sun, 05 Jul 2009 00:30:37 GMT
      Server: Apache/2.2
      Cache-Control: no-cache, must-revalidate
      Expires: Sat, 26 Jul 1997 05:00:00 GMT
      Vary: Accept-Encoding
      Content-Length: 17
      Keep-Alive: timeout=1, max=400
      Connection: Keep-Alive
      Content-Type: application/octet-stream

      192.0.2.239:36654
~~~~~~~~~~
{: #figxmpdisco title="Discovering External IP Address and Port"}

~~~~~~~~~~
   Request:

      POST /stupid.php?chid=i781hf64-0 HTTP/1.0
      User-Agent: Example/1.11.4
      Accept: */*
      Host: example.org
      Connection: Keep-Alive
      Content-Type: application/octet-stream
      Content-Length: 11

      Hello World

   Response:

      HTTP/1.1 200 OK
      Date: Sun, 05 Jul 2009 00:20:34 GMT
      Server: Apache/2.2
      Cache-Control: no-cache, must-revalidate
      Expires: Sat, 26 Jul 1997 05:00:00 GMT
      Vary: Accept-Encoding
      Content-Length: 0
      Keep-Alive: timeout=1, max=400
      Connection: Keep-Alive
      Content-Type: application/octet-stream
~~~~~~~~~~
{: #figxmpstore title="Storing Data"}

~~~~~~~~~~
   Request:

      GET /stupid.php?chid=i781hf64-0 HTTP/1.0
      User-Agent: Example/1.11.4
      Accept: */*
      Host: example.org
      Connection: Keep-Alive

   Response:

      HTTP/1.1 200 OK
      Date: Sun, 05 Jul 2009 00:21:29 GMT
      Server: Apache/2.2
      Cache-Control: no-cache, must-revalidate
      Expires: Sat, 26 Jul 1997 05:00:00 GMT
      Vary: Accept-Encoding
      Content-Length: 11
      Keep-Alive: timeout=1, max=400
      Connection: Keep-Alive
      Content-Type: application/octet-stream

      Hello World
~~~~~~~~~~
{: #figxmpretr title="Retrieving Data"}


Sample Implementation     {#impl}
=====================

~~~~~~~~~~
<?php
header("Cache-Control: no-cache, must-revalidate");
header("Expires: Sat, 26 Jul 1997 05:00:00 GMT");
header("Content-Type: application/octet-stream");

mysql_connect(localhost, "username", "password");
mysql_select_db("stupid");

$chid = mysql_real_escape_string($_GET["chid"]);

if ($_SERVER["REQUEST_METHOD"] == "GET") {
   if (empty($chid)) {
      echo $_SERVER["REMOTE_ADDR"] . ":" . $_SERVER["REMOTE_PORT"];
   } elseif ($result = mysql_query("SELECT `data` FROM `Data` " .
                         "WHERE `chid` = '$chid'")) {
      if ($row = mysql_fetch_array($result, MYSQL_ASSOC)) {
         echo base64_decode($row["data"]);
      } else {
         header("HTTP/1.0 404 Not Found");
      }
      mysql_free_result($result);
   } else {
      header("HTTP/1.0 404 Not Found");
   }
} elseif ($_SERVER["REQUEST_METHOD"] == "POST") {
   if (empty($chid)) {
      header("HTTP/1.0 404 Not Found");
   } else {
      mysql_query("DELETE FROM `Data` " .
                  "WHERE `timestamp` < DATE_SUB(NOW(), INTERVAL 5 MINUTE)");
      $data = base64_encode(file_get_contents("php://input"));
      if (!mysql_query("INSERT INTO `Data` (`chid`, `data`) " .
                       "VALUES ('$chid', '$data')")) {
         header("HTTP/1.0 403 Bad Request");
      }
   }
} else {
   header("HTTP/1.0 405 Method Not Allowed");
   header("Allow: GET, HEAD, POST");
}
mysql_close();
?>
~~~~~~~~~~
{: #figimpl title="STuPiD Sample Implementation"}


Using XMPP as Out-Of-Band Channel  {#xmpp}
=================================

XMPP {{I-D.ietf-xmpp-3920bis}} is a good choice for
an out-of-band channel.

The notification protocol is closely modeled after XMPP's
In-Band Bytestreams (IBB, see
http://xmpp.org/extensions/xep-0047.html). Just replace the
namespace and insert the STuPiD Retrieval URI instead of the
actual Base64 encoded data, see {{figxmpnots}}.
(Note that the current proposal redundantly sends a sid and a
seq as well as the chid composed of these two; it may be
possible to optimize this, possibly sending the constant prefix
of the URI once at bytestream creation time.)

Notifications MUST be processed in the order they are
received. If an out-of-sequence notification is received for a
particular session (determined by checking the 'seq' attribute),
then this indicates that a notification has been lost. The
recipient MUST NOT process such an out-of-sequence notification,
nor any that follow it within the same session; instead, the
recipient MUST consider the session invalid.  (Adapted from
http://xmpp.org/extensions/xep-0047.html#send)

Of course, other methods can be used for setup and teardown, such as Jingle
(see http://xmpp.org/extensions/xep-0261.html).

~~~~~~~~~~
      <iq from='romeo@montague.net/orchard'
          id='jn3h8g65'
          to='juliet@capulet.com/balcony'
          type='set'>
        <open xmlns='urn:xmpp:tmp:stupid'
              block-size='65536'
              sid='i781hf64'
              stanza='iq'/>
      </iq>
~~~~~~~~~~
{: #figxmpcri title="Creating a Bytestream: Initiator requests session"}


~~~~~~~~~~
      <iq from='juliet@capulet.com/balcony'
          id='jn3h8g65'
          to='romeo@montague.net/orchard'
          type='result'/>
~~~~~~~~~~
{: #figxmpcrr title="Creating a Bytestream: Responder accepts session"}



~~~~~~~~~~
      <iq from='romeo@montague.net/orchard'
          id='kr91n475'
          to='juliet@capulet.com/balcony'
          type='set'>
        <data xmlns='urn:xmpp:tmp:stupid'
              seq='0'
              sid='i781hf64'
              url='http://example.org/stupid.php?chid=i781hf64-0'/>
      </iq>
~~~~~~~~~~
{: #figxmpnots title="Sending Notifications: Notification in an IQ stanza"}

~~~~~~~~~~
      <iq from='juliet@capulet.com/balcony'
          id='kr91n475'
          to='romeo@montague.net/orchard'
          type='result'/>
~~~~~~~~~~
{: #figxmpnota title="Sending Notifications: Acknowledging notification using IQ"}

~~~~~~~~~~
      <iq from='romeo@montague.net/orchard'
          id='us71g45j'
          to='juliet@capulet.com/balcony'
          type='set'>
        <close xmlns='urn:xmpp:tmp:stupid'
               sid='i781hf64'/>
      </iq>
~~~~~~~~~~
{: #figxmpclor title="Closing the Bytestream: Request"}

~~~~~~~~~~
      <iq from='juliet@capulet.com/balcony'
          id='us71g45j'
          to='romeo@montague.net/orchard'
          type='result'/>
~~~~~~~~~~
{: #figxmpclos title="Closing the Bytestream: Success response"}
