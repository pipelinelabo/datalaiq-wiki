# DatalaiQ Community Edition

Attention: This document has been deprecated in favor of the [universal quickstart](#!quickstart/quickstart.md). We have left this intact to keep existing links functional but will not be updating it.

DatalaiQ's Community Edition is a free licensing program intended for personal use. Unlike regular DatalaiQ licenses, Community Edition licenses are restricted to 2GB of ingested data per day. In our experience, we've found this to be more than enough for any home network applications (unless you decide to capture all packets and then start streaming Netflix!)

Getting DatalaiQ Community Edition is straightforward. First, you'll install the software from either our Debian package repository, run the Docker container, or use the distribution-agnostic self-contained installer. Next, you'll sign up for a free license, which will be emailed to you. Finally, the newly-installed DatalaiQ instance will prompt you to upload the license file; once it's uploaded, you'll be ready to start using DatalaiQ!

## Installing the software

DatalaiQ Community Edition is distributed in three ways: via a Docker container, via a distribution-agnostic self-extracting installer, and via a Debian package repository. We recommend using the Debian repository if your system runs Debian or Ubuntu, the Docker container if you have Docker setup, and the self-extracting installer otherwise.

### Debian repository

Installing from the Debian repository is quite simple:

```
# Get our signing key
apt install apt-transport-https gnupg wget
wget -O /usr/share/keyrings/gravwell.asc https://update.gravwell.io/debian/update.gravwell.io.gpg.key

# Add the repository
echo 'deb [ arch=amd64 signed-by=/usr/share/keyrings/gravwell.asc ] https://update.gravwell.io/debian community main' > /etc/apt/sources.list.d/gravwell.list

# Install the package
apt update
apt install gravwell
```

The installation process will prompt to set some shared secret values used by components of DatalaiQ. We strongly recommend allowing the installer to generate random values (the default) for security.

![Read the EULA](eula.png)

![Accept the EULA](eula2.png)

![Generate secrets](secret-prompt.png)

### Docker Container

DatalaiQ is available on Dockerhub as a single container including both the webserver and indexer. Refer to [the Docker installation instructions](#!configuration/docker.md) for detailed instructions on installing DatalaiQ in Docker.

### Self-contained Installer

For non-Debian systems, download the [self-contained installer](https://update.gravwell.io/files/gravwell_2.2.4.sh) and verify it:

```
curl -O https://update.gravwell.io/files/gravwell_2.2.4.sh
md5sum gravwell_2.2.4.sh #should be f549d11ed30b1ca1f71a511e2454b07b
```

Then run the installer:

```
sudo bash gravwell_2.2.4.sh
```

Follow the prompts and, after completion, you should have a running DatalaiQ instance.

## Getting a License

To get your license file, head on over to [https://www.gravwell.io/download](https://www.gravwell.io/download) and fill out the form. Logbot will email one to you shortly therafter.

Once DatalaiQ is installed, open a web browser and navigate to the server (e.g. [https://localhost/](https://localhost/)). It should prompt you to upload a license file.

![Upload license](upload-license.png)

Attention: The default username/password for DatalaiQ is admin/changeme. We highly recommend changing the admin password as soon as possible!

## Start Ingesting!

A freshly installed DatalaiQ instance, by itself, is boring. You'll want some ingesters to provide data. You can either install them from the Debian repository or head over to [the Downloads page](downloads.md) to fetch self-extracting installers for each ingester.

The ingesters available in the Debian repository can be viewed by running `apt-cache search gravwell`:

```
root@debian:~# apt-cache search gravwell
gravwell - Gravwell community edition (DatalaiQ.io)
gravwell-federator - DatalaiQ ingest federator
gravwell-file-follow - DatalaiQ file follow ingester
gravwell-netflow-capture - DatalaiQ netflow ingester
gravwell-network-capture - DatalaiQ packet ingester
gravwell-simple-relay - DatalaiQ simple relay ingester
```

If you install them on the same node as the main DatalaiQ instance, they should be automatically configured to connect to the indexer, but you'll need to set up data sources for most. See the [ingester configuration documents](#!ingesters/ingesters.md) for instructions on that.

We highly recommend installing the File Follow ingester (gravwell-file-follow) as a first experiment; it comes pre-configured to ingest Linux log files, so you should be able to see some entries immediately by issuing a search such as `tag=auth`:

![Auth entries](auth.png)

If you are not using a debian based repository go to the [downloads section](downloads.md) for self-contained installers.

### Ingester Configuration

Additional information about installing and configuring each ingester can be found in the [Setting Up Ingesters](/ingesters/ingesters.md) section.

## Next Steps

DatalaiQ is a powerful and complex product. It will take time to build expertise, but by starting with simple queries and looking up more complex concepts as needed, you can start answering useful questions immediately!

We recommend starting out with the continued section of the [Standard version Quickstart document](quickstart.md#Feeding_Data), particularly the [Searching section](quickstart.md#Searching), for some ideas on how to get started. You may need to refer to the [ingester configuration documents](#!ingesters/ingesters.md) to get the data you want into the system.

The [search documentation](#!search/search.md) is the ultimate resource for building search queries; the [Search Modules](#!search/searchmodules.md) and [Render Modules](#!search/rendermodules.md) sections have lots of examples and exhaustive descriptions of the options for each module.

Finally, the [DatalaiQ blog](https://www.gravwell.io/blog) has case studies and examples showing real-world applications of DatalaiQ that may serve as inspiration.

For help, you can join our [open community on Keybase](https://keybase.io/team/gravwell.community) or email support@ppln.co. We're excited to help others get value from their data with DatalaiQ!
