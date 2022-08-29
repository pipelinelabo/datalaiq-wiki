# Updating Licenses

DatalaiQ licenses are valid for a limited period of time. You will be notified in the DatalaiQ GUI when your license is about to expire. Community Edition users can simply re-enter their information at [https://www.gravwell.io/download](https://www.gravwell.io/download) to get a new license emailed to them; paid customers should contact DatalaiQ support (support@ppln.co) to renew their contract.

Once you have your new license file, simply select the 'License' page from the Administrator section of the DatalaiQ menu and upload your new license:

![](license.png)

You can also deploy your new license by renaming the file to `/opt/gravwell/etc/license` on the webserver node and restarting your DatalaiQ processes. This will push the new license out to the indexers.
