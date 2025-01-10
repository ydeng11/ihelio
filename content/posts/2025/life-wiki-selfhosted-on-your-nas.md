---
title: "Life Wiki Selfhosted on Your NAS"
date: 2025-01-10T00:28:28-05:00
draft: false
categories: 
    - Programming
tags: 
    - NAS
    - Selfhosted
    - Wiki
    - Docker
---

# 

# Introduction

I used Notion for a couple of years and found it to be one of the best tools for note-taking and building a personal wiki. Why did I stop using it? It wasn't about the cost—Notion's freemium plan is perfectly sufficient for personal use. Instead, there were two main issues that drove me away.

First, typing math equations in Notion is cumbersome because you have to use a dedicated "Block" for them. Second, it's difficult to export or migrate your data to other platforms, which, while not entirely Notion's fault, still concerns me. Most importantly, I realized I should make better use of my Synology NAS. That's why I decided to explore open-source, self-hosted alternatives.

# Choices

To my surprise, there are more choices in the market than I could imagine in the beginning.

| **app name**  | **collaboration** | **cross platform** | **self-hosted server** | **browser app** | **knowledge management** | **selfhost score** |
| ------------- | ----------------- | ------------------ | ---------------------- | --------------- | ------------------------ | ------------------ |
| Silverbullets | N                 | Y                  | Y                      | Y               | ⭐⭐⭐                      | ⭐⭐⭐⭐⭐              |
| StandardNotes | N                 | Y                  | Y                      | Y               | ⭐⭐⭐⭐⭐                    | ⭐                  |
| Siyuan        | N                 | Y                  | N                      | N               | ⭐⭐⭐⭐⭐                    | ⭐                  |
| Bookstack     | N                 | Y                  | Y                      | Y               | ⭐⭐⭐⭐                     | ⭐⭐⭐⭐⭐              |
| Obsidian      | N                 | Y                  | N                      | N               | ⭐⭐⭐⭐⭐                    | ⭐⭐⭐                |
| LogSeq        | N                 | Y                  | N                      | N               | ⭐⭐⭐⭐⭐                    | ⭐⭐⭐⭐⭐              |
| Trilium       | N                 | Y                  | Y                      | Y               | ⭐⭐⭐⭐⭐                    | ⭐⭐⭐⭐⭐              |
| Joplin        | N                 | Y                  | Y                      | N               | ⭐                        | ⭐⭐⭐⭐⭐              |
| UseMemos      | N                 | Y                  | Y                      | Y               | ⭐                        | ⭐⭐⭐⭐⭐              |
| Wiki.js       | N                 | Y                  | Y                      | Y               | ⭐⭐⭐⭐⭐                    | ⭐⭐⭐⭐⭐              |
| Appflowy      | Y                 | Y                  | Y                      | N               | ⭐⭐⭐⭐⭐                    | ⭐                  |
| Affine        | Y                 | Y                  | Y                      | Y               | ⭐⭐⭐⭐⭐                    | ⭐⭐                 |
| AnyType       | Y                 | Y                  | Y                      | N               | ⭐⭐⭐⭐⭐                    | ⭐⭐                 |
| Docmost       | Y                 | Y                  | Y                      | Y               | ⭐⭐⭐⭐                     | ⭐⭐⭐⭐⭐              |
| Outline       | Y                 | Y                  | Y                      | Y               | ⭐⭐⭐⭐                     | ⭐⭐                 |

I tested each self-hosted tool at a basic level to see if it met my needs. Two must-have features for me are collaboration and a lightweight browser-based interface. Lastly, I'm looking at how easy it is to self-host and how truly they are self-hosted. Here's my shortlist:

- Affine – I ruled this out because it doesn't feel truly open source or self-hosted. There are ongoing GitHub discussions about this point.
- Docmost – It seems promising, but the community is still at an early stage.
- Outline – I ended up selecting Outline because it provides all the features I need and has a strong community. However, hosting it wasn't straightforward—it enforces a specific authentication process, which took me a couple of days to figure out.

