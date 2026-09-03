# Geoapify HTTPS Integration with Oracle Database 19c and Oracle APEX

This document describes how to configure an Oracle Database 19c environment to call the **Geoapify Address Autocomplete API** over HTTPS from PL/SQL using `UTL_HTTP`.

The intended architecture is:

```text
Oracle APEX 24.2.19
        |
        v
APEX_SCHEMA
        |
        | UTL_HTTP
        v
Network ACL
        |
        +---- DNS RESOLVE
        |
        +---- TCP CONNECT :443
        |
        v
Oracle Wallet
        |
        | TLS trust
        v
api.geoapify.com:443
        |
        v
Geoapify Autocomplete API
```


## Environment

| Component | Value |
|---|---|
| Oracle Database | 19c |
| Operating System | Oracle Linux 8 |
| `ORACLE_HOME` | `/opt/oracle/product/19c/dbhome_1` |
| Oracle APEX | 24.2.19 |
| APEX Application | 11 |
| Application Schema | `APEX_SCHEMA` |
| Geoapify Host | `api.geoapify.com` |
| Protocol | HTTPS |
| Port | `443` |
| Wallet | `/opt/oracle/wallets/geoapify` |


---

## 1. Prerequisites

You need:

- Oracle Database 19c
- Oracle Linux 8
- Oracle APEX 24.2.19
- Access to the Oracle Database server
- `sudo` or root access for wallet-directory setup
- SYSDBA access for network ACL configuration
- Access to the `APEX_SCHEMA` schema for testing
- `orapki`
- `openssl`
- Linux DNS/network access to `api.geoapify.com:443`
- A valid Geoapify API key for the final API test

Verify the Oracle home:

```bash
echo "$ORACLE_HOME"
```

Expected:

```text
/opt/oracle/product/19c/dbhome_1
```

Verify `orapki`:

```bash
/opt/oracle/product/19c/dbhome_1/bin/orapki 
```

---

## 2. Verify Linux Can Reach Geoapify

Perform this test on the Oracle Database server before changing Oracle configuration.

### DNS

```bash
nslookup api.geoapify.com
```

If `nslookup` is unavailable:

```bash
getent hosts api.geoapify.com
```

### HTTPS/TCP 443

```bash
curl -Iv https://api.geoapify.com/
```

An HTTP response such as `404`, `401`, or `403` is acceptable for this basic connectivity test. The important point is that DNS and the TCP/TLS connection succeed.

Problems such as these indicate a Linux/network problem:

```text
Could not resolve host
Connection timed out
No route to host
```

If Linux cannot reach the endpoint, fix firewall, proxy, NAT, DNS, or Internet-egress issues before continuing with Oracle configuration.

---

## 3. Create a Dedicated Oracle Wallet Directory

Use a dedicated wallet outside the Oracle Database home:

```text
/opt/oracle/wallets/geoapify
```

As `root`:

```bash
mkdir -p /opt/oracle/wallets/geoapify
chown oracle:oinstall /opt/oracle/wallets/geoapify
chmod 700 /opt/oracle/wallets/geoapify
```

If the Oracle OS user's primary group is not `oinstall`, verify it:

```bash
id oracle
```

Use the appropriate Oracle group when running `chown`.

Switch to the Oracle OS user:

```bash
su - oracle
```

---

## 4. Create the Oracle Wallet

Create an auto-login wallet:

```bash
/opt/oracle/product/19c/dbhome_1/bin/orapki wallet create \
  -wallet /opt/oracle/wallets/geoapify \
  -auto_login
```

Depending on the exact `orapki` build, it may prompt for a wallet password.

If your version expects `-pwd`, use:

```bash
/opt/oracle/product/19c/dbhome_1/bin/orapki wallet create \
  -wallet /opt/oracle/wallets/geoapify \
  -pwd 'YOUR_STRONG_WALLET_PASSWORD' \
  -auto_login
```

Verify the wallet files:

```bash
ls -la /opt/oracle/wallets/geoapify
```

You should see files similar to:

```text
ewallet.p12
cwallet.sso
```

The auto-login wallet file is:

```text
cwallet.sso
```

---

## 5. Inspect Geoapify's Current TLS Certificate Chain

