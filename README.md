# google-ultra

English

Google One has a 12-month AI Ultra trial offer that does not appear on its pages. It is returned by the Play billing library (BillingClient). This script runs the steps in one command: find the offer, open Play checkout, confirm the price is $0, and activate the trial.

It works by calling BillingClient from Google One's own process. You do not need Xposed or a modified APK.

## Two versions

**Simple version:** A web page. Double-click `web/index.html`, sign in, add a card, and click a plan. No Python, root access, or USB cable is needed.

**Script version:** `ultra0.py` requires a rooted phone and Frida. It retrieves offers from the actual client rather than relying on a browser fingerprint.

Try the simple version first, then the script if needed.

## What you get

**Ultra 5x, 20 TB, 12 months at $0.** Cancel before the trial ends to avoid a charge. After activation, the Play subscription page shows "Next payment: 2027-11-17, US$99.99/month." That is the price after the trial, not an immediate charge.

Offers also exist for the 30 TB tier (20x), but Play checks eligibility again at checkout. Most accounts see "You're not eligible for a free trial." The script will not proceed with that purchase.

## Requirements

A rooted Android phone connected to your computer over USB. Magisk, KernelSU, and APatch are supported. An emulator also works.

Install Python 3.9 or later and Frida on your computer:

```bash
pip3 install frida
```

Use adb from Android SDK platform-tools. After installation, `adb devices` should list the phone. On macOS, use `brew install android-platform-tools`. On Windows, download and extract the platform-tools archive and add its directory to PATH.

Enable "Developer options → USB debugging" on the phone, then allow the connection when prompted.

## Usage

### Simple version

The source is the single file `web/index.html`. Download it and double-click to open it in your browser:

1. Click "Sign in" and use the Google account you want to activate.
2. Click "Add card" and attach a card with $1 available. No payment is taken during the trial.
3. Return to the page and click the "Ultra 20 TB" card.

On Google's checkout page, confirm the amount is 0 before clicking Subscribe.

You can also use the hosted version without downloading the repository: https://gylive.github.io/google-ultra/web/

### Script version

First run the environment check. It checks the device, root access, and frida-server, pushes the billing module, and also disables proxies and Clash that may intercept Play traffic:

```bash
python3 ultra0.py --check
```

Continue when all checks pass.

Create a text file with one account per line, separating fields with four hyphens:

```
email----password----recovery_email----2FA_secret----card_number----expiry----CVC----cardholder----postal_code
```

You can leave the trailing fields empty. Newly purchased accounts usually have no 2FA; leave that field empty:

```
abc123@gmail.com----mypassword----
```

Example with a card:

```
abc123@gmail.com----mypassword--------4242424242424242----10/29----326----ZHANG SAN----97201
```

If no address is provided, the default is `123 Main St, Portland, OR 97201`. Oregon has no sales tax.

Run:

```bash
python3 ultra0.py accounts.txt
```

For a single account:

```bash
python3 ultra0.py --account "abc@gmail.com:password:2FA_secret"
```

Check the result on the subscriptions page:

```
Play Store → profile → Payments & subscriptions → Subscriptions
```

## What happens internally

1. Add the account to the phone if it is not already present.
2. Switch Play accounts, then read the profile panel to verify the selected account. Exit immediately on a mismatch to avoid using another account.
3. Start Google One, attach Frida, and load the billing module with `DexClassLoader`.
4. Query `g1.play.20tb` with `queryProductDetailsAsync`, returning five offers.
5. Select the longest free period (`eft-12month`, P1Y) and call `launchBillingFlow` on the main thread.
6. Monitor checkout. Click Subscribe only when the price is 0 or "12-month free trial" is shown. Close checkout without subscribing if the account is ineligible.
7. Enter the password automatically if Play requests it again.
8. Open the subscriptions page to verify the result and write it to `ultra0-report/`.

## Troubleshooting

**Account not eligible.** Play says "You're not eligible for a free trial, but you can still subscribe." Each account usually gets one chance: starting a trial removes the offer, even if you do not confirm it. Do not test with your primary account.

**No payment method.** A subscription requires a card. Play's add-card page is a WebView that adb cannot fill. The script pauses and asks you to complete it on the phone, then press Enter. Add the card manually once per device; other accounts then use the same card.

**Play did not switch accounts.** The script prints the account actually signed in and exits. Switch manually in Play, then rerun with `--no-switch` to verify the account without trying to switch again:

```bash
python3 ultra0.py accounts.txt --no-switch
```

**Checkout did not open.** Retry once. If it fails twice, check the device network: run `settings get global http_proxy` and clear the proxy if the result is not `null`. VPNs such as Clash may intercept DNS and prevent Play from connecting.

## Options

```bash
--plan 5x      # activate only 20 TB
--plan 20x     # query only 30 TB (most accounts cannot get the trial)
--plan auto    # default: check both tiers and choose the longest free period
--no-add       # skip adding an account that is already on the phone
--no-switch    # do not switch automatically; verify the current account only
```

## Notes

Remember to cancel the subscription in Play before the trial ends; otherwise, you will be charged $99.99.

The script runs locally. Account passwords and card numbers are not uploaded anywhere. Reports in `ultra0-report/` contain offer details and the result of each step.

This code uses the trial offers available to the client in the Play billing flow. It does not modify server-side data.

