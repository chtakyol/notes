Mentör’e Sunum ya da Yazı: * OWASP Top 10 Mobile ve güvenlik uygulamalarıyla ilgili olarak mentöre bir sunum yapmak veya yazılı bir doküman hazırlayarak sunmak. https://adesso.udemy.com/course/the-complete-mobile-ethical-hacking-course/learn/lecture/17749926?start=0#overview Kursu üzerinden eğitimi takip edip ethical hacking konusunda bilgi sahibi olmak.

# 1. Introduction
**OWASP (Open Worldwide Application Security Project)** is a global initiative that promotes secure software development. In mobile apps, security is vital due to sensitive data, untrusted environments, and common threats like reverse engineering. To guide developers, OWASP provides the **Mobile Top 10**, a list of the most critical mobile app security risks. The latest 2024 edition highlights issues like insecure credential handling, weak authentication, and supply chain vulnerabilities.
- M1: Improper Credential Usage
- M2: Inadequate Supply Chain Security
- M3: Insecure Authentication/Authorization
- M4: Insufficient Input/Output Validation
- M5: Insecure Communication
- M6: Inadequate Privacy Controls
- M7: Insufficient Binary Protections
- M8: Security Misconfiguration
- M9: Insecure Data Storage
- M10: Insufficient Cryptography
## M1: Improper Credential Usage
This refers to storing, handling, or transmitting credentials (usernames, passwords, tokens, API keys, etc.) **insecurely**. Attackers can exploit this to gain unauthorized access to user accounts or backend systems.
#### Example on Android:
- Storing user passwords or API tokens in **SharedPreferences** or **internal storage** without encryption.
- Hardcoding API keys directly in the app code or in resource files.
- Using **Basic Auth** without additional protection layers.
#### Prevention Tips:
- **Never hardcode sensitive credentials** in the app.
- Use **Android Keystore System** or **Environmental Variable** to store tokens securely.
- Use **EncryptedSharedPreferences** or **SQLCipher** if local storage is necessary.
- **Obfuscate code** (ProGuard/R8) and use string encryption if necessary.
- Implement **backend-side authentication logic** instead of relying solely on client-side security.
## M2: Inadequate Supply Chain Security
This refers to **risks from third-party libraries, SDKs, dependencies, and plugins** that may introduce vulnerabilities, malware, or backdoors into your app — either intentionally or unintentionally. The mobile app becomes vulnerable not because of your code, but because of **what you import and trust**.
#### Example on Android:
- Using outdated libraries with known CVEs (Common Vulnerabilities and Exposures).
- Including SDKs that collect excessive user data (privacy concerns).
- A malicious third-party library that leaks data or injects code at runtime.
- Automatically accepting transitive dependencies without auditing them.
#### Prevention Tips:
- **Audit all dependencies regularly** — review their source and maintainers.
- Use tools like:
    - **OWASP Dependency-Check**
    - **Snyk**
    - **Gradle’s `dependencyUpdates` plugin**
- Prefer **well-maintained, open-source libraries** or **official Google libraries** with a strong community.
- **Avoid unnecessary libraries** — every dependency is a risk.
- Lock dependency versions using `version catalogs` or dependency constraints.
## M3: Insecure Authentication/Authorization
This happens when the app **doesn't properly verify the user's identity** or doesn't **check what a user is allowed** to do. It can let attackers log in without valid credentials or access other users’ data.
#### Example on Android:
- Weak or missing authentication logic (e.g., just checking a local flag like `isLoggedIn = true`).
- Performing permission checks only on the client side instead of the backend.
- No session timeout or token expiration.
#### Prevention Tips:
- Use secure authentication methods like **OAuth 2.0** or **Firebase Auth**.
- Always check **authorization on the server side**, not just in the app.
- Make sure sessions expire after a period of inactivity.
- Avoid storing login status or user roles in SharedPreferences without encryption.
- Use **Firebase App Check** to ensure only valid, untampered app instances can access your backend services like Firebase Realtime Database, Firestore, or Cloud Functions.
## M4: Insufficient Input/Output Validation
This happens when the app doesn’t properly check user input or data from other sources. Attackers can send unexpected or malicious data to break the app, steal data, or trick it into behaving incorrectly.
#### Example on Android:
- Not validating form fields like email, phone number, or usernames.
- Accepting any input and passing it directly to the backend without checks.
- Showing unfiltered data in the UI, leading to issues like **HTML/JavaScript injection** in WebViews.
#### Prevention Tips:
- Always **validate and sanitize all input** — both on the app and on the backend.
- Use built-in input validation tools (e.g., regex for email or phone formats).
- Avoid directly displaying untrusted data in **WebViews** or HTML content.
- If you must use WebViews, **disable JavaScript unless absolutely needed**, and use **safe encoding** methods.
``` Kotlin
@Composable
fun EmailInputWithValidation() {
    var email by remember { mutableStateOf("") }
    var isValid by remember { mutableStateOf(true) }

    Column(modifier = Modifier.padding(16.dp)) {
        OutlinedTextField(
            value = email,
            onValueChange = {
                email = it
                isValid = android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches()
            },
            label = { Text("Email") },
            isError = !isValid,
            modifier = Modifier.fillMaxWidth()
        )

        if (!isValid && email.isNotEmpty()) {
            Text(
                text = "Please enter a valid email address",
                color = MaterialTheme.colorScheme.error,
                style = MaterialTheme.typography.bodySmall,
                modifier = Modifier.padding(top = 4.dp)
            )
        }
    }
}

```

