# Amazon SQS Ingester

The Amazon SQS Ingester (sqsIngester) is a simple ingester that can subscribe to both standard and FIFO SQS queues for ingest. Amazon SQS is a high volume message queue service that supports message delivery guarantees, "soft" ordering of messages, and "at-least-once" delivery of messages. 

For DatalaiQ, "at-least-once" delivery is an important caveat - The SQS ingester may receive duplicate messages with identical timestamps (depending on your configuration). It's also possible that the SQS ingester doesn't see some messages, depending on how your SQS workflow is deployed with other connected services. See [Amazon SQS](https://aws.amazon.com/sqs/) for more information.

## Basic Configuration

The SQS ingester uses the unified global configuration block described in the [ingester section](#!ingesters/ingesters.md#Global_Configuration_Parameters).  Like most other DatalaiQ ingesters, SQS supports multiple upstream indexers, TLS, cleartext, and named pipe connections, a local cache, and local logging.

## Queue Examples

```
[Queue "default"]
	Region="us-east-2"
	Queue-URL="https://us-east-2.amazon..."
	Tag-Name="sqs"
	AKID="AKID..."
	Secret="..."
	Assume-Local-Timezone=false #Default for assume localtime is false
	Source-Override="DEAD::BEEF" #override the source for just this Queue 

[Queue "default"]
	Region="us-west-1"
	Queue-URL="https://us-west-1.amazon..."
	Tag-Name="sqs"
	AKID="AKID..."
	Secret="..."
```

## Installation
If you're using the DatalaiQ Debian repository, installation is just a single apt command:

```
apt-get install gravwell-sqs
```

Otherwise, download the installer from the [Downloads page](#!quickstart/downloads.md). To install the Netflow ingester, simply run the installer as root (the actual file name will typically include a version number):

```
root@gravserver ~ # bash gravwell_sqs.sh
```

If there is no DatalaiQ indexer on the local machine, the installer will prompt for an Ingest-Secret value and an IP address for an indexer (or a Federator). Otherwise, it will pull the appropriate values from the existing DatalaiQ configuration. In any case, review the configuration file in `/opt/gravwell/etc/sqs.conf` after installation. A typical configuration will look like: 

```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Insecure-Skip-TLS-Verify=false
Pipe-Backend-Target=/opt/gravwell/comms/pipe 
Log-Level=INFO
Log-File=/opt/gravwell/log/sqs.log

# A Queue pulls from a specific SQS queue with a given AKID and Secret. See
# https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys
# for information about obtaining an AKID/Secret for your user.
[Queue "default"]
	Region="us-east-2"
	Queue-URL="https://us-east-2.amazon..."
	Tag-Name="sqs"
	AKID="AKID..."
	Secret="..."
```

Note that this configuration sends entries to a local indexer via `/opt/gravwell/comms/pipe`. Entries are tagged 'sqs'.

You can configure any number of `Queue` entries, one for each SQS queue, and provide unique authentication, tag names, etc., for each one.
