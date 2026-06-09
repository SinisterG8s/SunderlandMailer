# Installing the Sunderland Email Sender

This guide walks you through installing the app on your Windows PC. It takes
about 5 minutes and you only do it **once**. No technical knowledge needed —
just follow the steps in order.

If you get stuck at any point, take a screenshot and send it to whoever gave
you these files.

- - -

## What you should have

You'll have been given a folder containing **three files**. The names may have
a version number in them, but they look like this:

| File | Looks like | What it is |
| ---- | ---------- | ---------- |
| 📦 The app | `SunderlandEmailSender_1.0.0.0_x64.msix` | The program itself |
| 📜 The certificate | `SunderlandEmailSender_1.0.0.0_x64.cer` | A one-time "trust" file Windows needs |
| ⚙️ The installer helper | `Add-AppDevPackage.ps1` | Does the install for you automatically |

> 💡 **Keep all three files together in the same folder.** The helper needs the
> other two next to it. The easiest path is the **automatic** install below.

- - -

## The easy way (recommended)

This uses the helper file to install everything for you.

1. Find the file called <b>`Add-AppDevPackage.ps1`</b> (the ⚙️ one).
2. **Right-click** it and choose **"Run with PowerShell"**.
3. A blue window will open and ask some questions. **Say yes / press Enter** to
each one — it may ask permission to install the trust certificate, then to
install the app.
4. When you see the message that installation **succeeded**, you're done.
You can close the window.

> ⚠️ **If Windows shows a blue "Windows protected your PC" box:** click
> **"More info"**, then **"Run anyway"**. This is normal for company apps that
> aren't sold in the Microsoft Store.

> ⚠️ **If right-click doesn't show "Run with PowerShell":** use **The manual**
> way below instead — it does exactly the same thing.

That's it. Skip ahead to **Turn on file access** below.

- - -

## The manual way (only if the easy way didn't work)

You'll do two small steps: trust the certificate, then install the app.

### Step 1 — Trust the certificate (one time)

1. Double-click the <b>`.cer`</b> file (the 📜 one).
2. Click **"Install Certificate…"**.
3. Choose **"Local Machine"**, then click **Next**.
(If Windows asks for an admin password, enter it or ask whoever set up your PC.)
4. Choose **"Place all certificates in the following store"**, click **Browse**,
and select **"Trusted People"**. Click **OK**, then **Next**, then **Finish**.
5. You should see **"The import was successful."** Click **OK**.

### Step 2 — Install the app

1. Double-click the <b>`.msix`</b> file (the 📦 one).
2. Click the **Install** button.
3. When it finishes, the app may open automatically.

- - -

## Turn on file access (important — do this once)

The app needs permission to read your recipients spreadsheet and save its
progress. Without this, sending will fail.

1. Click the Windows **Start** button and type **"File system privacy"**, then
open **"File system privacy settings"**.
2. Make sure **"File system access"** is turned **On**.
3. Scroll down to the list of apps and turn **On** the switch next to
**Sunderland Email Sender**.

- - -

## Opening the app

From now on, just click the Windows **Start** button, type **"Sunderland"**,
and click the app. You can also right-click it in the Start menu and choose
**"Pin to taskbar"** so it's always one click away.

You do **not** need any of the three install files after this — you can delete
them or keep them somewhere safe in case you ever reinstall.

- - -

## First time you use it

1. Click **+ New profile**, give it your name, and a folder will open.
2. Drop your <b>`credentials.json`</b> (and optional <b>`signature.html`</b>) into
that folder — you'll have been given these.
3. Pick your profile in the app and sign in with Google when prompted.

(Whoever set this up can help you with the Google sign-in the first time.)

- - -

## Troubleshooting

| Problem | Fix |
| ------- | --- |
| "Run with PowerShell" is missing when I right-click | Use **The manual way** above. |
| "Windows protected your PC" popup | Click **More info → Run anyway**. |
| The app installs but sending fails / can't find the spreadsheet | Do the **Turn on file access** steps above. |
| It asks for an administrator password | Ask whoever manages your PC, or the person who sent you the files. |
| Nothing works | Take a screenshot of the error and send it to your contact. |

- - -

<br>

> **⬇️ The rest of this page is for the person who maintains the app — the sales
> team can stop reading here.**

## Maintainer notes (not for the sales team)

### Signing certificate

The app is signed with a self-signed certificate so Windows will let it
install. Key file: `windows/SunderlandEmailSender/SunderlandEmailSender_TemporaryKey.pfx`
(password-less). Public certificate for distribution:
`windows/SunderlandEmailSender/SunderlandEmailSender.cer`.

| Detail | Value |
|--------|-------|
| Subject / Publisher | `CN=gerard` (must match `Publisher` in `Package.appxmanifest`) |
| Issued | 9 June 2026 |
| **Expires** | **9 June 2031** |

**What expiry actually breaks:** Windows validates the signature only at
**install** time, not on every launch. So PCs that already have the app keep
working after the cert expires — *but* you won't be able to install on a new/
replacement PC or push an update until you generate a fresh cert.

### Renewing the certificate (do this before June 2031)

Run from the repo root in PowerShell. Keep the subject `CN=gerard` so Windows
treats it as the same app:

```powershell
$pfx = "windows\SunderlandEmailSender\SunderlandEmailSender_TemporaryKey.pfx"
$cer = "windows\SunderlandEmailSender\SunderlandEmailSender.cer"

$cert = New-SelfSignedCertificate -Type Custom -Subject "CN=gerard" `
  -KeyUsage DigitalSignature -KeyExportPolicy Exportable `
  -FriendlyName "Sunderland Email Sender" `
  -CertStoreLocation "Cert:\CurrentUser\My" `
  -NotAfter (Get-Date).AddYears(5) `
  -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.3")

[System.IO.File]::WriteAllBytes((Resolve-Path $pfx).Path, `
  $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pfx))
Export-Certificate -Cert ("Cert:\CurrentUser\My\" + $cert.Thumbprint) -FilePath $cer
```

Then rebuild the package (Release / x64), and hand out the new `.msix` plus the
new `.cer`. The team trusts the new `.cer` once, exactly like the first install.

> If the manifest `Publisher` and the cert `Subject` ever differ, signing or
> install will fail — they must match character-for-character.

### Avoiding this chore (and the "Run anyway" popup)

A real code-signing certificate removes both the yearly renewal pain and the
SmartScreen warning the team currently clicks through:

- **Azure Trusted Signing** — ~$10/month, simplest modern option.
- **Standard code-signing cert** (Sectigo / DigiCert) — ~£150–250/year.

With either, timestamp the signature so already-signed packages keep installing
even after the cert expires.
