rsyslog-configs
===============

This repository contains rsyslog configuration files useful in many logmanagement environments. The goal is to have a set of easily reusable configuration files that are tested, proven and well documented. For these configurations to be really useful in real world environments sufficient attention to issues related to security (CIA triangle), reliability  and scalability has been given.


Contributing
============
Contributing configuration samples and/or use-cases is encouraged and much appreciated! Just fork this repository and send me a pull request.


Versions
========
We try to support all our use-cases / configurations for several supported release branches of rsyslog:

*   7-stable
*   8-stable


7.x stable
----------
These configurations have only been tested with the stock rsyslog packages as provided with RHEL/CentOS 7.x. The use of the official distro packages is recommended since they are the most stable in regards to versioning and thus the configuration options. Tracking the upstream repo requires you to test your configuration as upstream keeps releasing new versions.


8.x stable
----------
These configurations have only been tested with the stock rsyslog packages as provided with RHEL/CentOS 8.x. The use of the official distro packages is recommended since they are the most stable in regards to versioning and thus the configuration options. Tracking the upstream repo requires you to test your configuration as upstream keeps releasing new versions.

Note: also tested on OpenBSD using rsyslog 8.2404.0 - not all features are available on OpenBSD


How does it work?
=================
The rsyslog configuration has been split into several files (snippets), each file providing a single type of functionality. Hopefuly, this model makes it easier to just pick the components you need to implement your specific use-case. Just grab the ``.conf`` files you need and drop them into the appropiate location; the global configuration file ``rsyslog.conf`` should go into ``/etc/`` all other files should go into ``/etc/rsyslog.d/``. Then you need to make sure the proper files are included in the right order (module->input->template->action). Make sure you also verify your configuration before you reload/restart rsyslogd by issuing the following command ``rsyslogd -N2``.

*   rsyslog.conf - the global configuration file
*   input_*.conf - input configuration files (receivers)
*   output_*.conf - output configuration files (actions)
*   forward_*.conf - forwarding configuration files

Snippets
--------

| Ref.      | Type      | Filename                     | Description                            |
| :-------- | :---------| -----------------------------| :--------------------------------------|
| i-uxs     | Input     | input_uxsocket.conf          | Traditional syslog socket              |
| i-sys     | Input     | input_journald.conf          | Systemd's journal                      |
| i-udp     | Input     | input_syslog_udp.conf        | Syslog over UDP                        |
| i-tcp     | Input     | input_syslog_tcp.conf        | Syslog over TCP                        |
| i-tls     | Input     | input_syslog_tls.conf        | Syslog over TLS                        |
| i-relp    | Input     | input_syslog_relp.conf       | Syslog over RELP/TLS                   |
| o-fp      | Output    | output_pipe.conf             | Write log to a named pipe (fifo)       |
| o-fc      | Output    | output_file_client.conf      | Split log based on facility            |
| o-fs      | Output    | output_file_server.conf      | Split log based on date and hostname   |
| f-udp     | Forward   | forward_syslog_udp.conf      | Syslog over UDP                        |
| f-tcp     | Forward   | forward_syslog_tcp.conf      | Syslog over TCP                        |
| f-tls     | Forward   | forward_syslog_tls.conf      | Syslog over TLS                        |
| f-relp    | Forward   | forward_syslog_relp.conf     | Syslog over RELP/TLS                   |


Predefined use-cases
--------------------
This section simply describes several predefined (example) use-cases, and how to implement them using the provided configuration examples.

| Use-case | Short description                      | Input                                       | Output                | Forward               |
| :------- | :------------------------------------- | :-------------------------------------------| :-------------------- | :-------------------- |
| [1]      | local logging                          | i-uxs, i-sys                                | o-fc                  | n/a                   |
| [2]      | local logging + traditional forwarding | i-uxs, i-sys                                | o-fc                  | f-udp                 |
| [3]      | local logging + improved forwarding    | i-uxs, i-sys                                | o-fc                  | f-tcp                 |
| [4]      | local logging + secure forwarding      | i-uxs, i-sys                                | o-fc                  | f-tls                 |
| [5]      | local logging + reliable forwarding    | i-uxs, i-sys                                | o-fc                  | f-relp                |
| [6]      | local logging + named pipe             | i-uxs, i-sys                                | o-fc, o-fp            | n/a                   |
| [7]      | log collector + reliable forwarding    | i-uxs, i-sys, i-udp, i-tcp, i-tls, i-relp   | n/a                   | f-relp                |


