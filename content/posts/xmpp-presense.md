---
title: "XMPP Presense"
date: 2024-06-24T08:04:00+03:00
draft: true
tags: ['xmpp']
---


Presense is the technology allowing you choose what informations of your network
availability you want to share. This is happening by subscription "handshake"

To request someone's presence you send him a subscruption request:

```xml
<presence from="george@im.company.org"
        to="mary@im.company.org"
        type="subscribe" />
```
The recepient can approve it or denied by sending `presence` stanza of type 
`subscribed` or `unsubscribed`:

```xml
<presence from="mary@im.company.org"
        to="george@im.company.org"
        type="subscribed" />
```

This is bidirectional process. So the recepient also send you a subscription
request and you also can subscrube or unsubscrbe to it.

```xml
<presence from="mary@im.company.org"
        to="george@im.company.org"
        type="subscribe" />

<presence from="george@im.company.org"
        to="mary@im.company.org"
        type="subscribed" />
```

After success handshake you will be automaticaly notified when someone's network
awailability changed with `presence` stanza without `type` attribute.

```xml
<presence from="mary@im.company.org"
        to="george@im.company.org">
    <show>xa</show>
    <status>On the meeting...</status>
</presence>
```

## How Presence Is Propagated

1) open xml stream
2) send initial presence stanza:

```xml
<presence />
```

3) Server check roster ( contact list ) and sends a presence notification to 
each person who subscribed to you with full JID with resource portion:

```xml
<presence from="george@im.company.org/office"
        to="patric@im.company.org"/>

<presence from="george@im.company.org/office"
        to="mary@im.company.org"/>
```

4) Now, everyone of your contact list ( who subscribe to you presence ) know 
about your availability. But you do not know about THEIR availability. To 
achieve that server sends presence of type `probe`:

```xml
<presence from="george@im.company.org/office"
        to="patric@im.company.org"
        type="probe" />

<presence from="george@im.company.org/office"
        to="mary@im.company.org"
        type="probe" />
```

5) Once contacts receive probes they check permissions according to their 
records. If your contact is offline the presence stanza will be of type
`unavailable` often with information when it was online last time. Like this:

```xml
<!-- contact is offline -->
<presence from="george@im.company.org/office"
        to="patric@im.company.org"
        type="unavailable">
    <delay xmlns="urn:xmpp:delay" stamps="2024-06-24T08:04:00+03:00" />
</presence>

<!-- contact is online -->
<presence from="mary@im.company.org/accounting"
        to="george@im.company.org/office" />
```

( basicaly `delay` element is the time when presence was last time changed - 
when presence was chanded to "unavailable" )

If contact has more then one connected resource you will receive more than one 
presence stanza from this contact.

## Status

Precense stanzas can contain more information than just `un/available`. As you 
was see before there also `<show/>` and `<status/>` elements.

`Show` element is limited to four predefined values:
- chat - you are available for communication
- away - you AFK. This is often triggered automaticaly after some timeout without
activity.
- xa - "eXtended Away`; more extended version of "away".
- dnd - "Do ot disturb"

`Status` element is for short text string describing your `show` status.

## Priority

Presence stanza can include one more element `<priority />`. It used for prioritize
availability statuses between connected resources:

```xml

<presence from="george@im.company.org/office">
    <priority>7</priority>
</presence>

<presence from="george@im.company.org/store">
    <priority>-1</priority>
</presence>
```

Value can be in range between -127 and +128 numbers. Higher priority resource
will more likely receive message from bare JID ( JID without resource portion ).

## Direct Presence

All previously examples was broadcasted messages. For example if you need to 
chat a little with some one but without permantent precense subscribtion.

```xml
<message from="george@im.company.org/office"
        to="garry.tech@im.company.org"
        type="chat">
    <body>I need the printer to fix. Can you help me?</body>
</message>

<presence from="george@im.company.org/office"
        to="garry.tech@im.company.org">
</presence>
```

And contact can respond with stanza like this:

```xml
<presence from="garry.tech@im.company.org"
        to="george@im.company.org/office"
        type="available" />
```

## Offline

To set your network status as offline you need to send stanza:

```xml
<presence type="unavailable" />
```

## Rich Presence

With precense you can also just change your status text, without changing 
network availability:

```xml
<presence>
    <status>At the coffe machine</status>
</presence>
```

## Presence and Rosters

When you send `iq` stanza to server to get roster list:

```xml
<iq from="george@im.company.org/office"
    id="un1qu31D"
    to="george@im.company.org"
    type="get">
    <query xmlns="jabber:iq:roster" />
</iq>
```

you will receive this kind of responce:

```xml
<iq from="george@im.company.org"
    id="un1qu31D"
    to="george@im.company.org/office"
    type="result">
    <query xmlns="jabber:iq:roster">
        <item jid="patric@im.company.org"/>
        <item jid="stuart@im.company.org"/>
        <item jid="Garry@im.company.org"/>
        <item jid="Potter@im.company.org"/>
    </query>
</iq>
```

In a real applications response will be more extended:


```xml
<iq from="george@im.company.org"
    id="un1qu31D"
    to="george@im.company.org/office"
    type="result">
    <query xmlns="jabber:iq:roster">
        <item jid="patric@im.company.org" name="Patric" subsciption="none" />
        <item jid="stuart@im.company.org" name="Stuart" subsciption="to" />
        <item jid="Garry@im.company.org" name="Garry" subsciption="from" />
        <item jid="Potter@im.company.org" name="Potter" subsciption="both" />
    </query>
</iq>
```

Also you can group contact list and the roster will look like this:


```xml
<iq from="george@im.company.org"
    id="un1qu31D"
    to="george@im.company.org/office"
    type="result">
    <query xmlns="jabber:iq:roster">
        <item jid="patric@im.company.org" name="Patric" subsciption="none">
            <group>Coworker</group>
        </item>
        <item jid="stuart@im.company.org" name="Stuart" subsciption="to">
            <group>Coworker</group>
        </item>
        <item jid="Garry@im.company.org" name="Garry" subsciption="from">
            <group>Coworker</group>
        </item>
        <item jid="Potter@im.company.org" name="Potter" subsciption="both">
            <group>Friends</group>
        </item>
    </query>
</iq>
```

Roster groups can be edited with next:

```xml

```

---

Notes and examples used from XMPP The Definitive Guide by Peter Saint-Andre