Do **not** blindly download and pin the current `api.geoapify.com` leaf certificate.

Inspect the live certificate chain instead:

```bash
openssl s_client \
  -connect api.geoapify.com:443 \
  -servername api.geoapify.com \
  -showcerts </dev/null
```

Near the end of the output, look for:

```text
Verify return code: 0 (ok)
```

The output also contains the certificate chain.

A cleaner version is:

```bash
openssl s_client \
  -connect api.geoapify.com:443 \
  -servername api.geoapify.com \
  -showcerts </dev/null 2>/dev/null
```

The objective is to identify the CA certificates needed to establish trust for the Geoapify server certificate.

> **Important:** The actual CA chain can change over time. Do not hard-code a particular CA name into this procedure without verifying the current chain.

---

## 6. Save the Certificate Chain for Inspection

Create a temporary directory:

```bash
mkdir -p /tmp/geoapify-certs
cd /tmp/geoapify-certs
```

Save the presented chain:

```bash
openssl s_client \
  -connect api.geoapify.com:443 \
  -servername api.geoapify.com \
  -showcerts </dev/null 2>/dev/null \
  > geoapify-chain.txt
```

Inspect it:

```bash
cat geoapify-chain.txt
```

The output contains one or more certificate blocks:

```text
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```

The first certificate is normally the server/leaf certificate. Following certificates are normally intermediate CA certificates.

---

## 7. Identify the Issuer and Root CA

For individual certificate files, inspect the subject and issuer:

```bash
openssl x509 -in certificate.pem -noout -subject -issuer
```

Conceptually, the chain may look like:

```text
subject=CN=api.geoapify.com
issuer=CN=Intermediate CA
```

followed by:

```text
subject=CN=Intermediate CA
issuer=CN=Root CA
```

The exact CA names are not fixed and should be determined from the current certificate chain.

---

## 8. Check Oracle Linux System CA Certificates

Oracle Linux 8 has a system CA store. Useful locations include:

```text
/etc/pki/ca-trust/
/etc/pki/ca-trust/source/anchors/
/etc/pki/ca-trust/extracted/pem/
```

You can inspect the system trust store with:

```bash
trust list
```

The Oracle wallet should contain the appropriate **trusted CA certificates**, rather than only the current Geoapify leaf certificate.

This is preferable because the Geoapify server certificate can be renewed while the issuing CA remains trusted.

Conceptually:

```text
Trusted Root CA
       |
       v
Intermediate CA
       |
       v
api.geoapify.com
```

---

## 9. Import Trusted CA Certificates into the Oracle Wallet

Suppose the required certificates have been identified as:

```text
/tmp/geoapify-certs/root-ca.pem
/tmp/geoapify-certs/intermediate-ca.pem
```

Import the root CA:

```bash
/opt/oracle/product/19c/dbhome_1/bin/orapki wallet add \
  -wallet /opt/oracle/wallets/geoapify \
  -trusted_cert \
  -cert /tmp/geoapify-certs/root-ca.pem
```

Then import the intermediate CA:

```bash
/opt/oracle/product/19c/dbhome_1/bin/orapki wallet add \
  -wallet /opt/oracle/wallets/geoapify \
  -trusted_cert \
  -cert /tmp/geoapify-certs/intermediate-ca.pem
```

Use:

```text
-trusted_cert
```

not:

```text
-user_cert
```

This integration does not use client-certificate authentication with Geoapify. The API key is separate from TLS trust.

---

## 10. Verify the Oracle Wallet

Run:

```bash
/opt/oracle/product/19c/dbhome_1/bin/orapki wallet display \
  -wallet /opt/oracle/wallets/geoapify
```

Verify that the expected CA certificates appear under:

```text
Trusted Certificates:
```

Also verify permissions:

```bash
ls -la /opt/oracle/wallets/geoapify
```

The Oracle Database OS user must be able to read:

```text
cwallet.sso
ewallet.p12
```

---

## 11. Configure the Oracle Network ACL

Connect as SYSDBA:

```bash
sqlplus / as sysdba
```

The configuration uses two ACEs:

1. `CONNECT` to `api.geoapify.com` on TCP port `443`
2. `RESOLVE` for `api.geoapify.com` without a port restriction

