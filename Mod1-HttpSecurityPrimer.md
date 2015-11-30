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

#### The first step is to create a certificate ####

The command to create a .pfx file is the following:

```
pvk2pfx.exe -pvk DevRoot.pvk -spc DevRoot.cer -pfx DevRoot.pfx
```

A .pfx file (personal information exchange file) facilitates the process of
importing and exporting certificates from/to the Windows certificate repository.

#### The second step is to trust the created certificate ####

By only making the root certificate trusted, one can trust all certificates
derived from that.

	- use 'mmc' - Microsoft Manager Console
	- Add the certificates snap-in
	- If a web server, select computer account
	- Click OK

The certificate can be trusted by importing the certificate file (*.cer) into the
*'Trusted Root Certification Authorities\Certificates'* folder.

Then, importing the actual certificate to *Personal\Certificates* does the trick
to make IIS do SSL. **Important**: The certificate imported must be of type ***.pfx**.

Making IIS use the SSL certificate doesn't solve the issue because the browsers don't
know how to use those certificates. To solve that, add the following line to the
C:\Windows\System32\etc\hosts file:

```
127.0.0.1 <certification_name>
```

## Self-hosting Command Line tools ##

```
netsh http add
```

- **urlaclr**: Creating http listeners
- **sslcert**: Creating a binding between the listener and an SSL certificate

## Using .NET APIs to interact with Certificates ##

Using Thinktecture.IdentityModel.Core nuget package:

```csharp
// By default, this API refuses to load untrusted/invalid certificates
var cert = X509.LocalMachine
				.My
				.SubjectDistinguishedName
				.Find("CN=serverdev", false) // Load it even if you think it's not okay
				.First();

// Validation certificates by providing policies
var chain = new X509Chain();
var policy = new X509ChainPolicy
{
	RevocationFlag = X509RevocationFlag.EntireChain,
	RevocationMode = X509RevocationMode.Online,
	UrlRetrievalTimeout = 60,
	VerificationFlags = X509VerificationFlags.IgnoreNotTimeValid
};

chain.ChainPolicy = policy;

if (!chain.Build())
{
	// Something is up with the certificates
	foreach (var element in chain.ChainElements)
	{
		foreach (var status in element.ChainElementStatus)
		{
			Console.WriteLine(status.StatusInformation);
		}
	}
}

var validator = X509CertificateValidator.ChainTrust;
validator.Validate(cert); // Goes up the chain to see if the root certificate is trusted.

validator = X509CertificateValidator.PeerTrust; // Checks trusted peers
validator = X509CertificateValidator.PeerOrChainTrust; // Both
```

## Resources ##

- Thinktecture.IdentityModel
	- http://github.com/thinktecture/Thinktecture.IdentityModel
- HttpConfig
	- http://www.stevestechspot.com/ABetterHttpcfg.aspx
- Netsh documentation
	- MSDN Library link
- Fiddler
	- http://www.telerik.com/download/fiddler