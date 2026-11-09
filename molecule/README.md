# Molecule tests

Install `ansible-core`, `molecule`, and `molecule-plugins[podman]` on the controller.

`molecule test` runs the Rocky Linux 9 and 10 systemd container tests. These check
service startup, custom frontend and stats ports, backend traffic, and idempotence
without active SELinux. The EL10 container also exercises Ansible's
`Missing selinux Python library` status while SELinux is inactive.

Convergence must recover the binding, install the tools and policy, and start
HAProxy. Verification requires SELinux to remain enforcing, all HAProxy
processes to run as `haproxy_t`, and the HAProxy domain to be enforcing too. It
checks the installed module and persistent boolean, then runs the same frontend,
backend, stats page, and stats socket checks as the container scenario.
Idempotence runs before verification. No host policy is shared with a container.