
These configurations have only been tested with the stock rsyslog packages as provided with RHEL/CentOS 7.x. The use of the official distro packages is recommended since they are the most stable in regards to versioning and thus the configuration options. Tracking the upstream repo requires you to test your conffiguration is the upstream versions keeps bumping.

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
Different use-cases can easily be implemented by combining several of the above configuration files. This section simply describes several example use-cases, and how to implement them using the provided configuration examples.

| Use-case | Short description                      | Input                 | Output                | Forward               |
| :------- | :------------------------------------- | :-------------------- | :-------------------- | :-------------------- |
| [1](#1)  | local logging                          | journald, uxsocket    | file                  | n/a                   | 
| [2](#2)  | local logging + traditional forwarding | journald, uxsocket    | file                  | syslog_udp            | 


<a name="1">
1. local logging
----------------
This configuration gathers log from the systemd journal and the traditional syslog socket and writes it's output to local files only. This setup is most common for environments where central logging is not required (eg. your personal laptop).
</a>


<a name="2">
2. local logging + traditional forwarding
-----------------------------------------
Like the first use-case but also sends it's log using the traditional syslog forwarding method (syslog over UDP). Forwarding syslog over UDP is the most unreliable forwarding method available. It's also the most common and best supported forwarding method, if your log server supports better protocols like TCP or TLS use that instead.
</a>


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