This keeps the network permission scoped to the application schema and the required HTTPS port.

---

## 12. Add CONNECT Permission for TCP 443

As SYS:

```sql
BEGIN
    DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
        host       => 'api.geoapify.com',
        lower_port => 443,
        upper_port => 443,
        ace        => XS$ACE_TYPE(
            privilege_list => XS$NAME_LIST('connect'),
            principal_name => 'APEX_SCHEMA',
            principal_type => XS_ACL.PTYPE_DB
        )
    );
END;
/
```

Expected:

```text
PL/SQL procedure successfully completed.
```

---

## 13. Add DNS RESOLVE Permission

Add a separate `RESOLVE` ACE:

```sql
BEGIN
    DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
        host => 'api.geoapify.com',
        ace  => XS$ACE_TYPE(
            privilege_list => XS$NAME_LIST('resolve'),
            principal_name => 'APEX_SCHEMA',
            principal_type => XS_ACL.PTYPE_DB
        )
    );
END;
/
```

Expected:

```text
PL/SQL procedure successfully completed.
```

---

## 14. Verify the ACL

In SQL*Plus:

```sql
SET LINESIZE 180
SET PAGESIZE 100

COLUMN HOST       FORMAT A35
COLUMN PRINCIPAL  FORMAT A25
COLUMN PRIVILEGE  FORMAT A12
COLUMN GRANT_TYPE FORMAT A10

SELECT host,
       lower_port,
       upper_port,
       principal,
       privilege,
       grant_type
FROM   dba_host_aces
WHERE  principal = 'APEX_SCHEMA'
ORDER BY host, lower_port, privilege;
```

The relevant entries should look essentially like:

```text
HOST                 LOWER_PORT UPPER_PORT PRINCIPAL     PRIVILEGE GRANT_TYPE
-------------------- ---------- ---------- ------------- --------- ----------
api.geoapify.com            443        443 APEX_SCHEMA   CONNECT   GRANT
api.geoapify.com                            APEX_SCHEMA   RESOLVE   GRANT
```

---

## 15. Do Not Modify Existing APEX ACLs

The existing APEX entries should remain unchanged:


> **Do not remove or modify the existing ACL entries just to add Geoapify access.**

---

## 16. Wallet ACLs Are Not Required for This Basic Setup

The Oracle wallet is being used as a trusted CA store for HTTPS.

The design does **not**:

- store the Geoapify API key in the wallet
- use client-certificate authentication
- require `use-passwords`
- require `use-client-certificates`

The API key and TLS trust are separate concerns.

---

## 17. Test as `APEX_SCHEMA`

This is important.

Testing as SYS does not prove that the application schema has the required network permissions.

Connect as the application schema:

```bash
sqlplus /nolog
SQL> connect APEX_SCHEMA@pdbforapex
```

Or establish a SQL session using the appropriate credentials for `APEX_SCHEMA`.

Enable output:

```sql
SET SERVEROUTPUT ON
```

---

## 18. Test Basic HTTPS Connectivity from PL/SQL

Run:

```sql
DECLARE
    l_req   UTL_HTTP.req;
    l_resp  UTL_HTTP.resp;
BEGIN

    UTL_HTTP.SET_WALLET(
        path     => 'file:/opt/oracle/wallets/geoapify',
        password => NULL
    );

    l_req := UTL_HTTP.BEGIN_REQUEST(
        url    => 'https://api.geoapify.com/',
        method => 'GET'
    );

    UTL_HTTP.SET_HEADER(
        l_req,
        'User-Agent',
        'Oracle-Database-19c'
    );

    l_resp := UTL_HTTP.GET_RESPONSE(l_req);

    DBMS_OUTPUT.PUT_LINE(
        'HTTP STATUS = ' || l_resp.status_code
    );

    UTL_HTTP.END_RESPONSE(l_resp);

EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('SQLCODE = ' || SQLCODE);
        DBMS_OUTPUT.PUT_LINE('SQLERRM = ' || SQLERRM);
        RAISE;
END;
/
```

### Wallet path

Use:

```text
file:/opt/oracle/wallets/geoapify
```

not:

```text
/opt/oracle/wallets/geoapify
```

`UTL_HTTP.SET_WALLET` expects the wallet location in URL form.

