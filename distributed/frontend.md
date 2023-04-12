# Distributed DatalaiQ Webserver

Just as DatalaiQ is designed to have multiple indexers operating at once, it can also have multiple webservers operating at once, pointing to the same set of indexers. Having multiple webservers allows for load balancing and high availability. Even if you only have a single webserver, deploying a datastore can provide useful resiliency, since the datastore can be used to restore a failed webserver or vice versa.

Once configured, distributed webservers will synchronize resources, users, dashboards, user preferences, and user search histories.

## The datastore server

DatalaiQ uses a separate server process called the datastore to keep webservers in sync. It can run on its own machine or it can share a server with a webserver. Fetch the datastore installer from [the downloads page](/quickstart/downloads), then run it on the machine which will contain the datastore.

### Configuring the datastore server

The datastore server should be ready to run without any changes to `gravwell.conf`. To enable the datastore server at boot-time and start the service, run the following commands:

```
systemctl enable gravwell_datastore.service
systemctl start gravwell_datastore.service
```

#### Advanced datastore config

By default, the datastore server will listen on all interfaces over port 9405. If for some reason you need to change this uncomment and set the following line in your `/opt/gravwell/etc/gravwell.conf` file:

```
Datastore-Listen-Address=10.0.0.5	# listen only on 10.0.0.5
Datastore-Port=9555					# listen on port 9555 instead of 9405
```

## Configuring webservers for distributed operation

To tell a webserver to start communicating with a datastore, set the `Datastore` and `External-Addr` fields in the "global" section of the webserver's `/opt/gravwell/etc/gravwell.conf`. For example, if the datastore server was was running on the machine with IP 10.0.0.5 and the default datastore port, and the webserver being configured was running on 10.0.0.1, the entry would look like this:

```
Datastore=10.0.0.5:9405
External-Addr=10.0.0.1:443
```

The `External-Addr` field is the IP address and port that *other webservers* should use to contact this webserver. This allows a user on one webserver to view the results of a search executed on another webserver.

```{note}
By default, the webserver will check in with the datastore every 10 seconds. This can be modified by setting the `Datastore-Update-Interval` field to the desired number of seconds. Be warned that waiting too long between updates will make changes propagate very slowly between webservers, while overly-frequent updates may cause undue system load. 5 to 10 seconds is a good choice.
```

## Disaster recovery

Due to the synchronization techniques used by the datastore and webservers, care must be taken if the datastore server is re-initialized or replaced. Once a webserver has synchronized with a datastore, it considers that datastore the ground truth on all topics; if a resource does not exist on the datastore, but the webserver had previously synchronized that resource with the datastore, the webserver will delete the resource.

The datastore stores data in the following locations:

* `/opt/gravwell/etc/datastore-users.db` (user database)
* `/opt/gravwell/etc/datastore-webstore.db` (dashboards, user preferences, search history)
* `/opt/gravwell/etc/resources/datastore/` (resources)

If any of these locations are accidentally lost or deleted, they should be restored from one of the webserver systems before restarting the datastore. Assuming the datastore is on the same machine as one of the webservers, use the following commands:

```
cp /opt/gravwell/etc/users.db /opt/gravwell/etc/datastore-users.db
cp /opt/gravwell/etc/webstore.db /opt/gravwell/etc/webstore.db
cp -r /opt/gravwell/resources/webserver/* /opt/gravwell/resources/datastore/
```

If the datastore is on a separate machine, use `scp` or another file transfer method to copy those files from a webserver server.

## Load balancing

Gravwell now offers a custom load balancing component specifically designed to distribute users across multiple webservers with minimal configuration. See [the load balancing configuration page](loadbalancer) for information on setting it up.
