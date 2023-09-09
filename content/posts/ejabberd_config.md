---
title: "Ejabberd config"
date: 2023-09-09T22:08:18+03:00
draft: false
tags: ['ejabberd', 'docker', 'administration']
---

> IMPORTANT! I'm not an expert in setting up ejabberd in a proper and most important
secure way. This config is my first attempt to set up things in a way I want to setup.
So this article will be updated soon...

Ejabberd is an amazing platform allowing you to setup communication between users fast.
In my case it should be as an microservice ( cuz' why should I need to reinvent the 
wheel and write chat from scratch ? ). Final code is [here](https://github.com/shaninalex/ejabberd-docker-compose-config)

So to do that create file somewhere in your project directory `./config/ejabberd/ejabberd.yml` 
and put there content:

```yml
hosts:
  - localhost

loglevel: 4
log_rotate_size: 10485760
log_rotate_date: ""
log_rotate_count: 1
log_rate_limit: 100

certfiles:
  - /home/ejabberd/conf/server.pem

ca_file: "/home/ejabberd/conf/cacert.pem"

## When using let's encrypt to generate certificates
##certfiles:
##  - /etc/letsencrypt/live/localhost/fullchain.pem
##  - /etc/letsencrypt/live/localhost/privkey.pem
##
##ca_file: "/etc/letsencrypt/live/localhost/fullchain.pem"
#

listen:
  -
    port: 5222
    ip: "::"
    module: ejabberd_c2s
    max_stanza_size: 262144
    shaper: c2s_shaper
    access: c2s
    starttls_required: true
  -
    port: 5269
    ip: "::"
    module: ejabberd_s2s_in
    max_stanza_size: 524288
  -
    port: 5443
    ip: "::"
    module: ejabberd_http
    tls: true
    request_handlers:
      "/admin": ejabberd_web_admin
      "/api": mod_http_api
      "/bosh": mod_bosh
      "/captcha": ejabberd_captcha
      "/upload": mod_http_upload
      "/ws": ejabberd_http_ws
      "/oauth": ejabberd_oauth
  -
    port: 5280
    ip: "::"
    module: ejabberd_http
    request_handlers:
      "/admin": ejabberd_web_admin
      "/api": mod_http_api
      "/oauth": ejabberd_oauth

      
  -
    port: 1883
    ip: "::"
    module: mod_mqtt
    backlog: 1000
  ##
  ## https://docs.ejabberd.im/admin/configuration/#stun-and-turn
  ## ejabberd_stun: Handles STUN Binding requests
  ##
  ##-
  ##  port: 3478
  ##  ip: "0.0.0.0"
  ##  transport: udp
  ##  module: ejabberd_stun
  ##  use_turn: true
  ##  turn_ip: "{{ IP }}"
  ##  auth_type: user
  ##  auth_realm: "example.com"
  ##-
  ##  port: 3478
  ##  ip: "0.0.0.0"
  ##  module: ejabberd_stun
  ##  use_turn: true
  ##  turn_ip: "{{ IP }}"
  ##  auth_type: user
  ##  auth_realm: "example.com"
  ##- 
  ##  port: 5349
  ##  ip: "0.0.0.0"
  ##  module: ejabberd_stun
  ##  certfile: "/home/ejabberd/conf/server.pem"
  ##  tls: true
  ##  use_turn: true
  ##  turn_ip: "{{ IP }}"
  ##  auth_type: user
  ##  auth_realm: "example.com"
  ##
  ## https://docs.ejabberd.im/admin/configuration/#sip
  ## To handle SIP (VOIP) requests:
  ##
  ##-
  ##  port: 5060
  ##  ip: "0.0.0.0"
  ##  transport: udp
  ##  module: ejabberd_sip
  ##-
  ##  port: 5060
  ##  ip: "0.0.0.0"
  ##  module: ejabberd_sip
  ##-
  ##  port: 5061
  ##  ip: "0.0.0.0"
  ##  module: ejabberd_sip
  ##  tls: true

s2s_use_starttls: optional

acl:
  admin:
    user:
      - "admin@localhost"

access_rules:
  local:
    allow: local
  c2s:
    deny: blocked
    allow: all
  announce:
    allow: admin
  configure:
    allow: admin
  muc_create:
    allow: local
  pubsub_createnode:
    allow: local
  trusted_network:
    allow: loopback

oauth_access: all

api_permissions:
  "console commands":
    from:
      - ejabberd_ctl
    who: all
    what: "*"
  "admin access":
    who:
      - access:
          - allow:
            - acl: admin
      - oauth:
        - scope: "ejabberd:admin"
        - access:
          - allow:
              - acl: admin
    what:
      - "*"
      - "!stop"
      - "!start"

shaper:
  normal: 1000
  fast: 50000

shaper_rules:
  max_user_sessions: 10
  max_user_offline_messages:
    5000: admin
    100: all
  c2s_shaper:
    none: admin
    normal: all
  s2s_shaper: fast

max_fsm_queue: 10000

acme:
   contact: "mailto:example-admin@example.com"
   ca_url: "https://acme-staging-v02.api.letsencrypt.org/directory"

modules:
  mod_adhoc: {}
  mod_admin_extra: {}
  mod_announce:
    access: announce
  mod_avatar: {}
  mod_blocking: {}
  mod_bosh: {}
  mod_caps: {}
  mod_carboncopy: {}
  mod_client_state: {}
  mod_configure: {}
  mod_disco: {}
  mod_fail2ban: {}
  mod_http_api: {}
  mod_http_upload:
    put_url: https://@HOST@:5443/upload
  mod_last: {}
  mod_mam:
    ## Mnesia is limited to 2GB, better to use an SQL backend
    ## For small servers SQLite is a good fit and is very easy
    ## to configure. Uncomment this when you have SQL configured:
    ## db_type: sql
    assume_mam_usage: true
    default: never
  mod_mqtt: {}
  mod_muc:
    access:
      - allow
    access_admin:
      - allow: admin
    access_create: muc_create
    access_persistent: muc_create
    access_mam:
      - allow
    default_room_options:
      allow_subscription: true  # enable MucSub
      mam: false
  mod_muc_admin: {}
  mod_offline:
    access_max_user_messages: max_user_offline_messages
  mod_ping: {}
  mod_privacy: {}
  mod_private: {}
  mod_proxy65:
    access: local
    max_connections: 5
  mod_pubsub:
    access_createnode: pubsub_createnode
    plugins:
      - flat
      - pep
    force_node_config:
      ## Avoid buggy clients to make their bookmarks public
      storage:bookmarks:
        access_model: whitelist
  mod_push: {}
  mod_push_keepalive: {}
  mod_register:
    ## Only accept registration requests from the "trusted"
    ## network (see access_rules section above).
    ## Think twice before enabling registration from any
    ## address. See the Jabber SPAM Manifesto for details:
    ## https://github.com/ge0rg/jabber-spam-fighting-manifesto
    ip_access: trusted_network
  mod_roster:
    versioning: true
  mod_sip: {}
  mod_s2s_dialback: {}
  mod_shared_roster: {}
  mod_stream_mgmt:
    resend_on_timeout: if_offline
  mod_vcard: {}
  mod_vcard_xupdate: {}
  mod_version:
    show_os: false
```

This setup is quite big, but it's okay. Next add a service to your docker compose file:

```yml

services:
  # ---------------
  ejabberd:
    image: ejabberd/ecs
    container_name: chat_app
    ports:
      - 5222:5222 # client to server
      - 5280:5280 # web
    volumes:
      - ./config/ejabberd/ejabberd.yml:/home/ejabberd/conf/ejabberd.yml
  # ---------------
```

And start docker compose. 
After all things successfully started we need to create admin user and issue oauth
token ( [ejabberd docs](https://docs.ejabberd.im/developer/ejabberd-api/oauth/) ) for him:

```bash
# create user
$ docker exec -it chat_app /home/ejabberd/bin/ejabberdctl register admin localhost adm123

# issue token
$ docker exec -it chat_app /home/ejabberd/bin/ejabberdctl oauth_issue_token admin@localhost 3600 ejabberd:admin
2IfdlCG9BGdK94uRUvqF8tt5envdol6U	[<<"ejabberd:admin">>]	3600 seconds
```

Now you can use this token with [API calls](https://docs.ejabberd.im/developer/ejabberd-api/admin-api/) 
and managing ejabberd server - create users, rooms etc. It was the main idea 
behind all this shenanigans.

## Objectives

For example, we have a large application. And we need to "add" a simple chat. And this 
is the solution. We integrate a new service into the architecture, configure it, and 
"expose" the necessary ports to integrate the entire application with the chat.
At least that's how I see it.

## TODO

I need to develop an interface-application that will:

- Synchronize user creation.
- Automate the creation and updating of tokens.
- Provide user management functionality for administrators (ban, unban, etc.).

I believe it should be easier than creating a chat app from scratch... And you still 
need to create a management interface for it! So I think creating an interface app 
around a [well-documented API](https://docs.ejabberd.im/developer/ejabberd-api/admin-api/) 
is much easier. Maybe I will write a post about it...

And most importantly, you need to secure the application by correctly configuring all 
access, closing unnecessary ports, using only secure SSL connections, and generating 
valid certificates.

So, if you are an experienced sysadmin and want to write an angry letter about a 
"misconfigured" setup, go ahead! I will definitely read it and update the post 
accordingly.

### Used documentation and posts
- [Official documentation](https://docs.ejabberd.im/get-started/)
- [Issue on Github about API accesses](https://github.com/processone/ejabberd/issues/4008)
- [docker ejabberd](https://github.com/processone/docker-ejabberd)