---

## 19. Interpret the HTTPS Test

### Successful HTTP response

For example:

```text
HTTP STATUS = 404
```

or another HTTP response code indicates that the connection reached Geoapify.

The important path is:

```text
APEX_SCHEMA
     |
     | ACL
     v
DNS
     |
     | TCP 443
     v
TLS / Wallet
     |
     v
Geoapify
```

A `404`, `401`, or similar HTTP status from the root endpoint is not necessarily an integration failure. It proves that the HTTP request reached the remote service.

### ORA-24247

```text
ORA-24247: network access denied by access control list
```

Likely causes:

- ACL is missing or incorrect
- The request is executing as a different database user
- `APEX_SCHEMA` does not have the required `CONNECT`/`RESOLVE` privileges

### ORA-29024

```text
ORA-29024: Certificate validation failure
```

Likely cause:

- The wallet does not contain the required trusted CA chain
- The wrong CA certificate was imported
- The certificate chain has changed

### ORA-28759

```text
ORA-28759: failure to open file
```

Likely causes:

- Incorrect wallet path
- Incorrect directory permissions
- Missing `cwallet.sso`
- Oracle OS user cannot read the wallet

### Request hangs or times out

Check:

- Linux firewall
- NAT
- proxy configuration
- Internet routing
- outbound TCP 443 access

---

## 20. Test the Geoapify Address Autocomplete API

For address autocomplete, use:

```text
https://api.geoapify.com/v1/geocode/autocomplete
```

The important parameters include:

- `text`
- `limit`
- `format`
- `apiKey`

For a test, use a placeholder API key:

```sql
SET SERVEROUTPUT ON SIZE UNLIMITED
SET DEFINE OFF
```

> `SET DEFINE OFF` is useful in SQL*Plus because `&` has special meaning for substitution variables.

Then run:

```sql
DECLARE
    l_api_key  VARCHAR2(500) := 'YOUR_GEOAPIFY_API_KEY';
    l_url      VARCHAR2(32767);
    l_req      UTL_HTTP.req;
    l_resp     UTL_HTTP.resp;
    l_buffer   VARCHAR2(32767);
BEGIN
    UTL_HTTP.SET_WALLET(
        path     => 'file:/opt/oracle/wallets/geoapify',
        password => NULL
    );

    l_url :=
        'https://api.geoapify.com/v1/geocode/autocomplete'
        || '?text='
        || UTL_URL.ESCAPE(
               url                   => 'Newark, New Jersey, USA',
               escape_reserved_chars => TRUE,
               url_charset            => 'UTF-8'
           )
        || '&limit=5'
        || '&format=json'
        || '&apiKey='
        || UTL_URL.ESCAPE(l_api_key, TRUE, 'UTF-8');

    DBMS_OUTPUT.PUT_LINE('Calling Geoapify...');

    l_req := UTL_HTTP.BEGIN_REQUEST(l_url);

    UTL_HTTP.SET_HEADER(
        l_req,
        'User-Agent',
        'Oracle-APEX'
    );

    l_resp := UTL_HTTP.GET_RESPONSE(l_req);

    DBMS_OUTPUT.PUT_LINE(
        'HTTP STATUS = ' || l_resp.status_code
    );

    BEGIN
        LOOP
            UTL_HTTP.READ_TEXT(
                r    => l_resp,
                data => l_buffer,
                len  => 32767
            );

            DBMS_OUTPUT.PUT_LINE(l_buffer);
        END LOOP;

    EXCEPTION
        WHEN UTL_HTTP.END_OF_BODY THEN
            NULL;
    END;

    UTL_HTTP.END_RESPONSE(l_resp);

EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE(
            'SQLCODE = ' || SQLCODE
        );

        DBMS_OUTPUT.PUT_LINE(
            'SQLERRM = ' || SQLERRM
        );

        BEGIN
            UTL_HTTP.END_RESPONSE(l_resp);
        EXCEPTION
            WHEN OTHERS THEN
                NULL;
        END;

        RAISE;
END;
/
```

A successful request should return JSON containing address suggestions.

---

## 21. Important SQL*Plus Note About `&apiKey`

In SQL*Plus, this:

```text
&apiKey
```