Local logging (UC#1)
--------------------
This configuration gathers log from the systemd journal and the traditional syslog socket and writes it's output to local files only. This setup is most common for environments where central logging is not required (eg. your personal laptop).

Local logging + traditional forwarding (UC#2)
---------------------------------------------
Like the first use-case but also sends it's log using the traditional syslog forwarding method (syslog over UDP). Forwarding syslog over UDP is the most unreliable forwarding method available. It's also the most common and best supported forwarding method, if your log server supports better protocols like TCP or TLS use that instead.

Local logging + improved forwarding (UC#3)
------------------------------------------
Like the previous use-case but using TCP as the forwarding protocol. This method has a higher level of reliability because rsyslog can now detect when the destination is unavailable. In this case log messages will be queued and resend when the destination becomes available again. Although a lot better than using UDP; but still there's no 100% guarentee that messages can't be lost.

Local logging + secure forwarding (UC#4)
----------------------------------------
This use-case implements local logging with secure (TLS) forwarding. This method of forwarding is pretty good when dealing with strict requirements regarding confidentiality, integrity and authenticity (CIA) of logs. Unfortunately this method does also suffer from the same limitations as the TCP forwarding use-case, there's still no guarantee that messages can't be lost.

Local logging + reliable forwarding (UC#5)
------------------------------------------
This use-case implements local logging with reliable forwarding (RELP over TLS). This method of forwarding is the most reliable in terms of log delivery (RELP) as well as confidentiality, integrity and authenticity (TLS). This is the recommended configuration when you need to forward logs to a central server.

Local logging + named pipe (UC#6)
---------------------------------
This configuration gathers log from the systemd journal and the traditional syslog socket and writes it's output to local files and also to a named pipe (fifo). This configuration can be used when you need to send your local log to another process on the system, for instance a SIEM agent/collector. This is not a very reliable method for transfering log to another process.

Log collector + reliable forwarding (UC#7)
------------------------------------------
This use-case implements a log collector server which can receive syslog on UDP, TCP, TLS and RELP. Additionally it forwards all log to a central log server using RELP over TLS.

Setting up TLS
==============
When you need some of the TLS features of rsyslog please follow these instructions to create the required certificates.

Setup a CA
----------
This section describes how to create a Certificate Authority (CA) and server certificate to use with TLS transport methods. To begin with, you need to generate the root CA key, this is what signs all issued certificates.

    openssl genrsa -out rootCA.key 4096


Generate the self-signed root CA certificate, using the previousely generated key.

    openssl req -x509 -new -nodes -key rootCA.key -days 365 -out rootCA.crt

Setup host certificate
----------------------
Once you have the root CA certificate generated, you can use that to generate additional SSL certificates for systems and/or services. We can now create the certificate for the rsyslog service on this system, but first we need to create a key.

    openssl genrsa -out host.key 2048


Then we can create a certificate signing request (CSR). Make sure the Common Name (CN) is set to the FQDN, hostname or IP address of the machine you're going to put this on. These values should be used in the rsyslog permittedpeers configuration directive when using x509/name authentication.

    openssl req -new -key host.key -out host.csr


The next step is to take the CSR and generate a signed certificate using the root CA certificate and key you generated previously. Now you have an SSL certificate (in PEM format) called host.crt. This is the certificate you want your rsyslog service to use.

    openssl x509 -req -in host.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out host.crt -days 365

rsyslog TLS configuration
-------------------------
Now that you have the required certificates, you need to edit the global configuration file (``/etc/rsyslog.conf``) to enable the TLS settings and adjust for the names & locations of your certificates.
