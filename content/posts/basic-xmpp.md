---
title: "Basic xmpp"
date: 2024-06-23T23:41:48+03:00
draft: false
tags: ['xmpp']
---

### Domains

Every JabberID (JID) contain domain portion. It's an server address. Every JID 
looks like this:

```
test@test.com
worker@chat.company.org
admin@chat.application.com
```

### Users

User name is case insensetive ACII characters string without special characters

### Resources

When you connect your client to an XMPP server server ( or manualy ) assign 
resoirce id for that particular connection. This is using for routing trafic to
connection instead of any other open connection ( if exists ). It looks like this:

```
alex@jabber.org/home
manager@chat.company.org/office
```

### XMPP URIs

see RFC5122 or XEP-0147

### Streaming XML

To start streaming xml you need to open long live TCP connection with XMPP server.
Main an the only message types exists in this stream are:
- `<message />`
- `<presense />`
- `<iq />`

This snippets called `stanza` .

# Stanza

### Message

Messages not asknowladged. Used in groupchat, alerts and notifications etc.
Message stanzas came in five types ( determin by the `type` attribute ):
- normal - This is simple message simular to email message
- chat - exchanged in real-time "sessions" between two entities, such as IM 
between two users
- groupchat - exchanged in multi-user chat room
- headline - this type of message used to send alerts and notifications without
expecting response at all. ( client should not able a user to reply )
- error - server send this type of messages if something was wrong

Also `<message>` contain `to` and `from` required attributes.

Messages also contain some payload elements: `<body/>` and `<subject />`. So in 
overal basic message can looks like this:

```xml
<message from="madhatter@wonderland.lit/foo"
        to="alice@wonderland.lit"
        type="chat">
    <body>Who are you?</body>
    <subject>Query</subject>
</message>
```

### Presense

Presense stanzas used for udpating avaliability status in the network. But not 
every user on server can see your status. At first he should be subscribed to 
see your status updates. This is broader topic for discussion than the current
artice. But basic presence message can look's like this:

```xml
<presence from="george@im.company.org/office">
    <show>xa</show>
    <status>Making coffe on the kitched</status>
</presense>
```

### IQ

The "Info Query" stanza used to get or set some information or settings on the 
XMPP server. This type of stanzas should return replyes tracked by `id` attribute.
It also has `type` attribute:
- get - for requesting some information
- set - for changing
- result - responging entity after request
- error - if response with some errors

Basic `iq` stanza can look like this:

```xml
<iq from="george@im.company.org/office"
    id="un1qu31D"
    to="george@im.company.org"
    type="get">
    <query xmlns="jabber:iq:roster" />
</iq>
```
( George asked for contact list ) . The response can be liike this:

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

Adding new contact to list:

```xml
<iq from="george@im.company.org/office"
    id="n3wun1qu31D"
    to="george@im.company.org"
    type="set">
    <query xmlns="jabber:iq:roster">
        <item jid="joseph@im.company.org"/>
    </query>
</iq>
```
In this case server will return empty result `iq`:

```xml
<iq from="george@im.company.org"
    id="n3wun1qu31D"
    to="george@im.company.org/office"
    type="result" />
```

Did you notice how resource portion of address work's in this type of messages?


# To be continued...

In the next articles we will deep dive into XMPP messaging mechanism with more
interesting examples :)

---

Notes and examples used from XMPP The Definitive Guide by Peter Saint-Andre