is interpreted as a substitution variable.

If SQL*Plus prompts for a value such as:

```text
Enter value for apikey:
```

disable substitution before running the block:

```sql
SET DEFINE OFF
```

Then use:

```sql
SET SERVEROUTPUT ON SIZE UNLIMITED
SET DEFINE OFF
```

---

## 22. Production URL Construction

The previous autocomplete example is a connectivity test.

Production code should not concatenate raw user input directly into the URL:

```plsql
'?text=' || :P1_ADDRESS
```

Address input can contain:

- spaces
- `&`
- `#`
- `/`
- Unicode characters
- other reserved URL characters

Use `UTL_URL.ESCAPE`.

Example:

```plsql
l_url :=
       'https://api.geoapify.com/v1/geocode/autocomplete'
    || '?text='
    || UTL_URL.ESCAPE(
           url                   => l_search_text,
           escape_reserved_chars => TRUE,
           url_charset           => 'UTF-8'
       )
    || '&limit=5'
    || '&format=json'
    || '&apiKey='
    || UTL_URL.ESCAPE(
           l_api_key,
           TRUE,
           'UTF-8'
       );
```

---

## 23. Do Not Expose the Geoapify API Key

The browser should not know the production API key.

Avoid:

```text
Browser
   |
   +-- Geoapify API key
```

Prefer:

```text
APEX Page
    |
    | search text only
    v
APEX_SCHEMA
    |
    | server-side PL/SQL
    v
Geoapify API
```

The API key should not be placed in:

- JavaScript
- HTML
- APEX page source
- browser-visible AJAX code
- client-side URLs
- GitHub repositories

---

## 24. Recommended Production Package

The APEX application should call a server-side package rather than building the Geoapify request itself.

A simple interface can be:

```sql
CREATE OR REPLACE PACKAGE geoapify_pkg AS

    FUNCTION autocomplete (
        p_text IN VARCHAR2
    ) RETURN CLOB;

END geoapify_pkg;
/
```

The implementation should:

1. Retrieve the API key from a protected server-side configuration.
2. Set the Oracle wallet.
3. URL-encode the search text.
4. Build the Geoapify autocomplete URL.
5. Call Geoapify with `UTL_HTTP`.
6. Read the JSON response.
7. Return the JSON to the APEX application.

The APEX page should only provide the search text to the package.

Conceptually:

```text
APEX_SCHEMA
     |
     +-- protected Geoapify configuration
     |       |
     |       +-- API key
     |
     +-- GEOAPIFY_PKG
             |
             +-- autocomplete()
             |
             +-- geocode()
```

Do not hard-code the API key into every PL/SQL block.

Avoid production code such as:

```plsql
l_api_key := 'abc123...';
```

---

## 25. API Key Restrictions

For a server-side database integration, consider restricting the production Geoapify API key to the stable public outbound IP address used by the Oracle server, if the network architecture permits this.

This provides another layer of protection if the key is accidentally exposed.

Keep in mind that the restriction must match the actual public egress IP seen by Geoapify, not necessarily the Oracle server's private RFC1918 address.

---

## 26. Final Architecture

The completed setup looks like this:

```text
                         Oracle Linux 8
                              |
                              |
                     Oracle Database 19c
                              |
                              |
                       APEX Application 11
                              |
                              v
                         APEX_SCHEMA
                              |
                              |
                           UTL_HTTP
                              |
                              v
                       Network ACL check
                              |
                    +---------+---------+
                    |                   |
                 RESOLVE             CONNECT
             api.geoapify.com    api.geoapify.com:443
                    |                   |
                    +---------+---------+
                              |
                              v
                         HTTPS / TLS
                              |
                              v
                       Oracle Wallet
                              |
             /opt/oracle/wallets/geoapify
                              |
                         Trusted CA(s)
                              |
                              v
                    api.geoapify.com:443
                              |
                              v
                   Geoapify Autocomplete
```

Existing APEX ACLs remain separate.

---

## 27. Complete Implementation Sequence

### Step 1 — Test Linux connectivity

```bash
getent hosts api.geoapify.com
curl -Iv https://api.geoapify.com/
```

### Step 2 — Create wallet directory

