# Shelly Wall Display — Security Posture (Audit Response)

*Scope: current-generation Wall Display hardware (XL, X2i, X1i, D1, U1), Android 11+. First generation legacy devices, Wall Display and Wall Display X2, are in deprecation and excluded from this response.*

## 1. Touch Panel OS Ownership

The Wall Display is a purpose-built appliance, not a general-purpose tablet. The hardware, the Android OS image, and the controlling application are all built and owned by Shelly. The application runs as a **privileged system application** signed with the **platform signing key**, so the application and the operating system share a single chain of trust that only Shelly controls.

**Arbitrary app installation is not possible** on the device — there is no sideloading path and no user-level installation of unverified packages. Where additional applications are available, they are delivered solely through a **curated Shelly app store containing only applications that have been checked, verified, and approved by Shelly**. The end user cannot install software outside this controlled, vetted channel.

On current hardware the application enrols as **Device Owner** and runs in a locked kiosk mode. The end user cannot exit to a general Android launcher or reach system settings outside the curated in-app interface. Ownership of the platform — OS, firmware, and application — therefore rests entirely with Shelly across the full device lifecycle.

## 2. Patching Ability & Cadence

Shelly retains full ability to patch both the application and the OS/firmware layers remotely. The fleet is centrally managed and cloud-connected, so updates can be delivered to in-field devices without end-user interaction.

Shelly follows a **continuous-delivery model rather than a fixed release calendar**: feature and maintenance updates are released as soon as they are ready and validated, rather than being held for scheduled windows. **Security and critical hotfixes are handled out-of-band and mitigated within 3 days of confirmation.** Application and OS/firmware updates are both distributed through Shelly-controlled channels and applied through the platform's standard, integrity-checked update mechanisms.

## 3. ADB / Debug Posture

Production devices ship with the Android **`user` build type**, on which ADB and developer/debug facilities are **disabled by default**. The application itself is not built in a debuggable configuration. Debug access is not available through normal device operation; where it can be enabled at all, it is only via a privileged, non-user-facing procedure reserved for authorised Shelly support and engineering purposes.

## 4. App Access Controls

The device's local control API supports **HTTP Digest authentication using SHA-256**. When enabled, credentials are never stored or transmitted in plaintext — only a derived digest hash is persisted, in application-private storage — and all operations except for the one providing basic device information require authentication, with unauthenticated requests rejected.

Authentication is **not enabled by default**; it is activated when the user sets a device password. To drive secure configuration, both the integrated WebUI and the Shelly Smart Control app **actively and repeatedly prompt the user to enable authentication** until a password is set.

Remote control over Bluetooth is gated once the device is connected to a network: it requires either digest authentication (where enabled) or **explicit user consent on the device**. Prior to network setup, Bluetooth serves as the local provisioning channel used to bring the device online. Cloud connectivity is **outbound-only** and token-authenticated; the device is not designed to accept unsolicited inbound connections from the internet.

## 5. Secure Update Model

All update and provisioning traffic runs over **enforced TLS (minimum TLS 1.2; legacy protocols disabled)** to Shelly-operated endpoints. The **curated app store** additionally applies **certificate pinning** on top of standard certificate-chain validation.

Update authenticity is anchored on **signing-key continuity**. Application packages are rejected by the platform installer unless signed with the established platform/application key. **OS/firmware (OTA) updates are validated through platform-key signing**, which both establishes the integrity of the package and verifies that it originates from Shelly before it is applied. An update therefore cannot be substituted with a differently-signed or tampered artifact.

## 6. Vulnerability Monitoring

The device's practical exposure to operating-system-level vulnerabilities is substantially reduced by design. The attack surface is deliberately minimised: kiosk/Device-Owner lockdown, no third-party or unverified app installation, no general-purpose browser or launcher, debugging disabled by default, outbound-only cloud connectivity, and local control interfaces that support authentication, which users are actively prompted to enable. Combined with central provisioning and fleet-wide remote update capability, this limits the conditions under which an unpatched platform vulnerability could realistically be reached or exploited.

The underlying Android platform is maintained, under Shelly's direction, by Shelly's hardware partner (Smatek), who provides security support and platform patches on request. To complement this, Shelly is **establishing a formal, recurring CVE-monitoring process** covering both the bundled application dependencies and the Android base, so that newly disclosed vulnerabilities affecting the platform are tracked, triaged, and — where they are reachable given the controls above — remediated through the update channels described in this document.
