# Kadalu Lite Volume

Collection of utilities to create and Start a fully functional Kadalu Volume without a Management plane.

Kadalu Lite provides infra to create and use the Kadalu Volume without any overhead of management layers. This also means we loose some of the features provided by the Management layer like listing the Volumes, Checking the status etc. There are other external possible ways through which these Volumes can be monitored, we will cover those aspects soon here.

## Create and use the Kadalu Lite Volume

Install `kadalu-lite` in all the Storage nodes and Client nodes.

```
$ kadalu-lite volume <name> --id=<volume-id> <storage-units>
```

For example,

```
$ kadalu-lite volume metavol --id 1 \
    replica server1.example.com:4201:/exports/metavol/s1 \
            server2.example.com:4201:/exports/metavol/s2 \
            server3.example.com:4201:/exports/metavol/s3
Generated the server and client service files for the metavol. 
- kadalu-lite-metavol-server.service
- kadalu-lite-metavol-client@.service

Copy the server service file to the following nodes.
- server1.example.com
- server2.example.com
- server3.example.com

And start the Kadalu Volume server in all the Storage nodes.

$ systemctl enable kadalu-lite-metavol-server
$ systemctl start kadalu-lite-metavol-server

Start the client service in nodes wherever required.

$ systemctl start kadalu-lite-metavol-client@mnt-metavol

Note: mnt-metavol is the mount point `/mnt/metavol`
```

If the Server or Client service files are not persisted then no worries. Run the above command again to get it.

## Design

Systemd service file includes the following to start the Kadalu Volume server process.

```
[Unit]
Description=Kadalu Lite Volume Server
After=network.target

[Service]
PIDFile=/var/run/kadalu/lite-meta-server.pid
ExecStart=kadalu-lite server --name=meta replica server1.example.com:4201:/exports/metavol/s1 server2.example.com:4201:/exports/metavol/s2 server3.example.com:4201:/exports/metavol/s1 --id=1

[Install]
WantedBy=multi-user.target
```

Kadalu lite takes care of starting the server process and Auto heal process if the Volume belongs to Replica family.

Systemd service file of client will be

```
[Unit]
Description=Kadalu Lite Volume Client %I
After=network.target

[Service]
PIDFile=/var/run/kadalu/%i.pid
ExecStart=kadalu-lite client --name=meta replica server1.example.com:4201:/exports/metavol/s1 server2.example.com:4201:/exports/metavol/s2 server3.example.com:4201:/exports/metavol/s1 --id=1 --mount=%i

[Install]
WantedBy=multi-user.target
```

`kadalu-lite server` and `kadalu-lite client` generates the Volfiles required to use the Kadalu Storage based on the information available in the command line arguments.

## Open issues/TODO
- Documentation to prepare the Storage units
- Volume Options support
- Gfapi support
- NFS Ganesha Support