## Community

Join any of the technical discussion groups. **Read the [group rules](RULES.md) before joining: all advertising is prohibited, and violators will be removed.**

<table>
<tr>
<td align="center"><b>Group 2</b><br><img src="assets/wechat-group.jpg" width="220" alt="Group 2"></td>
<td align="center"><b>Group 3</b><br><img src="assets/wechat-group3.jpg" width="220" alt="Group 3"></td>
<td align="center"><b>Group 4</b><br><img src="assets/wechat-group4.jpg" width="220" alt="Group 4"></td>
</tr>
<tr>
<td align="center"><b>Group 5</b><br><img src="assets/wechat-group5.jpg" width="220" alt="Group 5"></td>
<td align="center"><b>Group 6</b><br><img src="assets/wechat-group6.jpg" width="220" alt="Group 6"></td>
<td align="center"><b>Group 7</b><br><img src="assets/wechat-group7.jpg" width="220" alt="Group 7"></td>
</tr>
<tr>
<td align="center"><b>Group 7</b><br><img src="assets/wechat-group7.jpg" width="220" alt="Group 7"></td>
<td align="center"><b>Group 8</b><br><img src="assets/wechat-group8.jpg" width="220" alt="Group 8"></td>
<td align="center"><b>Group 9</b><br><img src="assets/wechat-group9.jpg" width="220" alt="Group 9"></td>
</tr>
<tr>
<td align="center"><b>Group 10</b><br><img src="assets/wechat-group10.jpg" width="220" alt="Group 10"></td>
<td align="center"><b>Group 11</b><br><img src="assets/wechat-group11.jpg" width="220" alt="Group 11"></td>
</tr>
</table>

## Related links

This project is published and discussed in the [LINUX DO](https://linux.do) community. Thank you for the support.

- [LINUX DO](https://linux.do) — a community for new idealists
- [LINUX DO Open Source Promotion](https://linux.do/tag/1014) — the open-source project promotion section

[![LINUX DO](https://img.shields.io/badge/LINUX%20DO-Community-1f6feb)](https://linux.do)

## Repository layout

```
web/index.html      simple version: standalone web page, also used by the hosted version
web/en.html         alternate English web entry point
simple/ultra.py     minimal script implementation with an embedded module
simple/README.md    script tutorial
ultra0.py           full script: batch processing, tier selection, account switching
assets/            group QR codes
```

Hosted version: https://gylive.github.io/google-ultra/web/

## Disclaimer and Terms of Use

**About this project.** This repository documents how Google Play Billing behaves on the client, with a proof-of-concept (PoC) script to test it. It is a security research project, not a service or product.

**No affiliation with Google.** This project is maintained by an individual and is not affiliated with, authorized by, endorsed by, or sponsored by Google LLC or its affiliates. Google, Google One, Google Play, Gemini, and their logos belong to Google LLC and are referenced descriptively only.

**Risks.** Using this code may violate Google's Terms of Service, resulting in account restrictions or suspension. It may also violate local laws or regulations. You are solely responsible for deciding whether and how to use it and for any consequences. The author provides no legal advice and takes no responsibility for your actions.

**Services not provided.** This repository does not provide accounts, activate subscriptions on others' behalf, sell payment methods, or accept paid commissions. It does not encourage or induce any of these activities.

**Research and learning only.** This repository is intended solely for security research, interface analysis, and technical discussion. Commercial, profit-making, bulk, or large-scale use is prohibited. Anyone using it for those purposes acts independently of the author and bears full responsibility.

**No paid activation, resale, or bulk use.** Using this repository for paid activation services, reselling subscriptions or eligibility, registering or farming accounts in bulk, taking orders for profit, or using others' accounts or payment instruments is strictly prohibited. Such conduct may be unlawful or criminal. The person involved bears all legal responsibility; the author expressly opposes it and provides no assistance.

**Data and privacy.** The web version runs in your local browser and the script on your local device. They communicate only with official Google domains and do not send your Google account, password, recovery email, 2FA, or card information to the author or third-party servers, and do not collect or store that information. Use only accounts and payment methods that you own or are authorized to use. You are responsible for the consequences of entering anyone else's information.

**Minors.** This repository is not intended for minors. Do not use it if you have not reached the local legal age of full civil capacity.

**Local compliance.** Laws concerning exploitation of platform promotions or bypassing billing checks vary by jurisdiction. You must determine whether using this project is lawful where you live. The author makes no guarantee of legality in any jurisdiction.

**Removal at any time.** The author may modify, archive, or delete any or all repository content without notice and does not guarantee that any interface, offer, or feature will remain available.

**Limitation of liability.** The code is provided "as is" under the MIT License, without express or implied warranties, including merchantability, fitness for a particular purpose, and non-infringement. The author is not liable for direct, indirect, incidental, special, or consequential losses arising from use of or inability to use this project, including account loss, charges, data loss, or lost profits. Your sole remedy is to stop using it.

**Rights-holder notices.** Rights holders or their representatives who believe this repository infringes their rights may contact the author through a repository issue or the contact methods listed. Verified requests will be addressed by removing the relevant content or the entire repository.

**Do not use if you disagree.** If you disagree with any of these terms, stop using the project and delete your copies immediately.

## License

MIT. See [LICENSE](LICENSE).

