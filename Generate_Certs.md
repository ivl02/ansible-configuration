# Generating a TLS Certificate

For the demo purposes, self-signed certificate will be used, so the first step is to manually generate a TLS certificate.

> **Tip**: *In production environment, a TLS certificate should be bought from a CA (Certificate Authority) or use free service such as Let's Encrypt.*

Create a files subdirectory of playbooks directory, and then generate the TLS certificate and key:
```sh
$ mkdir files
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -subj /CN=localhost \
    -keyout files/nginx.key -out files/nginx.crt
```

Output should look like:

```sh
Generating a 2048 bit RSA private key
...............................................+++
.+++
writing new private key to 'nginx.key'
-----
```

> **Note**: *If above command is executed from **files** directory then parameters **-keyout** and **-out** should look like: `-keyout nginx.key` and `-out nginx.crt`.*

Above commands will generate the **nginx.key** and **nginx.crt** in the **files** directory. The certificate has an expiration date of 1 year (365 days) from the day created.