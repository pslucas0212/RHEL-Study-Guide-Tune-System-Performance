# RHEL Study Guide - Tune System Performance

[RHEL Study Guide - Table of Contents](https://github.com/pslucas0212/RHEL-Study-Guide)  

Let's check that the tuned pack is installed
```
# dnf list tuned
Last metadata expiration check: 0:24:30 ago on Mon 08 Jan 2024 05:47:31 PM EST.
Installed Packages
tuned.noarch
```

Let's check that tuned is active
```
# systemctl is-active tuned
active
```

Let's use tune-adm to see what tuneing profile is active
```
# tuned-adm active
Current active profile: virtual-guest
```

Let's see what tuning profiles are available
```
# tuned-adm list
Available profiles:
- accelerator-performance     - Throughput performance based tuning with disabled higher latency STOP states
- balanced                    - General non-specialized tuned profile
- desktop                     - Optimize for the desktop use-case
- hpc-compute                 - Optimize for HPC compute workloads
- intel-sst                   - Configure for Intel Speed Select Base Frequency
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
- optimize-serial-console     - Optimize for serial console use.
- powersave                   - Optimize for low power consumption
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
- virtual-guest               - Optimize for running inside a virtual guest
- virtual-host                - Optimize for running KVM guests
Current active profile: virtual-guest
```

Let's make the balanced profile active
```
# tuned-adm profile balanced
```

Let's use a different option to see what tuning profile is active and the tuning profile purpose
```
# tuned-adm profile_info
Profile name:
balanced

Profile summary:
General non-specialized tuned profile

Profile description:

```

How would we check processes that are consuming the most cpu (percentage cpu)
```
# ps aux --sort=pcpu
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.9 171396 17032 ?        Ss   17:44   0:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 31
root           2  0.0  0.0      0     0 ?        S    17:44   0:00 [kthreadd]
...
root        2077  0.0  0.2 225708  3796 pts/0    R+   18:16   0:00 ps aux --sort=pcpu
root        1980 99.4  0.1 225340  2252 ?        RN   18:08   8:03 sha1sum /dev/zero
root        1996 99.6  0.1 225340  2224 ?        R<   18:08   8:03 md5sum /dev/zero
```

Let's look at the nice for the two most cpu intensive processes. We use $() substitution command to invoke a subshell run the prgrep command.
```
# ps -o pid,pcpu,ni,comm $(pgrep sha1sum;pgrep md5sum)
    PID %CPU  NI COMMAND
   1980 99.3   2 sha1sum
   1996 99.5  -2 md5sum
```

Let's change these two processes priority (renice)
```
# renice -n 10 1980 1996
1980 (process ID) old priority 2, new priority 10
1996 (process ID) old priority -2, new priority 10
```

Check the nice one more time for these two processes
```
# ps -o pid,pcpu,ni,comm $(pgrep sha1sum;pgrep md5sum)
    PID %CPU  NI COMMAND
   1980 99.4  10 sha1sum
   1996 99.6  10 md5sum
```

