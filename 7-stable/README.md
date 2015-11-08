
These configurations have only been tested with the rsyslog packages as provided with RHEL/CentOS 7.x.

Configuration files
===================
The rsyslog configuration has been split into several seperate files. This makes it easy to just use the components you need for your specific use-case.

*   a global configuration file
*   input
*   output
*   forward

Input
-----
Several input methods are available, just grab the relevant ``.conf`` configuration files and drop them into the ``/etc/rsyslog.d/`` directory.

*   journald - systemd's journal log
*   relp - reliable log transfer
*   syslog / tcp - legacy syslog over tcp
*   syslog / tls - syslog over tls tunnel
*   syslog / udp - legacy syslog over udp
*   uxsocket - unix domain socket (``/dev/log``)


Output
------
Several output methods are available, just grab the relevant ``.conf`` configuration files and drop them into the ``/etc/rsyslog.d/`` directory.

*   file - write log to local files


Forward
-------
Several forwaring methods are available, just grab the relevant ``.conf`` configuration files and drop them into the ``/etc/rsyslog.d/`` directory.

*   syslog / udp - legacy syslog forwarding
*   syslog / tcp - legacy syslog forwarding
*   syslog / tls - syslog over tls tunnel


Use-cases
=========
Different use-cases can be easily implemented by combining several of the above configuration files. This section simply describes several example use-cases, and how to implement them using the provided configuration examples.

| :Use-case description         | :Input                | :Output               | :Forward              |
| ----------------------------- | --------------------- | --------------------- | --------------------- |
| Only local logging            | journald, uxsock      | file                  | n/a                   | 



Setting up TLS
==============
This section describes how to create a Certificate Authority (CA) and server certificate to use with TLS transport methods. To begin with, you need to generate the root CA key, this is what signs all issued certificates.

    openssl genrsa -out rootCA.key 4096


Generate the self-signed root CA certificate, using the previousely generated key.

    openssl req -x509 -new -nodes -key rootCA.key -days 365 -out rootCA.crt


Once you have the root CA certificate generated, you can use that to generate additional SSL certificates for systems and/or services. We can now create the certificate for the rsyslog service on this system, but first we need to create a key.

    openssl genrsa -out host.key 2048


Then we can create a certificate signing request (CSR). Make sure the Common Name (CN) is set to the FQDN, hostname or IP address of the machine you're going to put this on. These values should be used in the rsyslog permittedpeers configuration directive when using x509/name authentication.

    openssl req -new -key host.key -out host.csr


The next step is to take the CSR and generate a signed certificate using the root CA certificate and key you generated previously. Now you have an SSL certificate (in PEM format) called host.crt. This is the certificate you want your rsyslog service to use.

    openssl x509 -req -in host.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out host.crt -days 365


