# srslte-docker-emulated

This is a minimal example of an end-to-end [srsLTE] system running with Docker
and shared memory. Core network, base station and user device all run in
separate containers. The air interface is emulated via radio samples in shared
memory.

See it happen with

    docker-compose up

After a while you'll se the UE attach:

    srsue1_1  | Network attach successful. IP: 172.16.0.2
    shmem-srsenb | User 0x46 connected

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
Dockerfile.
