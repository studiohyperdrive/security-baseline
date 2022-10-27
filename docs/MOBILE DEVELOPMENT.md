# Mobile app development

Security issues are no different in the case of mobile applications—wherein the application is downloaded from the Internet (e.g., Apple Store or Google Play Store) and installed in the user’s device. Similar to web applications that are exposed on the Internet, mobile applications that are installed in BYOD devices are entry points to the enterprise network.

Installed mobile applications, if not protected appropriately, can be reverse engineered to get their source code, which is in human-readable form. Platforms such as iOS and Android—the two most popular mobile platforms today—are not immune to the threat of reverse engineering. A few easy steps and widely available (often free) tools make it easy for an attacker to:

* Extract the installed application from the mobile device
* Analyze or reverse engineer the code to find vital information, e.g., business logic, the application programming interface (API) used and embedded internal URLs
* Modify the code to change application behavior
* Inject malicious code

## Table of Contents

<details>
    <summary>Toggle</summary>

* [Technical specifications](./SECURITY_BASELINE.md)
* [Authentication](./AUTHENTICATION.md)
* [Mobile App Development](#mobile-app-development)
    * [Best practices](#best-practices)

</details>

## Best practices

### Do not persist sensitive data

Avoid persisting sensitive data to the device, to avoid memory scraping and other reverse engineering risks. Instead, prefer retrieval of private data on-demand, thus reducing the lifetime of the sensitive information on the device. To take this a step further and protect the in-memory data, you can leverage libraries like SecureString, Obfuscator, or Android NDK to encode your strings and enforce that the data can only be accessed when the device is unlocked using biometric authentication.

If this approach is not possible or user experience dictates some sort of secure persistence, our recommendation would be to encrypt the sensitive data using a tool like Facebook Conceal and store the decryption key in the operating system’s secure credential store.

### Omit sensitive data from logging

Disable any local logging commands that display sensitive data during a release build as these entries are easily retrieved using a binary file reader after a device sync. Custom app telemetry logging via App Center / Firebase / Crashlytics that includes sensitive data should never be allowed because you’re essentially handing that information over to a third party.

### Obfuscate source code

When working cross-platform, (depending on the platform) the JavaScript code is likely not obfuscated in any way, which means the application’s code can easily be extracted from the installed .ipa or .apk file.

Obfuscating the source code is not seen as a substantial security measure, but will make reverse engineering the code far more difficult.

### Secure all communication

Ensure that all connections between the mobile applications and the back-end servers are performed using SSL. SSL certificate check is enforced by the client application to safeguard against interception and Man-in-the-middle (MitM) attacks.

### Third-party inventory

It is recommended to create an inventory of open-source and third-party libraries that are used in the application that is being developed and retain the inventory as part of the development artifacts. Because open source comes from multiple parties and is introduced in the application code by developers from in-house and/or outsourced partners, it is essential that the inventory tracks the open-source component in the code and determines if these components are affected by known vulnerabilities.

A benefit of open-source inventory is that when any security incident takes place involving these libraries, remediation can be very quick, especially when the enterprise has several applications in its portfolio. A lack of information on open-source components that are used in applications can make it difficult to initiate remediation activities.
