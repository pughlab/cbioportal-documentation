---
layout: default
---

### [Deployment Guide](deployment-guide.html)

## 1. Authentication and authorization

cBioPortal uses Spring Security for its authentication system. At UHN, we use an OpenID Connect (OIDC) authentication system instead of direct use of LDAP. This is because UHN is slowly moving towards a single sign-on system based on OIDC. Although an institution-wide service doesn't yet exist, we have deployed an OIDC provider which is backed by LDAP/Active Directory servers on both the SIMS and RIS networks. When an institution-wide service is available, we'll need to reconfigure the OIDC secrets and URLs to work with that, but nothing else should need to change.

The identities returned by the OIDC service depend on the `email` scope, and are typically either an `@uhn.ca` or an `@uhnresearch.ca` address, depending on the authenticating server. These are the user identifiers used (currently) at UHN to identify users for authorization.

This is very largely because these values are readable. A more stable identity will probably be adopted in due course.

cBioPortal uses two database tables to manage users and permissions: `users` and `authorities`.

To add a new user to the users table, use an SQL command like:

```SQL
INSERT INTO users (email, name, enabled) VALUES ('Jane.Doe@uhn.ca', 'Jane Doe', 1);
```

(The hard part here is finding the email to use. This might well involve a search of the LDAP directory tree, as people may have several email addresses, and only one is returned by OIDC from the LDAP data.)

Authorization is based on the `authorities` table, and can be added using an SQL command like this:

```SQL
INSERT INTO authorities (email, authority) VALUES ('Jane.Doe@uhn.ca', 'BIOPORTAL AT UHN:GLOOP');
```

Where `GLOOP` is the name of the study that this user should be allowed to access.

Authorities do not seem to be cached by cBioPortal, so adding a new authorization to the tables doesn't require the server to be restarted.
