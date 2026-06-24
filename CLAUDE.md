# aruba-aos — Agent Operating Guide

> **⛔ NO DIRECT APPLIES TO ANY DEVICE — EVER.**
>
> Direct changes to **any** device — router, firewall, switch, access point, hypervisor, mail gateway, or any other appliance — are **NEVER** permitted, by anyone, for any reason. This bans hand-run `tofu apply`, hand-run `ansible-playbook`, SSH/serial/CLI config writes, REST/API mutations, and web-GUI/console edits.
>
> **Every change MUST flow through the sanctioned pipeline:** declare intent in **prod-netbox** (the single source of truth), then realize it **only** through **prod-semaphore** (the sanctioned runner). A change that did not go **prod-netbox → prod-semaphore** must never reach a device.
>
> **Sole exception:** a specific direct action is permitted *only* when the operator authorizes that exact action in advance by answering an explicit, **alarm-flavored `AskUserQuestion`** — one that names the device, the precise action, and the risk — **in the affirmative**. No standing grants, no inferred permission, no carrying one approval to another action or device. Absent that in-the-moment "yes," the answer is no.
>
> **Never offload the work onto the operator.** When you are blocked, ask for the break-glass authorization that lets *you* do the job — never ask the operator to run a command, SSH in, or make the change on your behalf. The operator grants permission; they do not perform your labour.

Native OpenTofu/Terraform provider for **ArubaOS-Switch (AOS-S)** via REST API
v8. Sibling of `../openwrt-ubus` (same generic-over-the-API philosophy, same
toolchain). The workspace-root `../CLAUDE.md` applies; this adds specifics.

## What this is / isn't

- **Is:** a provider for AOS-S (ProVision-era 2530/2920/2930F, 16.x firmware),
  driven entirely through the documented `/rest/v8` REST API (cookie auth).
- **Isn't:** an ArubaOS-CX provider. CX has `aruba/terraform-provider-aoscx`.
  Do not pull CX concepts (NETCONF, declarative config replace) in here.

## Design tenets

General Go/provider standards: see `/home/jameson/source/ai-prompts/go.md` (§8).

- **The generic resources here are `arubaos_object` (+ data source)** — they
  address any REST path. Resist adding typed resources until there's a real
  ergonomics need (todo 4.1).
- **The subset plan modifier is `subsetMatches`**; `body` is the keys we manage.
- **PUT is idempotent** on AOS-S; create = POST to `create_path` (collection) or
  PUT to `path` (upsert). Singletons (`system`, `stp`, `dns`, `lldp`) use
  `delete_method = "NONE"`.

## Toolchain

General Go/provider standards: see `/home/jameson/source/ai-prompts/go.md` (§7, §10).

- Go 1.26.4 (`/home/jameson/.local/go`), `terraform-plugin-framework` v1.19.0.
- Provider address: `registry.terraform.io/JamesonRGrieve/tofu-aruba-aos`.

## Hard rules

General Go/provider standards: see `/home/jameson/source/ai-prompts/go.md` (§7, §8).

- **No secrets in the repo.** Creds come from the provider config (OpenBao →
  `TF_VAR_*` via Semaphore). The lab switch lives at `192.168.2.210`.
- **The target is a production backbone switch** (OPNsense LAG on Trk3, every
  VLAN). Drive changes via Semaphore.
- Reuse `../openwrt-ubus`'s vetted dependency versions.