I also tried Appflowy and AnyType, both of which came close to meeting my requirements. However, Appflowy imposes many limitations on self-hosting, and AnyType is resource-heavy, requiring MongoDB, Minio, and multiple sync nodes. By contrast, Outline can simply use a local filesystem, which has worked very well for me so far.

# Selfhost Outline & Authenlia

I chooses to use `Authenlia` as the authentication service for `Outline` as I need a lightweight solution given it will only be accessed by my family and again I can use the local filesystem to power `Authenlia`.

## Authenlia

To host Authenlia, we need create the `configuration.yml`:

```yaml
log:
  level: info

server:
  address: 'tcp://:9091/'

session:
  name: authelia_session
  same_site: lax
  secret: "redacted"
  expiration: 3600
  inactivity: 300
  cookies:
    - domain: "your.domain"
      authelia_url: "https://your.auth.domain"
      default_redirection_url: "https://www.your.outline.domain"
      name: "authelia_session"
      same_site: "lax"
      inactivity: "5m"
      expiration: "1h"
      remember_me: "1d"

access_control:
  default_policy: one_factor
  rules: []

storage:
  encryption_key: "redacted"
  local:
    path: /config/db.sqlite3

identity_providers:
  oidc:
    hmac_secret: "redacted"
    jwks:
      - key: {{ secret "/config/private.pem" | mindent 10 "|" | msquote }}
    clients:
      - id: "unique client id"
        description: "Outline Wiki"
        secret: "redacted"
        public: false
        authorization_policy: one_factor
        redirect_uris:
          - "https://your.outline.domain/auth/oidc.callback"
        scopes:
          - "openid"
          - "offline_access"
          - "profile"
          - "email"
        userinfo_signed_response_alg: "none"
        token_endpoint_auth_method: "client_secret_post"

notifier:
  disable_startup_check: false
  filesystem:
    filename: "/config/notification.txt"

authentication_backend:
  file:
    path: /config/user_database.yml

identity_validation:
  reset_password:
    jwt_secret: "redacted"
    jwt_lifespan: "5 minutes"
    jwt_algorithm: "HS256"
```

You could run `openssl rand -hex 32` to generate the key for the redacted place above.

You also need create a hash of the password using `docker run --rm -it authelia/authelia:latest authelia crypto hash generate argon2` which prompt for your password. After you have the hash, you could go ahead creating the `user_database.yml` file storing the login data:

```yaml
users:
  admin:
    displayname: "Foo Poo"
    password: "the hash generated from above command"
    email: "email"
    groups:
      - admins
      - dev
```