## M5: Insecure Communication
This risk happens when data sent between the app and servers is not properly protected. Attackers can intercept or modify this data if it’s not encrypted or if secure connection checks are weak.
#### Example on Android:
- Sending data over **HTTP instead of HTTPS**.
- Using **invalid or self-signed SSL certificates** without proper validation.
- Not validating the server’s identity (e.g., no certificate pinning).
- Leaking sensitive data (like tokens or credentials) through insecure network calls or logs.
#### Prevention Tips:
- Always use **HTTPS (TLS 1.2 or higher)** for all network communication.
- Implement **certificate pinning** using tools like **TrustKit** or **network security config**.
- Never log sensitive data like tokens or passwords.
- Use libraries like **Retrofit + OkHttp** with secure configurations.
- Monitor network traffic during development using tools like **Charles Proxy** or **Wireshark** to spot leaks.
## M6: Inadequate Privacy Controls
This occurs when apps fail to protect user privacy by not properly handling sensitive data or not giving users enough control over their personal information. It can lead to accidental exposure or unauthorized access to sensitive data.
#### Example on Android:
- Storing sensitive information (like location, contacts, or financial data) **without encryption** or **clear access controls**.
- Sharing **personal data** with third parties without user consent or providing no way for users to review/modify shared data.
- Not properly clearing sensitive data when the app is uninstalled or after the session ends.
#### Prevention Tips:
- **Encrypt sensitive data** both at rest and in transit (use **EncryptedSharedPreferences** or **SQLCipher**).
- **Limit data collection** to only what’s necessary for the app’s functionality and get **explicit user consent**.
- Use Android’s **permission model** to request sensitive data access (e.g., location, camera).
- Provide users with an option to **delete their data** or review what’s stored (e.g., via settings).
- Use **Android’s Secure Storage** options and **Data Protection API** for extra layers of security.
## M7: Insufficient Binary Protections
This risk occurs when the app’s binary code (APK) is not properly protected against reverse engineering or tampering, making it easier for attackers to modify or exploit the app.
#### Example on Android:
- The app’s binary is unprotected, and attackers can easily decompile and reverse-engineer the APK.
- No **code obfuscation** or **app integrity checks** are applied, making it easier to extract secrets or modify functionality.

#### Prevention Tips:
- Use **code obfuscation** (ProGuard or R8) to make it harder to reverse-engineer the code.
- Implement **integrity checks** to detect if the app’s code has been tampered with (e.g., checksum or hash validation).
- Use **tamper detection** mechanisms like **SafetyNet Attestation** or **Firebase App Check**.
## M8: Security Misconfiguration
This occurs when app settings, servers, or APIs are not securely configured, leaving them vulnerable to attacks. Misconfigurations can expose sensitive information or allow attackers to access unauthorized resources.
#### Example on Android:
- The app is not properly **restricting API access**, allowing any user or app to access certain data or functionality.
- Leaving **debug or development features** enabled in the production environment, which could expose internal data or control features.
#### Prevention Tips:
- Ensure all **API endpoints** require proper authentication and authorization.
- Disable **debugging** and any development features in production.
- Use **proper server-side security settings** like firewalls, access controls, and proper database configurations.
## M9: Insecure Data Storage
This occurs when the app stores sensitive data in a way that allows attackers to easily access it. Without proper encryption or storage protection, sensitive data like passwords, tokens, or personal information can be exposed.
#### Example on Android:
- Storing sensitive data in **unencrypted SharedPreferences** or **local databases**.
- Storing private keys or tokens in **plain text** on the device, which can be accessed through reverse engineering.
#### Prevention Tips:
- Use **EncryptedSharedPreferences** or **SQLCipher** for database encryption.
- Never store sensitive data in **plain text**. Always encrypt it before saving.
- Use **Android Keystore** for securely storing sensitive information like cryptographic keys.
## M10: Insufficient Cryptography
This risk arises when the app uses weak or outdated cryptographic algorithms for encrypting or securing data. This makes it easier for attackers to decrypt or manipulate the data.
#### Example on Android:
- Using weak or deprecated algorithms like **MD5** or **SHA-1** for hashing or encryption.
- Implementing custom, insecure cryptography instead of using trusted libraries or standards.
#### Prevention Tips:
- Use strong cryptographic algorithms like **AES** for encryption and **SHA-256** for hashing.
- Rely on trusted libraries like **Google’s Tink** or the **Android Keystore** for cryptographic operations.
- Regularly update the app to replace deprecated or weak cryptographic methods with more secure alternatives.