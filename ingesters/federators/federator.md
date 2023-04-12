# The DatalaiQ Federator

The Federator is an entry relay: ingesters connect to the Federator and send it entries, then the Federator passes those entries to an indexer.  The Federator can act as a trust boundary, securely relaying entries across network segments without exposing ingest secrets or allowing untrusted nodes to send data for disallowed tags.  The Federator upstream connections are configured like any other ingester, allowing multiplexing, local caching, encryption, etc.

![](federatorDiagram.png)

## IngestListener Examples

```
[IngestListener "enclaveA"]
	Ingest-Secret = CustomSecrets
	TLS-Bind = 0.0.0.0:4024
	TLS-Certfile = /opt/gravwell/etc/cert.pem
	TLS-Keyfile = /opt/gravwell/etc/key.pem
	Tags=windows
	Tags=syslog-*

[IngestListener "enclaveB"]
	Ingest-Secret = OtherIngestSecrets
	Cleartext-Bind = 0.0.0.0:4023
	Tags=apache
	Tags=bash
```


## Use Cases

 * Ingesting data across geographically diverse regions when there may not be robust connectivity
 * Providing an authentication barrier between network segments
 * Reducing the number of connections to an indexer
 * Controlling the tags an data source group can provide

## Installation

If you're using the DatalaiQ Debian repository, installation is just a single apt command:

```
apt-get install gravwell-federator
```

Otherwise, download the installer from the [Downloads page](/quickstart/downloads). Using a terminal on the DatalaiQ server, issue the following command as a superuser (e.g. via the `sudo` command) to install the Federator:

```console
root@gravserver ~ # bash gravwell_federator_installer.sh
```

The Federator will almost certainly require configuration for your specific setup; please refer to the following section for more information. The configuration file can be found at `/opt/gravwell/etc/federator.conf`.

## Example Configuration

The following example configuration connects to two upstream indexers in a *protected* network segment and provides ingest services on two *untrusted* network segments.  Each untrusted ingest point has a unique Ingest-Secret, with one serving TLS with a specific certificate and key pair. The configuration file also enables a local cache, making the Federator act as a fault-tolerant buffer between the DatalaiQ indexers and the untrusted network segments.

```
[Global]
	Ingest-Secret = SuperSecretUpstreamIndexerSecret
	Connection-Timeout = 0
	Insecure-Skip-TLS-Verify = false
	Encrypted-Backend-target=172.20.232.105:4024
	Encrypted-Backend-target=172.20.232.106:4024
	Ingest-Cache-Path=/opt/gravwell/cache/federator.cache
	Max-Ingest-Cache=1024 #1GB
	Log-Level=INFO

[IngestListener "BusinessOps"]
        Ingest-Secret = CustomBusinessSecret
        Cleartext-Bind = 10.0.0.121:4023
        Tags=windows
        Tags=syslog

[IngestListener "DMZ"]
       Ingest-Secret = OtherRandomSecret
       TLS-Bind = 192.168.220.105:4024
       TLS-Certfile = /opt/gravwell/etc/cert.pem
       TLS-Keyfile = /opt/gravwell/etc/key.pem
       Tags=apache
       Tags=nginx
```

Ingesters in the DMZ can connect to the Federator at 192.168.220.105:4024 using TLS encryption. These ingesters are **only** allowed to send entries tagged with the `apache` and `nginx` tags. Ingesters in the business network segment can connect via cleartext to 10.0.0.121:4023 and send entries tagged `windows` and `syslog`. Any mis-tagged entries will be rejected by the Federator; acceptable entries are passed to the two indexers specified in the Global section.

## Troubleshooting

Common configuration errors for the Federator include:

* Incorrect Ingest-Secret in the Global configuration
* Incorrect Backend-Target specification(s)
* Invalid or already-taken Bind specifications
* Enforcing certification validation when upstream indexers or Federators do not have certificates signed by a trusted certificate authority (see the `Insecure-Skip-TLS-Verify` option)
* Mismatched Ingest-Secret for downstream ingesters
