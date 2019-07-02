Ghostunnel
======================

Ghostunnel is a SSL/TLS proxy with mutual authentication. 

See https://github.com/square/ghostunnel


The HUB leverages the proxy to provide remote access to backend services like database servers (eg Postgres, MySQL).
In order to ensure the security of data and processes, it is mandatory to limit access only to authorized users.
Furthermore, it is advisable to expose only the *resource* (ie the database) to clients, and avoid 
the usage of network tunnels like VPN, SSH port-forward etc which expose the whole network to unsecured client devices.

The adopted solution leverages a couple of *ghostunnel* proxies, configured with mutual authentication:

- the **server-side** proxy is exposed on the internet,
  possesses a long-duration certificate and accepts connections only from clients which present a valid certificate.
  This proxy exposes an external TCP port which is binded to the internal database server.

- the **client-side** proxy verifies the server certificate, presents a *short-lived* certificate with a valid 
  identity (gathered from OIDC - see *smallstep ca*) and after the successful handshake will expose a local port (on the client)
  which exposes the server-side database as local

This schema ensures that 

- clients can access cloud databases with any tool, for any kind of operation, even for days-long backup/restores
- database servers are never exposed on the public network
- communication is encrypted via TLS, without dependencies on external softwares, libraries etc
- connection requests are validated against a real identity, provided by the identity provider (AAC)
- only real users with a valid *role* will be able to obtain a temporary certificate, independently from database credentials
- certificates are provisioned and invalidated programmately, without operator access
- certificates can have a short duration, from minutes to less than a day, an approach which could avoid the need for CRLs

1. Server proxy
------------------


The **server-side** proxy is run via docker or kubernetes with the following parameters:

- the address (host+port) of the internal database server
- the address (host+port) of the external public mapping
- the root certificate for the CA
- the server certificate and private key for mutual authentication
- a *policy* which checks for additional properties on valid certificate (eg groups, cn, san etc).

Example execution for exposing a local MySQL on 3306 on the local port 8306 with TLS.

```
docker run --rm squareup/ghostunnel server --listen localhost:8306 --target localhost:3306 --cert ghost.crt --key ghost.key --cacert root_ca.crt --allow-all
```

2. Client proxy
------------------


The **client-side** proxy is executed from the binary and requires the same parameters as the server side, but obviously in an inverted fashion.

- the address (host+port) of the cloud database server
- the address (host+port) of the local port mapping
- the root certificate for the CA
- the client certificate and private key for mutual authentication
- a *policy* which checks for additional properties on valid certificate (eg groups, cn, san etc).

Example execution for a cloud Postgres, with a combined keystore (cert+key) and the override of the server identity.

```
$ ~/ghostunnel-v1.4.1-linux-amd64-with-pkcs11 client --listen localhost:5432 --target 13.80.78.44:8543 --cacert root_ca.crt --keystore test.pem --override-server-name ghostunnel
```

Do note the ``--override-server-name`` flag which is required if the server identity (CN) does not match the DNS.
In this case the provided IP can not match the identity ``CN=ghostunnel``, and the certificate has not SAN fields.



