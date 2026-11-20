---
layout: default
title: cf-monitord
sorting: 50
keywords: [monitor]
aliases:
  - "/reference-components-cf-monitord.html"
---

`cf-monitord` is the monitoring daemon for CFEngine. It samples probes defined
in policy using `measurements` type promises and attempts to learn the normal
system state based on current and past observations. Current estimates are made
available as [special variables][special variables] (e.g.
[`$(mon.av_cpu)`][mon#mon.av_cpu]) to `cf-agent`, which may use them to inform
policy decisions.

`cf-monitord` keeps the promises made in `common` and `monitor` bundles and is
affected by `common` and `monitor` control bodies.

**Notes:**

- `cf-monitord` always considers the class `monitor` to be defined.

## Command reference

{{< CFEngine_include_snippet(cf-monitord.help, [\s]*--[a-z], ^$) >}}

## Standard measurements:

The `cf-monitord` service monitors a number of variables as standard on Unix
and Windows systems. Windows is fundamentally different from Unix and
currently has less support for out-of-the-box probes.

1.  users: Users logged in
2.  rootprocs: Privileged system processes
3.  otherprocs: Non-privileged process
4.  diskfree: Free disk on / partition
5.  loadavg: % kernel load utilization
6.  netbiosns_in: netbios name lookups (in)
7.  netbiosns_out: netbios name lookups (out)
8.  netbiosdgm_in: netbios name datagrams (in)
9.  netbiosdgm_out: netbios name datagrams (out)
10. netbiosssn_in: netbios name sessions (in)
11. netbiosssn_out: netbios name sessions (out)
12. irc_in: IRC connections (in)
13. irc_out: IRC connections (out)
14. cfengine_in: CFEngine connections (in)
15. cfengine_out: CFEngine connections (out)
16. nfsd_in: nfs connections (in)
17. nfsd_out: nfs connections (out)
18. smtp_in: smtp connections (in)
19. smtp_out: smtp connections (out)
20. www_in: www connections (in)
21. www_out: www connections (out)
22. ftp_in: ftp connections (in)
23. ftp_out: ftp connections (out)
24. ssh_in: ssh connections (in)
25. ssh_out: ssh connections (out)
26. wwws_in: wwws connections (in)
27. wwws_out: wwws connections (out)
28. icmp_in: ICMP packets (in)
29. icmp_out: ICMP packets (out)
30. udp_in: UDP dgrams (in)
31. udp_out: UDP dgrams (out)
32. dns_in: DNS requests (in)
33. dns_out: DNS requests (out)
34. tcpsyn_in: TCP sessions (in)
35. tcpsyn_out: TCP sessions (out)
36. tcpack_in: TCP acks (in)
37. tcpack_out: TCP acks (out)
38. tcpfin_in: TCP finish (in)
39. tcpfin_out: TCP finish (out)
40. tcpmisc_in: TCP misc (in)
41. tcpmisc_out: TCP misc (out)
42. webaccess: Webserver hits
43. weberrors: Webserver errors
44. syslog: New log entries (Syslog)
45. messages: New log entries (messages)
46. temp0: CPU Temperature core 0
47. temp1: CPU Temperature core 1
48. temp2: CPU Temperature core 2
49. temp3: CPU Temperature core 3
50. cpu: %CPU utilization (all)
51. cpu0: %CPU utilization core 0
52. cpu1: %CPU utilization core 1
53. cpu2: %CPU utilization core 2
54. cpu3: %CPU utilization core 3
55. microsoft_ds_out: Samba/MS_ds name sessions (out)
56. www_alt_in: Alternative web service connections (in)
57. www_alt_out: Alternative web client connections (out)
58. imaps_in: encrypted imap mail service sessions (in)
59. imaps_out: encrypted imap mail client sessions (out)
60. ldap_in: LDAP directory service service sessions (in)
61. ldap_out: LDAP directory service client sessions (out)
62. ldaps_in: LDAP directory service service sessions (in)
63. ldaps_out: LDAP directory service client sessions (out)
64. mongo_in: Mongo database service sessions (in)
65. mongo_out: Mongo database client sessions (out)
66. mysql_in: MySQL database service sessions (in)
67. mysql_out: MySQL database client sessions (out)
68. postgres_in: PostgreSQL database service sessions (in)
69. postgres_out: PostgreSQL database client sessions (out)
70. ipp_in: Internet Printer Protocol (in)
71. ipp_out: Internet Printer Protocol (out)
72. io_reads: Number of I/O reads
73. io_writes: Number of I/O writes
74. io_readdata: Aggregate mount of data read across all devices
75. io_writtendata: Aggregate amount of data written across all devices
76. mem_total: Total system memory
77. mem_free: Free system memory
78. mem_cached: Size of disk cache
79. mem_swap: Total swap size
80. mem_freeswap: Free swap size

Slots with a higher number are used for custom measurement promises in
CFEngine Enterprise.

These values collected and analyzed by `cf-monitord` are transformed
into agent variables in the `$(mon.name)` context.

**Note:** There is no way for force a refresh of the monitored data.

## Data storage

`cf-monitord` records data in `$(sys.statedir)` (typically `/var/cfengine/state`).

- `cf_observations.lmdb`
- `nova_measures.lmdb`
- `ts_key`
- `env_data`
- `cf_incoming.<service id>`
- `cf_outgoing.<service id>`
- `cf_state.lmdb`
- `history.lmdb`

## Statistical classes

`cf-monitord` automatically defines classes based on the data it has collected.
Classes are named for the measurement id (the promise handle in the case of
custom measurement promises), followed by a direction suffix and, for most
samples, a level suffix.

Each sample is compared with two baselines: the expected value for the same
time of the week (learned over previous weeks) and a recent, rolling average.

The direction suffix says where the sample is relative to both baselines:

- `_high` :: The last measurement was above both the weekly expectation and the
  recent average.
- `_low` :: The last measurement was below both the weekly expectation and the
  recent average.
- `_normal` :: The two baselines disagree: the measurement was above one and
  below the other. This is typical just after a spike, when the measurement is
  below the recent average again but still above the weekly expectation, and
  while the weekly expectations are still being learned.

The level suffix follows the direction suffix. It is based on the combined
deviation from both baselines, each measured in its own standard deviations, so
the steps are multiples of √2:

- `_normal` :: The combined deviation is at most √2 (about 1.4).
- `_dev1` :: The combined deviation is more than √2.
- `_dev2` :: The combined deviation is more than 2√2 (about 2.8). `_dev1` is
  also defined.
- `_anomaly` :: The combined deviation is more than 3√2 (about 4.2). `_dev1`
  and `_dev2` are also defined.
- `_microanomaly` :: The measurement is less than 6 away from the weekly
  expectation (see below), but the combined deviation is more than 2√2.

Independently of the level, `cf-monitord` looks for a leap (step) in the last 10
samples:

- `_ldt` :: A leap was detected, meaning a distinct change in the average. Only
  `_high_ldt` and `_low_ldt` are defined, relative to the weekly expectation.

Classes with `_dev2`, `_anomaly`, `_microanomaly` and `_ldt` persist for 10
minutes, so agent runs on a normal schedule see them. Classes with `_normal` and
`_dev1` describe the current sample only.

Two rules decide whether any of these classes are defined:

- If either baseline has a standard deviation of 0, no classes are defined for
  that measurement. A measurement that has always had the same value, for
  example 0, never produces statistical classes.
- A measurement less than 6 away from the weekly expectation, in the unit of
  the measurement, never produces `_dev1`, `_dev2` or `_anomaly`. For such a
  measurement only the direction class (for example `rootprocs_high`) and
  possibly `_microanomaly` are defined. For custom measurements, choose a unit
  where a difference worth noticing is larger than 6.

The following prefixes may be used when defining classes:

- `entropy_` ::

Examples of classes: `rootprocs_high_dev2`, `cpu1_high_anomaly`,
`loadavg_high_ldt`, `syslog_normal_dev1`, `ssh_in_high_microanomaly` and
`entropy_postgresql_out_low`.

The values behind these classes are available as variables in the
[`mon` context](../../special-variables/mon/): `mon.value_<id>` is the last
measurement, `mon.av_<id>` the weekly expectation and `mon.dev_<id>` the
combined standard deviation of both baselines.

## Control promises

Settings describing the details of the fixed behavioral promises
made by `cf-monitord`. The system defaults will be sufficient for
most users. This configurability potential, however, will be a key
to developing the integrated monitoring capabilities of CFEngine.

```cf3
body monitor control
{
  #version => "1.2.3.4";
  forgetrate => "0.7";
  tcpdump => "false";
  tcpdumpcommand => "/usr/sbin/tcpdump -i eth1 -n -t -v";
}
```

### forgetrate

**Description:** Decimal fraction [0,1] weighting of new values over
old in 2d-average computation

Configurable settings for the machine-learning algorithm that
tracks system behavior. This is only for expert users. This
parameter effectively determines (together with the monitoring
rate) how quickly CFEngine forgets its previous history.

**Type:** `real`

**Allowed input range:** `0,1`

**Default value:** 0.6

**Example:**

```cf3
body monitor control
{
  forgetrate => "0.7";
}
```

**See also:** [`default:control_monitor.forgetrate`](/reference/special-variables/control_monitor/#defaultcontrol_monitorforgetrate)

**History:** Added in CFEngine 3.0.0

### histograms

**Deprecated:** Ignored, kept for backward compatibility

`cf-monitord` now always keeps histograms information, so this
option is a no-op kept for backward compatibility. It used to cause
CFEngine to learn the conformally transformed distributions of
fluctuations about the mean.

**Type:** [`boolean`][boolean]

**Default value:** true

**Example:**

```cf3
body monitor control
{
  histograms => "true";
}
```

**See also:** [`default:control_monitor.histograms`](/reference/special-variables/control_monitor/#defaultcontrol_monitorhistograms)

### monitorfacility

**Description:** Menu option for syslog facility

**Type:** (menu option)

**Allowed input range:**

    LOG_USER
    LOG_DAEMON
    LOG_LOCAL0
    LOG_LOCAL1
    LOG_LOCAL2
    LOG_LOCAL3
    LOG_LOCAL4
    LOG_LOCAL5
    LOG_LOCAL6
    LOG_LOCAL7

**Default value:** `LOG_USER`

**Example:**

    body monitor control
    {
    monitorfacility => "LOG_USER";
    }

**See also:** [`default:control_monitor.monitorfacility`](/reference/special-variables/control_monitor/#defaultcontrol_monitormonitorfacility)

### tcpdump

**Description:** true/false use tcpdump if found

Interface with TCP stream if possible.

**Type:** [`boolean`][boolean]

**Default value:** false

    body monitor control
    {
    tcpdump => "true";
    }

**See also:** [`default:control_monitor.tcpdump`](/reference/special-variables/control_monitor/#defaultcontrol_monitortcpdump)

**History:** Added in CFEngine 3.0.0

### tcpdumpcommand

**Description:** Path to the tcpdump command on this system

If this is defined, the monitor will try to interface with the TCP
stream and monitor generic package categories for anomalies.

**Type:** `string`

**Allowed input range:** `"?(/.*)`

**Example:**

```cf3
body monitor control
{
  tcpdumpcommand => "/usr/sbin/tcpdump -i eth1";
}
```

**See also:** [`default:control_monitor.tcpdumpcommand`](/reference/special-variables/control_monitor/#defaultcontrol_monitortcpdumpcommand)

**History:** Added in CFEngine 3.0.0
