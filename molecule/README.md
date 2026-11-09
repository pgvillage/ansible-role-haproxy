# Molecule tests

Install `ansible-core`, `molecule`, and `molecule-plugins[podman]` on the controller.

`molecule test` runs the Rocky Linux 9 and 10 systemd container tests. These check
service startup, custom frontend and stats ports, backend traffic, and idempotence
without active SELinux. The EL10 container also exercises Ansible's
`Missing selinux Python library` status while SELinux is inactive.

## SELinux enforcing

Use two disposable VMs: openSUSE Leap 16.0 and Rocky Linux 9, with SELinux
enforcing, the distribution's SELinux policy and system Python bindings installed,
Python 3 available as `/usr/bin/python3`, and root SSH access from the controller.
Configure SSH aliases `haproxy-suse16` and `haproxy-el9`, or provide their addresses:

```sh
MOLECULE_SELINUX_SUSE_HOST=192.0.2.10 \
MOLECULE_SELINUX_EL_HOST=192.0.2.11 \
molecule test -s selinux
```

This scenario uses existing hosts and leaves them running. Preparation stops
HAProxy, removes the role's previous policy, resets `haproxy_connect_any`, and
removes the policy compiler. On SUSE it also removes the SELinux Python binding
and checks that Ansible actually reports the missing-library status. Use fresh
VMs for independent test runs; discard them after testing.

Convergence must recover the binding, install the tools and policy, and start
HAProxy. Verification requires SELinux to remain enforcing, all HAProxy
processes to run as `haproxy_t`, and the HAProxy domain to be enforcing too. It
checks the installed module and persistent boolean, then runs the same frontend,
backend, stats page, and stats socket checks as the container scenario.
Idempotence runs before verification. No host policy is shared with a container.

The enforcing scenario fails if the hosts are not enforcing; it does not silently
skip SELinux checks. It is separate from the container CI job because it requires
VMs with their own SELinux policy stores.
