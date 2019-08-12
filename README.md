# taiga-docker

A small Docker Image for the open-source project management software Taiga.io
- The recipe is based on the existing recipe
by [benhutchins](https://github.com/benhutchins): see
<https://github.com/benhutchins/docker-taiga> for his approach. Then forked
from [fanningert/docker-taiga](https://github.com/fanningert/docker-taiga)
since it was unmaintained.

Tested / supported versions of Taiga:

- Version 3.2.2
- Version 4.2.1 to 4.2.12

## Available Docker Images at DockerHub

Image Name  | Tag        | Taiga Version
------------|------------|---------------
m0wer/docker-taiga | latest      | 4.2.12
m0wer/docker-taiga | v4.2.12      | 4.2.12
...                | ...         | ...
m0wer/docker-taiga | v4.2.1      | 4.2.1
m0wer/docker-taiga | test        | ?

See: <https://hub.docker.com/r/m0wer/docker-taiga/>

## Image details

- The image contains Taiga backend and Taiga frontend.
- Frontend and backend are served by a Nginx server instance running on port 80.
- LDAP plugin from https://github.com/ensky/taiga-contrib-ldap-auth is integrated and can be enabled if required.
- The Nginx server instance accepts HTTP connections only (no HTTPS), but you can create an own image that enables HTTPS by adjusting the nginx.conf template.
- PostgreSQL database server is not included, i.e. a db server instance is expected to be running elsewhere (either in a different container that is linked, or hosted somewhere else).
- (Open-)LDAP server is not included.
- The image is published on Dockerhub as `m0wer/docker-taiga`

Important environment parameters for configuration:
- `TAIGA_HOSTNAME`: The name of the server/domain, through which the Taiga instance will be accessed in the end, e.g. `taiga.yourdomain.tld` (default: `localhost`).
- `TAIGA_SSL`: Defines whether the Taiga instance will be accessible through SSL or not, (default: `false`).
- `TAIGA_SECRET_KEY`: Defines the secret key used internally by Taiga (default: `!!!PLEASE-REPLACE-ME!!!`), e.g. for password hashing or similar operations. Please change the default value.
- `TAIGA_DB_HOST`: Defines the hostname of the server on which the PostgreSQL instance is running (default: `localhost`).
- `TAIGA_DB_NAME`: Defines the database name on the PostgreSQL instance (default: `postgres`).
- `TAIGA_DB_USER`: Defines the database username for PostgreSQL connections (default: `postgres`).
- `TAIGA_DB_PASSWORD`: Defines the password for database connections (default: `!!!PLEASE-REPLACE-ME!!!`).
- `TAIGA_DEFAULT_FROM_EMAIL`: Default "from" email address for notification
  emails (default: `john@doe.com`).
- `TAIGA_EMAIL_BACKEND`: Backend for sending emails (default:
   `django.core.mail.backends.smtp.EmailBackend`).
- `TAIGA_EMAIL_USE_TLS`: Boolean to use or not TLS with the email server
   (default: `false`).
- `TAIGA_EMAIL_USE_SSL`: Boolean to use or not SSL with the email server
   (default: `false`).
- `TAIGA_EMAIL_HOST`: Email server host (default: `localhost`).
- `TAIGA_EMAIL_PORT`: Email server port (default: `25`).
- `TAIGA_EMAIL_USER`: Email server user, if login is required.
- `TAIGA_EMAIL_PASSWORD`: Email server password, if login is required.
- `TAIGA_PUBLIC_REGISTER_ENABLED`: Defines whether new user registrations are enabled (default: `false`).
- `TAIGA_BACKEND_DEBUG`: Enables or disables backend debugging (default: `false` ).
- `TAIGA_FRONTEND_DEBUG`: Enables or disables frontend debugging (default: `false`).
- `TAIGA_FEEDBACK_ENABLED`: Enables sending of feedback to Taiga.io developers (default: `false`).
- `TAIGA_DEFAULT_LANGUAGE`: Defines the default language in the frontend (default: `en`).
- `TAIGA_DEFAULT_THEME`: Defines the default theme to be used (default: `material-design`).
- `LDAP_ENABLE`: Enables or disables support for LDAP-based authentication (default: `false`).
- `LDAP_SERVER`: The hostname or IP address of the LDAP server (default: empty).
- `LDAP_PORT`: The port on which the LDAP server is listening (default: `389`).
- `LDAP_BIND_DN`: Defines the user to be used for LDAP queries (default: empty).
- `LDAP_BIND_PASSWORD`: Sets the password of this user (default: empty).
- `LDAP_SEARCH_BASE`: The base below which to search for user accounts (default: empty).
- `LDAP_SEARCH_PROPERTY`: The property field to query for when looking up user accounts (default: `sAMAccountName`).
- `LDAP_EMAIL_PROPERTY`: The property in which the email address of a user is stored (default: `mail`).
- `LDAP_FULL_NAME_PROPERTY`: The property in which the full name of a user is stored (default: `displayName`).
- `TAIGA_BACKEND_WEBHOOKS_ENABLED`: Activate webhooks (default: `false`).
- `TAIGA_BACKEND_WEBHOOKS_BLOCK_PRIVATE_ADDRESS`: Disallow webhooks access to reserved and private address (default: `true`).

Important directories inside of the image:
- Backend distribution is located in `/taiga.io/taiga-back`
- Frontend distribution is located in `/taiga.io/taiga-front`
- Uploaded media files (e.g. attachments, project logos, profile pcitures) are stored in `/taiga.io/taiga-back/media`

## Example usage with docker-compose

The following docker-compose.yml recipe shows how you can easily start your own Taiga instance, in combination with an own PostgreSQL container image. The important directories of both containers are linked as volumes, for easy backup.

```
postgres:
  image: postgres:9.6.1-alpine
  environment:
    - POSTGRES_DB=taiga
    - POSTGRES_USER=taiga
    - POSTGRES_PASSWORD=3MwR95cj9YAP7zm2lKrU
  volumes:
    - taiga-db:/var/lib/postgresql/data

taiga:
  image: m0wer/docker-taiga
  ports:
    - 80:80
  links:
    - postgres
  environment:
    - TAIGA_HOSTNAME=localhost
    - TAIGA_DB_HOST=postgres
    - TAIGA_DB_NAME=taiga
    - TAIGA_DB_USER=taiga
    - TAIGA_DB_PASSWORD=3MwR95cj9YAP7zm2lKrU
    - TAIGA_SECRET_KEY=<YOUR_SECRET_KEY_HERE>
    - TAIGA_PUBLIC_REGISTER_ENABLED=true
    - TAIGA_BACKEND_DEBUG=false
    - TAIGA_FRONTEND_DEBUG=false
    - TAIGA_FEEDBACK_ENABLED=false
    - TAIGA_DEFAULT_LANGUAGE=de
    - TAIGA_SSL=false
    - TAIGA_DEFAULT_THEME=material-design
    - LDAP_ENABLE=false
  volumes:
    - taiga-media:/taiga.io/taiga-back/media
```

To bring both up, simply type:

`docker-compose up -d`
