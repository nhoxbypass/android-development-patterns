# Security

## Secure Android Apps

* **Never ship a production app without enabling Proguard/R8**: This will obfuscate your code and will make it unreadable so if someone tries to reverse engineer your app. This will also make your APK smaller since it will remove unused code from your APK
* **APIs must have keys and store them safely**: To avoid directly call the API endpoints without this key. But never ever put a server secret key in the client code
* **You must write Firebase security rules**: If not, with simple curl command, anyone can delete and manipulate your entire database.
* **Restrict google API key access**: Because a bad person finds your unrestricted API key and make a lot of requests using your Key, which make you receive an invoice as tall as Mount Everest
* **HTTPS everything**: Use HTTPS communications to ensure maximum security and the data you get from the network or post to your servers won’t be compromised.
* **Never ever store Password and private keys in shared preference**: SharedPreferences API using XML files to store, so anyone can read it in plain text.
* **Watch what you print in LogCat**
* **Use Internal Storage for Sensitive data**
* **Do not pass sensitive information through Broadcast**: because other apps can register it and listen to your events
* **Use WebView carefully**: Because WebView consumes web content, so any common web security (like cross-site-scripting,..) can affect your app.
* **Keep your dependencies up to date**
* **Protect your Service and content provider with Permission**: to prevent that make sure to set the service `exported` flag to `false`.