When you login, you need use `admin` as the user name and the password you typed in to generate the hash. It is not the password and email you used under `admin` stanza above. And you don't have to create many users since `Outline` doesn't support multiple workspaces in selfhosted version ([discussions](https://github.com/outline/outline/issues/6837)).

The last one is the jwks key we could generate with the command:

```bash
docker run --rm -v "$(pwd)":/output authelia/authelia:latest authelia crypto certificate rsa generate --directory /output
```

which will generate `private.pem` and `public.crt` at present directory and we only need `private.pem` here.

After we have all the files ready, we could create the `docker-compose.yaml`:

```docker
services:
  authelia:
    image: authelia/authelia:4.38
    container_name: authelia
    restart: unless-stopped
    environment:
      - TZ=UTC
      - X_AUTHELIA_CONFIG_FILTERS=template
    ports:
      - "9095:9091"
    volumes:
      # this mounted folder should have all three files you just created: 
      # configuration.yml, user_database.yaml and private.pem
      - '/your/folder/authelia/config:/config'
```

## Outline

This is my `docker-compose.yaml` hosting `Outline` on my NAS:

```docker
version: "3.7"
services:
  outline:
    image: outlinewiki/outline:0.81.1
    # this is to give Outline container the permission to access the mounted folder, which could
    # be different for you. You could use `ls -nd` to check the UID and GID for a particular folder
    user: "1026:100" 
    ports:
      - "3002:3000"
    depends_on:
      - postgres
      - redis
    volumes:
      - /yours/outline/storage:/var/lib/outline/data
    networks:
      - npm_network
    environment:
      NODE_ENV: production
      SECRET_KEY: "key generated using openssl rand -hex 32"
      UTILS_SECRET: "key generated using openssl rand -hex 32" 
      # HTTP
      # URL must be the same one used in your authenlia configuration.yml
      URL: https://your.outline.domain
      PORT: 3000
      FORCE_HTTPS: true
      WEB_CONCURRENCY: 1
      # Rate limiter
      RATE_LIMITER_ENABLED: true
      RATE_LIMITER_DURATION_WINDOW: 60
      RATE_LIMITER_REQUESTS: 600
      # Authentication
      OIDC_CLIENT_ID: "must be the same as the one used in authenlia"
      OIDC_CLIENT_SECRET: "must be the same as the one used in authenlia"
      # The OIDC config can be found from https://your.auth.domain/.well-known/openid-configuration
      OIDC_AUTH_URI: https://your.auth.domain/api/oidc/authorization
      OIDC_TOKEN_URI: https://your.auth.domain/api/oidc/token
      OIDC_USERINFO_URI: https://your.auth.domain/api/oidc/userinfo
      OIDC_LOGOUT_URI: https://your.outline.domain
      OIDC_USERNAME_CLAIM: username
      OIDC_SCOPES: openid offline_access profile email
      # Storage
      FILE_STORAGE: local
      FILE_STORAGE_LOCAL_ROOT_DIR: /var/lib/outline/data
      FILE_STORAGE_UPLOAD_MAX_SIZE: 1086214400
      # Database
      DATABASE_URL: postgres://user:pass@postgres:5432/outline
      PGSSLMODE: disable
      # Redis
      REDIS_URL: redis://redis:6379
      # Other
      LOG_LEVEL: info
      ENABLE_UPDATES: true
      DEFAULT_LANGUAGE: en_US

  redis:
    image: redis:7.4.1
    expose:
      - 6379
    networks:
      - npm_network
    command: redis-server

  postgres:
    image: postgres:12.22
    expose:
      - 5432
    networks:
      - npm_network
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: outline
    volumes:
      - /volume1/docker/outline/postgres:/var/lib/postgresql/data

networks:
  # we need use bridge so Nginx could forward the traffic
  npm_network:
    driver: bridge
```

This should get your `Outline` up and running and the last is to configure the proxy host at the NPM (aka. Nginx Proxy Manager) so we could access it from internet.

## Nginx Proxy Manager

You can easily create an SSL certificate with `Let's Encrypt` through Nginx Proxy Manager (NPM). When generating the certificate, make sure it includes both domains used to reach Outline and Authenlia (for example, `your.outline.domain` and `your.auth.domain`). You might also want to create a certificate for your top-level domain if needed. After that, simply create proxy hosts for each domain, ensuring they point to the correct internal addresses.

Use http scheme for the destinations otherwise you prob will see 502 error. At least I have to use http scheme here and it will be secure since we enforce HTTPS on server side and use SSL cert for the connection.

Regarding the destination address, I tried to use `service:port` since they are all on bridge network but unfortunately it doesn't work for me. I have to use the `nas-ip:host-port` to access it. Finally, remember to forward ports **80** (HTTP) and **443** (HTTPS) from your router to NPM. This ensures all inbound web traffic is properly routed through Nginx Proxy Manager, allowing it to forward the traffic from internet.

# Conclusion

I recently migrated all my notes from `Notion` and `Obsidian` to `Outline` and couldn't be happier so far. This document is the first I've created post-migration, and the experience has been smooth—especially `Outline`'s robust Markdown support for math equations. The one drawback I've noticed is the lack of a birds-eye view, which makes it cumbersome to manage numerous documents solely from the sidebar. Fortunately, there's ongoing discussion about adding database support, which could help create a customized homepage and improve overall organization.
