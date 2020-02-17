# srslte-docker-emulated

This is a minimal example of an end-to-end [srsLTE] system running with Docker
and shared memory. Core network, base station and user device all run in
separate containers. The air interface is emulated via radio samples in shared
memory.

See it happen with

    docker-compose up

After a while you'll se the UE attach:

    virtual-srsue | Network attach successful. IP: 172.16.0.2
    virtual-srsenb | User 0x46 connected

Now you can test the connection in a new terminal:

    docker exec -i -t virtual-srsepc ping 172.16.0.2
    PING 172.16.0.2 (172.16.0.2) 56(84) bytes of data.
    64 bytes from 172.16.0.2: icmp_seq=1 ttl=64 time=25.3 ms
    64 bytes from 172.16.0.2: icmp_seq=2 ttl=64 time=24.2 ms

*Credits go to [jgiovatto] for implementing the shared memory radio interfaces
and to [FabianEckermann] for figuring out how to integrate it with Docker's IPC
functionality.*

[srsLTE]: https://github.com/srsLTE/srsLTE
[jgiovatto]: https://github.com/jgiovatto
[FabianEckermann]: https://github.com/FabianEckermann

**A note on configuration:** During build, the example config files are copied
into the workdir. These are the files you see used in the compose file with some
option overrides. If you want to play around with the config yourself, it is
much easier to place your custom files in this directory and `ADD` them in the
Dockerfile. You can find the exact versions in [srsepc], [srsenb] and [srsue].

[srsepc]: https://github.com/jgiovatto/srsLTE/tree/5d82f19988bc148d7f4cec7a0f29184375a64b40/srsepc
[srsenb]: https://github.com/jgiovatto/srsLTE/tree/5d82f19988bc148d7f4cec7a0f29184375a64b40/srsenb
[srsue]: https://github.com/jgiovatto/srsLTE/tree/5d82f19988bc148d7f4cec7a0f29184375a64b40/srsue

**Adding UEs:** The compose file contains an optional second UE. It uses the
second IMSI from the default user_db.csv (srsEPC). To add more UEs, add IMSIs to
the csv and tell the UEs to use them.

### Internet access for UEs

By default, containers are attached to a Docker network with a default
route. This means everyone has internet access through the virtualized Docker
network. It takes two extra steps to make UEs access the internet through the
EPC instead. First configure network address translation at the EPC

    docker exec virtual-srsepc iptables -t nat -A POSTROUTING -s 172.16.0.0/24 -o eth0 -j MASQUERADE

This will masquerade all forwarded traffic from UEs (matched by source IP
address) leaving the EPC's eth0 (Docker) interface.

Second, tell the UE to route traffic via the EPC by default

    docker exec virtual-srsue ip route replace default via 172.16.0.1

Now you have network access through the EPC

    docker exec virtual-srsue ping google.com

You can verify that this ping is using the LTE connection by checking whether
it has about 20 ms added latency due to uplink scheduling or by waiting until
the UE enters "RRC IDLE" state, in which your ping command will trigger a
random access and connection setup. The UE enters that state after one minute
of not having sent or received any data through the LTE connection, so make
sure no pings are running.
