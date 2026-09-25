# Build Environment

The Terraform and AWS CLI work in this repo runs from a dedicated Linux VM on a self-hosted hypervisor (Proxmox VE) rather than directly from a laptop — a small piece of real infrastructure standing in for what would otherwise be a CI runner or bastion host in a team setting.

## Provisioning approach

The VM was provisioned from the official Ubuntu Server 24.04 **cloud image**, not an interactive ISO install, configured declaratively via a custom **cloud-init** user-data file rather than through Proxmox's built-in Cloud-Init GUI fields.

Why this matters: this is the same mechanism AWS EC2, GCP, and Azure all use to bootstrap VM instances — a golden image plus a config payload applied at first boot, rather than a one-time manual setup. Building the lab VM this way means the config is version-controlled and reproducible (see the sanitized snippet below), and the workflow transfers directly to writing `user_data` for EC2 instances later in this project.

## Security posture

- **Key-only SSH access** — the cloud image ships with no valid password login by default; a public key is injected via cloud-init and password authentication stays disabled. This mirrors how AWS EC2 key pairs work and was a deliberate choice, not a default left in place.
- **Non-root sudo user** — day-to-day access is through a dedicated non-root user with `sudo` rather than direct root login.
- **Network isolation** — the VM sits on an isolated, LAN-only segment with a fixed (non-DHCP) address, not exposed beyond the local network.

## Tooling

- **Terraform**, installed from HashiCorp's official apt repository (not a manually-downloaded binary) so it stays managed and upgradable like any other package.
- **AWS CLI v2**, installed via AWS's official bundled installer — Ubuntu's apt repos either omit the AWS CLI or carry a stale v1 build, so the vendor-provided installer is the canonical method here.

## Reference: cloud-init configuration

A sanitized copy of the cloud-init user-data used to provision this VM is included at [`cloud-init/tf-ops-user-data.example.yaml`](./cloud-init/tf-ops-user-data.example.yaml) — real SSH key material and hostnames specific to the lab environment have been replaced with placeholders.
