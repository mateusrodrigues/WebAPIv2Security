# Module 1 - HTTP Security Primer #

## Creating certificates ##

### Creating a Root Certificate ###

| Command         | Explanation              |
| --------------- | ------------------------ |
| makecert.exe    | The command              |
| -r              | Self-Signed              |
| -n "CN=DevRoot" | Name                     |
| -pe             | Exportable               |
| -sv DevRoot.pvk | Name of private key file |
| -a sha1         | Hashing algorithm        |
| -len 2048       | Key length               |
| -b 01/21/2010   | Valid from               |
| -e 01/21/2030   | Valid to                 |
| -cy authority   | Certificate type         |
| DevRoot.cer     | Name of certificate file |

- The command above creates a Root Certificate.
- Like a Certificate Authority, you can create certificates from root certificates.
- Trusting only the root certificate automatically trusts all certificates created from it.

### Creating an SSL Certificate ###

| Command                | Explanation                   |
| ---------------------- | ----------------------------- |
| makecert.exe           | The command                   |
| -iv DevRoot.pvk        | File name of root private key |
| -ic DevRoot.cer        | File name of root certificate |
| -n "CN=web.local"      | Name                          |
| -pe                    | Exportable                    |
| -sv web.local.pvk      | Name of private key file      |
| -a sha1                | Hashing algorithm             |
| -len 2048              | Key length                    |
| -b 01/21/2010          | Valid from                    |
| -e 01/21/2030          | Valid to                      |
| -sky exchange          | Certificate type              |
| web.local.cert         | Name of certificate file      |
| -eku 1.3.6.1.5.5.7.3.1 | Extended key usage            |

- Creates an SSL certificate from the Root Certificate created before.

## Building an SSL Development Environment ##

The command to create a .pfx file is the following:

```
pvk2pfx.exe -pvk DevRoot.pvk -spc DevRoot.cer -pfx DevRoot.pfx
```

