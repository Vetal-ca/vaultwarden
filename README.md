# Helm chart for Vaultwarden

[![MIT Licensed](https://img.shields.io/github/license/guerzon/vaultwarden)](https://github.com/guerzon/vaultwarden/blob/main/LICENSE)
[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/vaultwarden)](https://artifacthub.io/packages/search?repo=vaultwarden)

[Vaultwarden](https://github.com/dani-garcia/vaultwarden), formerly known as **Bitwarden_RS**, is an "alternative implementation of the Bitwarden server API written in Rust and compatible with [upstream Bitwarden clients](https://bitwarden.com/download/), perfect for self-hosted deployment where running the official resource-heavy service might not be ideal."

## Description

This [Helm](https://helm.sh/docs/) chart is used to deploy `vaultwarden` with a stable configuration to Kubernetes clusters.

The `vaultwarden` project can be found [here](https://github.com/dani-garcia/vaultwarden). To learn more about Vaultwarden, please visit the [wiki](https://github.com/dani-garcia/vaultwarden/wiki).

### Change of Resource Type in Versions >= 0.18.0

Starting from version 0.18.0, when a stateless configuration is detected that utilizes an external database and persistent storage, a `Deployment` is automatically used in favor of the current `StatefulSet`. This enables running multiple pods simultaneously, thereby enhancing the processes of updates, rollbacks, and scalability for load balancing. This automatic detection can be overridden by manually specifying a `resourceType`.

## Prerequisites

- Kubernetes >= 1.12
- Helm >= 3.1.0
- `docker` and `make` for generating the chart documentation

## Usage

Add the repository:

```bash
helm repo add vaultwarden https://guerzon.github.io/vaultwarden
```

Refer to the detailed documentation [here](./charts/vaultwarden/README.md).

## Follow-up: mobile push and breach reports

These two options stay off until you set them. They are independent of each other. Do one, both, or neither. The commands below use release name `vaultwarden` and namespace `vaultwarden`. Use your own if they differ.

Keep the credentials in Kubernetes Secrets. Leave `pushNotifications.installationId.value`, `pushNotifications.installationKey.value`, and `hibp.value` empty so the id and keys are not stored in the Helm release.

### Mobile push

Push lets the official Bitwarden mobile apps, browser extension, and web vault pick up vault changes without a manual sync. Vaultwarden sends the notice to Bitwarden's push service, which delivers it to the device. The pod needs outbound HTTPS to the relay and identity addresses below. The phone or tablet needs a Bitwarden app from the App Store, Google Play, or Aurora Store, and it needs to reach `firebaseinstallations.googleapis.com`. Apps installed from F-Droid or other stores are built without Firebase and will not receive these notifications.

1. Open <https://bitwarden.com/host> and submit the email address for this installation. Save the installation id and the installation key. If you requested them for the European Union region, you will point the chart at the EU relay in step 3.
2. Create a Secret in the same namespace as the release. The key names must match the chart:

   ```bash
   kubectl create secret generic vaultwarden-push \
     --namespace vaultwarden \
     --from-literal=PUSH_INSTALLATION_ID='paste-installation-id' \
     --from-literal=PUSH_INSTALLATION_KEY='paste-installation-key'
   ```

3. Add this to the Helm values you already use for the release. Both `existingSecretKey` fields are required. With either one left empty, the chart does not mount that variable and push stays inactive.

   ```yaml
   pushNotifications:
     enabled: true
     existingSecret: vaultwarden-push
     installationId:
       existingSecretKey: PUSH_INSTALLATION_ID
     installationKey:
       existingSecretKey: PUSH_INSTALLATION_KEY
   ```

   For an installation id issued for the EU region, add:

   ```yaml
   pushNotifications:
     relayUri: https://api.bitwarden.eu
     identityUri: https://identity.bitwarden.eu
   ```

   The US defaults are `https://push.bitwarden.com` and `https://identity.bitwarden.com`.

4. Upgrade the release with that values file:

   ```bash
   helm upgrade vaultwarden ./charts/vaultwarden \
     --namespace vaultwarden \
     --values /path/to/your-values.yaml
   ```

5. Log out and log back in on each client so it can register for push. A device that connected before Vaultwarden 1.30.2 and still does not receive notifications needs its app data cleared, or the app reinstalled, and then a fresh login.
6. Check it from the web vault: rename a folder and confirm the mobile app shows the new name after a few seconds.

Switching between the US and EU relay later means logging out everywhere and clearing the mobile app data, then repeating these steps with the other region's id, key, and URIs.

### Have I Been Pwned breach report

The web vault's data-breach report checks usernames and email addresses saved in the vault against [Have I Been Pwned](https://haveibeenpwned.com/). That check needs a paid API key from <https://haveibeenpwned.com/API/Key>. The exposed-passwords report uses the free Have I Been Pwned range API and already works with no key.

1. Request a key at <https://haveibeenpwned.com/API/Key> and copy it.
2. Create a Secret in the release namespace:

   ```bash
   kubectl create secret generic vaultwarden-hibp \
     --namespace vaultwarden \
     --from-literal=HIBP_API_KEY='paste-api-key'
   ```

3. Add this to the same Helm values:

   ```yaml
   hibp:
     existingSecret: vaultwarden-hibp
     existingSecretKey: HIBP_API_KEY
   ```

   `existingSecretKey` can be omitted when the Secret key is `HIBP_API_KEY`. Set it when the key name is different.

4. Upgrade the release, as in the push steps above.
5. In the web vault, open Tools, then Reports, then Data breach report, and run Check breaches.

A saved admin-page config overrides this environment variable. If the report still says the key is missing, open `/admin`, set the Have I Been Pwned API key there once, or clear that saved field so the Secret is the one Vaultwarden reads.

## Disclaimer

Please do your due-diligence before using this chart for a production deployment.

Nevertheless, if you find any issues while using this chart, or have any suggestions, I would appreciate it if you would [submit an issue](https://github.com/guerzon/vaultwarden/issues/new). Alternatively, PRs are appreciated!

## License

See [LICENSE](./LICENSE).

## Author

This Helm chart was created and maintained by [Lester Guerzon](https://guerzon.net).

### Maintainers

Interested in maintaining this chart? Drop me an email at lester@guerzon[.]net.

### Credits

- The `vaultwarden` project can be found [here](https://github.com/dani-garcia/vaultwarden)
- Further information about `Bitwarden` and 8bit Solutions LLC can be found [here](https://bitwarden.com/)

## References

- Guides: <https://github.com/dani-garcia/vaultwarden/wiki>
- Configuration: <https://github.com/dani-garcia/vaultwarden/blob/main/.env.template>
- Releases: <https://github.com/dani-garcia/vaultwarden/releases>
