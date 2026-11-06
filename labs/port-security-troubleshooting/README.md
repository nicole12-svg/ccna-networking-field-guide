# Port Security Violation Logging Troubleshooting Lab

## Overview

This Packet Tracer troubleshooting lab investigates why port-security
violation messages were not appearing on the central Syslog server,
despite port security and remote logging being configured on the
switches.

The issue was not a failure of Syslog itself. The affected access
interfaces had a port-security maximum of **2 MAC addresses**, which
allowed both devices connected through the unauthorized hub. Since the
configured limit was not exceeded, no port-security violation occurred
and there was no violation event to send to the Syslog server.

The fix was to restore the safer default maximum of **1 secure MAC
address per access port**, clear the previously learned sticky MAC
addresses, and then verify that a second device caused a violation. With
the violation mode set to `restrict`, the switch generated a
notification that was forwarded to the Syslog server.

## Problem Statement

An unauthorized hub and an additional device were discovered on the
network. The Security team expected port-security violation records to
be available on the Syslog server, but the server's Syslog log was
empty.

The task was to investigate the cause and resolve the issue, if
possible.

## Topology and Environment

-   Cisco Packet Tracer network.
-   Admin PC used to access network devices.
-   Access switches with port security configured.
-   A hub connecting two end devices to an access switchport.
-   Central Syslog server receiving remote switch logs.
-   Images are stored in the `images/` directory.

## Investigation

### 1. Checked port-security and Syslog configuration

Inspected the switches and confirmed that port security was enabled on
the relevant access interfaces. Remote Syslog configuration was also
present on the switches that were expected to forward logs.

The `show logging` output showed that remote logging was configured and
the trap level was `informational`. The Syslog service on the server was
enabled.

### 2. Verified the Syslog destination

Compared the configured logging host with the actual Syslog server
address and checked IP connectivity. The correct Syslog server address
in this lab was `172.16.10.251`.

During troubleshooting, temporary logging-host changes were made while
checking the addressing. These were reverted, and the switches were
returned to the correct destination. SW5 was also configured to forward
logs remotely because it previously had console logging only.

A successful ping confirmed IP reachability, but it did not by itself
prove that a port-security event had occurred or that a Syslog message
would be generated.

### 3. Examined the port-security maximum

The key discovery came from inspecting the port-security configuration
on the affected interfaces.

The interfaces had:

``` cisco
switchport port-security maximum 2
```

The unauthorized hub connected two devices. Each device presented a
different MAC address, but the configured maximum allowed two secure MAC
addresses on the port.

Therefore, connecting and using both devices did **not** exceed the
configured limit. No violation was triggered, so there was no
port-security violation message to forward to the Syslog server.

This was the root cause: the maximum MAC address setting was too
permissive for the intended one-device-per-access-port policy.

## Resolution

The affected port-security interfaces were reset to the default maximum
of one secure MAC address.

The procedure used was:

1.  Shut down the affected interfaces using an interface range.
2.  Clear the previously learned sticky secure MAC addresses.
3.  Remove the explicit maximum of two MAC addresses.
4.  Bring the interfaces back up.
5.  Verify the port-security status and confirm that each affected port
    had only one secure MAC address permitted.

Representative commands (adjust the interface range to match the lab):

``` cisco
enable
configure terminal
interface range fa0/2 - 5
 shutdown
exit
do clear port-security sticky
interface range fa0/2 - 5
 no switchport port-security maximum 2
 no shutdown
end
```

## Verification and Testing

Because the lab was locked against adding or removing devices, the
existing hub and connected PCs were used to generate the test traffic.

1.  Sent traffic (using `ping`) from the first PC connected through the
    hub. The switch learned that PC's MAC address on the secured access
    port.
2.  Sent traffic from the second PC. Its different MAC address exceeded
    the one-address maximum.
3.  With the violation mode configured as `restrict`, the switch dropped
    traffic from the violating MAC address, incremented the security
    violation counter, and generated a violation notification.
4.  Checked the Syslog server and confirmed that the port-security
    violation messages were now appearing.

Useful verification commands:

``` cisco
show port-security
show port-security interface fa0/x
show port-security address
show logging
```

Replace `fa0/x` with the relevant access interface.

## Result

-   The affected access ports retained port security.
-   The explicit maximum of two secure MAC addresses was removed,
    restoring the default maximum of one.
-   Previously learned sticky MAC addresses were cleared before
    retesting.
-   The second device behind the hub now exceeded the permitted MAC
    address count.
-   The `restrict` violation mode generated a notification.
-   The Syslog server received and displayed the port-security violation
    logs.

The lab was successfully resolved. The absence of violation logs was
caused by the fact that the configured MAC address limit was never
exceeded---not by a failure to generate a violation notification.

## Lessons Learned

-   A configured security feature does not guarantee that a violation
    will occur; the policy values determine what is considered a
    violation.
-   Always inspect the complete interface configuration, especially
    settings such as `switchport port-security maximum`.
-   Two devices behind a hub do not automatically cause a violation if
    the port is configured to allow two secure MAC addresses.
-   `restrict` generates violation notifications when the configured
    limit is exceeded; it cannot log a violation that never happened.
-   Local logging, remote Syslog forwarding, and the Syslog server
    service are separate parts of the logging path and should be
    verified independently.
-   A successful ping verifies IP reachability, not that a specific
    Syslog event has been generated and delivered.
-   Verify the actual server IP and network addressing before changing
    logging destinations.
-   Use existing devices and controlled traffic to test security
    behavior when the lab topology is locked.

## Screenshots

Place the following screenshots in the `images/` directory:

  -----------------------------------------------------------------------
  Image                               Description
  ----------------------------------- -----------------------------------
  `Before-fix.png`                    Syslog server before the fix,
                                      showing no port-security violation
                                      logs.

  `topology.png`                      Packet Tracer network topology.

  `Lab-complete.png`                  Syslog server after the fix,
                                      showing the received violation
                                      logs.
  -----------------------------------------------------------------------
