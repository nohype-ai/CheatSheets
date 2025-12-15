# Code Signing Cheat Sheet

## Basic Motivation

Code Signing has several related purposes:

1. **Control (The "Walled Garden")**: It ensures Apple remains the sole gatekeeper of software distribution. By requiring code signatures, only apps from the App Store or approved enterprise channels can run.
2. **Quality**: Acts as the technical enforcement mechanism for App Store policies. Since apps must be signed to run on devices, they are forced to pass Apple's quality review standards (App Store app review).
3. **Safety**: Guarantees the app code hasn't been altered since it was signed. Protects against any tampering with an app, be it on the developer machine after compilation, during distribution (at Apple), or on the user's device after installation.
4. **Accountability**: Links the app to a real-world entity (the developer team). If an app is malicious, Apple can revoke that team's certificate, effectively stopping the app from running on all devices.
5. **Permissions**: Acts as the gatekeeper for system features by binding "entitlements" to the binary. Sensitive capabilities (iCloud, Apple Pay, Push Notifications) or permissions (Camera, Microphone, TCC) are strictly tied to the code signature. If the signature is invalid or changes, these permissions are reset or denied.

## Overview

For illustration, we look at an idealized general scenario in which one team of developers in the context of one company develops multiple apps for multiple platforms (iOS, macOS), and each developer might work on more than one of those apps:

![](code-signing/Code_Signing_Fuckery.jpg)

## Essentials

* Developer Account
  * Identified by a team ID, representing the entire development team within the company.
  * All the other concepts, including developers and apps, are associated with this one team ID.
* Certificates (Signing Identities)
  * Only two certificates are needed: Development and Distribution. These are used to sign **all apps** on **all machines**. Install them, along with their private keys, on each developer's machine.
  * Universal "**Apple** Development" and "**Apple** Distribution" certificates cover both macOS and iOS.
  * A developer can generate a certificate on their machine, storing it locally in Keychain Access. The certificate, along with its private key, must be exported and securely shared with the team. A private git repo is generally not considered secure enough for this purpose. All this applies even to the development certificate.
  * Certificates are typically installed in the "Login" keychain, but using "iCloud" could be useful for a developer working across multiple machines.
* Profiles
  * One profile per app (App ID), platform (iOS, macOS ...), and build type (development / distribution) is required.
    * Example: MyApp macOS Development Profile
    * **Note**: Even with the universal "Apple Development" / "Apple Distribution" certificates and an App ID shared between iOS and macOS, you strictly need **separate provisioning profiles** for each platform. For example because many entitlements and capabilities are platform-specific. Also because the actual built binaries and resulting app bundles are platform-specific.
  * Each profile uses one of the two certificates (development / distribution) according to its intended build type.
  * Development profiles can also be associated with specific test devices (see below).
  * When creating a distribution profile, the unspecific option "App Store" refers to the iOS App Store.
  * Developers' test devices (or development machines for macOS) must be added to the respective development profile. When a device is added, developers are not required to install the updated profile, as long as they don't use that newly registered device.
  * The same signing identity (certificate) may be referenced by multiple profiles. For example, device tests might use a dedicated profile that contains the corresponding test devices.
* Test Devices
  * **Mandatory on Device**: You must sign code to run it on a physical device. Even simple debug builds must be signed. For that purpose, the device's UDID **must** be registered and included in the development provisioning profile.
  * **macOS**:
    * **Ad-Hoc (Local)**: No explicit signing or UDID registration required. "Sign to Run Locally" works for basic debugging on your own machine.
    * **Fully Signed**: If you use a real "Apple Development" certificate (e.g., to test **iCloud**, **Push Notifications**, or other restricted entitlements), you **must** register the Mac's **UDID** (not the UUID!) in the developer portal and include it in the profile.
  * The Simulator (iOS/iPadOS/watchOS/tvOS/visionOS) doesn't enforce code signing.

## Tips

* Ensure there are no expired or unused/unintended profiles and certificates installed, as these can confuse Xcode:

    * Keychain Access app
	* System settings -> Privacy & Security -> Profiles -> Provisioning
	
* Cleaning the build, restarting Xcode, or even rebooting the Mac sometimes helps Xcode to recognize newly installed certificates or resolve related issues.

* Register devices by **UDID**, even though the form in App Store Connect is confusing!

    ![](code-signing/use-udid.png)

	macOS system report shows the difference:
	![](code-signing/uuid-vs-udid.png)

## Other Aspects

* Role-Based Access Control
  * Assign roles (Admin, Member, etc.) to team members to control permissions related to certificates and profiles management.
* Revocation and Renewal
  * Plan for certificate expiration and renewal. Remember, revoking a certificate affects all associated profiles and signed apps.
* Continuous Integration Systems
  * If using CI systems like Jenkins or GitLab, consider automating certificate and profile handling using tools like Fastlane.
* Two-Factor Authentication
  * Be aware of two-factor authentication when sharing accounts across developers. It might require additional coordination.
* App-Specific Passwords
  * App-specific passwords can give people (or the CI) selective access to the developer account, for example so they can manage and release only one specific app.
* Notarization for macOS Apps
  * If distributing macOS apps outside the App Store, understand the notarization process, which provides an extra layer of security.