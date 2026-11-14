# Molecule tests

Install `ansible-core`, `molecule`, and `molecule-plugins[podman]` on the controller.

`molecule test` runs the Rocky Linux 9 and 10 systemd container tests. These check
service startup, custom frontend and stats ports, backend traffic, and idempotence
without active SELinux. The EL10 container also exercises Ansible's
`Missing selinux Python library` status while SELinux is inactive.

The verifier runs the frontend, backend, stats page and stats socket checks, and
checks that no custom policy is deployed while SELinux is inactive. Containers
can't run SELinux enforcing, so the policy installation and HAProxy under an
enforcing policy (`haproxy_t`, the module, the persistent boolean) are not
tested by Molecule.