```bash
sudo mkdir -p /opt/oracle/wallets/geoapify
sudo chown oracle:oinstall /opt/oracle/wallets/geoapify
sudo chmod 700 /opt/oracle/wallets/geoapify
```

### Step 3 — Create wallet

```bash
/opt/oracle/product/19c/dbhome_1/bin/orapki wallet create \
  -wallet /opt/oracle/wallets/geoapify \
  -auto_login
```

### Step 4 — Inspect Geoapify TLS chain

```bash
openssl s_client \
  -connect api.geoapify.com:443 \
  -servername api.geoapify.com \
  -showcerts </dev/null
```

### Step 5 — Import the required trusted CA certificates

```bash
/opt/oracle/product/19c/dbhome_1/bin/orapki wallet add \
  -wallet /opt/oracle/wallets/geoapify \
  -trusted_cert \
  -cert /path/to/root-ca.pem
```

```bash
/opt/oracle/product/19c/dbhome_1/bin/orapki wallet add \
  -wallet /opt/oracle/wallets/geoapify \
  -trusted_cert \
  -cert /path/to/intermediate-ca.pem
```

### Step 6 — Verify the wallet

```bash
/opt/oracle/product/19c/dbhome_1/bin/orapki wallet display \
  -wallet /opt/oracle/wallets/geoapify
```

### Step 7 — Add `CONNECT` as SYS

```sql
BEGIN
    DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
        host       => 'api.geoapify.com',
        lower_port => 443,
        upper_port => 443,
        ace        => XS$ACE_TYPE(
            privilege_list => XS$NAME_LIST('connect'),
            principal_name => 'APEX_SCHEMA',
            principal_type => XS_ACL.PTYPE_DB
        )
    );
END;
/
```

### Step 8 — Add `RESOLVE`

```sql
BEGIN
    DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
        host => 'api.geoapify.com',
        ace  => XS$ACE_TYPE(
            privilege_list => XS$NAME_LIST('resolve'),
            principal_name => 'APEX_SCHEMA',
            principal_type => XS_ACL.PTYPE_DB
        )
    );
END;
/
```

### Step 9 — Verify the ACL

```sql
SELECT host,
       lower_port,
       upper_port,
       principal,
       privilege,
       grant_type
FROM   dba_host_aces
WHERE  principal = 'APEX_SCHEMA'
ORDER BY host, lower_port, privilege;
```

### Step 10 — Connect as `APEX_SCHEMA`

```bash
sqlplus APEX_SCHEMA
```

### Step 11 — Set the wallet

```plsql
UTL_HTTP.SET_WALLET(
    'file:/opt/oracle/wallets/geoapify',
    NULL
);
```

### Step 12 — Test basic HTTPS

```text
https://api.geoapify.com/
```

### Step 13 — Test autocomplete

```text
https://api.geoapify.com/v1/geocode/autocomplete
```

### Step 14 — Move to the production package

After connectivity and authentication are verified, the APEX developer can implement the server-side `GEOAPIFY_PKG` and protected API-key configuration.

---

## 28. Verification Checklist

Use this checklist before handing the integration to the APEX developer.

- [ ] Linux resolves `api.geoapify.com`
- [ ] Linux can reach TCP 443
- [ ] `/opt/oracle/wallets/geoapify` exists
- [ ] Wallet is owned by the Oracle OS user/group
- [ ] Wallet permissions are restricted
- [ ] `cwallet.sso` exists
- [ ] Required trusted CA certificates are present
- [ ] `APEX_SCHEMA` has `CONNECT` to `api.geoapify.com:443`
- [ ] `APEX_SCHEMA` has `RESOLVE` for `api.geoapify.com`
- [ ] Existing ACL entries were left untouched
- [ ] `UTL_HTTP.SET_WALLET` works as `APEX_SCHEMA`
- [ ] Basic HTTPS request reaches Geoapify
- [ ] Autocomplete endpoint returns HTTP 200 with a valid API key
- [ ] API key is not present in browser-side code
- [ ] API key is not committed to Git
- [ ] Production API key is stored server-side
- [ ] Production API key restrictions have been considered

---

## Notes

The most important production security rule is simple:

> **The Geoapify API key belongs on the server side, not in the browser and not in the Git repository.**
