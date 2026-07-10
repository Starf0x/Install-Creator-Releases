# User Guide

A complete walkthrough of Intune Install Creator: first-run setup, building an MSI package, building an EXE package, and what each field actually does.

## Contents

- [First launch](#first-launch)
- [The main window](#the-main-window)
- [Building an MSI package](#building-an-msi-package)
- [Building an EXE package](#building-an-exe-package)
- [Removing a desktop shortcut before install](#removing-a-desktop-shortcut-before-install)
- [Importing a registry file](#importing-a-registry-file)
- [What "Save" actually produces](#what-save-actually-produces)
- [Staying up to date](#staying-up-to-date)
- [Troubleshooting](#troubleshooting)

## First launch

The first time you run `IntuneInstallCreator.exe`, no preferences exist yet, so the **Preferences** window opens automatically before anything else. This is a one-time setup — fill in:

| Field | What it's for |
|---|---|
| **Vendor Name Replace Space with** | The character used in place of a space when building the vendor part of a folder/file name (e.g. `-` turns "Adobe Inc" into "Adobe-Inc"). |
| **Product Name Replace Space with** | Same idea, for the product name. |
| **Version Replace Dot with** | The character used in place of the dots in a version number when it appears in a generated file/folder name. |
| **File Name Replace Space with** | The character used when renaming the installer file itself (spaces in filenames cause no end of trouble in batch scripts, so they get replaced). |
| **Source Save Location** | The root folder where every package this tool builds will be created. Pick somewhere with room to grow — every vendor/product/version combination gets its own subfolder. |
| **Auto Create Intune File** | Tick this if you want the `.intunewin` file built automatically every time you click Save, instead of doing it as a separate step. |

You can reopen this window any time via the **Settings** button on the main window.

## The main window

![Main window](docs_main_window.png)

At the top, choose your **Mechanism**: **MSI** or **EXE**. The form changes to show the fields relevant to that installer type. A few fields are always visible regardless of mechanism:

- **Vendor** / **ProductName** / **Version** — identify what you're packaging.
- **Build** — a build/release identifier you choose (required).
- **Language** — the language of this build (required).
- **Check LNK File?** — see [Removing a desktop shortcut](#removing-a-desktop-shortcut-before-install).
- **Install REG File** — see [Importing a registry file](#importing-a-registry-file).

## Building an MSI package

1. Set **Mechanism** to **MSI**.
2. Click the **…** button next to the MSI file field and browse to your `.msi`.
   Once selected, **Vendor**, **ProductName**, **Version**, and **Product Code** are read directly out of the MSI and filled in for you — no need to open the MSI in another tool to find these.
3. Optionally browse to an `.mst` transform file if your install needs one.
4. Fill in **Install parameters** (e.g. `/qn` for a fully silent install) and **Uninstall parameters** if needed.
5. Fill in **Build** and **Language**.
6. Click **Save**.

## Building an EXE package

For installers that aren't MSI-based (common with third-party EXE installers):

1. Set **Mechanism** to **EXE**.
2. Browse to the installer **EXE file**.
3. Fill in the **install parameters** your installer accepts for a silent install (this varies by installer — check the vendor's documentation, e.g. `/S`, `/silent`, `/qn`, etc.).
4. Fill in the **uninstall command** (the path/command used to uninstall the app once installed) and its parameters.
5. Fill in **Vendor**, **ProductName**, **Version**, **Build**, and **Language**.
6. Click **Save**.

## Removing a desktop shortcut before install

Some installers drop a desktop shortcut you don't want. Tick **Check LNK File?**, then browse to the `.lnk` file (or type the path it would appear at) — the generated install script will check for it and remove it before installing.

## Importing a registry file

If your install needs to import a `.reg` file as part of installation (MSI packages only), point **Install REG File** at it. It gets copied alongside the installer and imported automatically during install.

## What "Save" actually produces

Clicking **Save**:

1. Creates a folder structure under your configured **Source Save Location**: `Vendor\Product\Version-Language-Build\`.
2. Writes `Install.cmd` and `Uninstall.cmd` into a `Sources` subfolder, built from the details you entered — including logging and install-success/failure tracking via the registry.
3. Copies your installer (and transform/registry file, if any) into that same `Sources` folder.
4. If **Auto Create Intune File** is enabled (or you trigger it separately), packages the `Sources` folder into a `.intunewin` file using the bundled Win32 Content Prep Tool, and copies `Install.cmd` into the Intune import folder — both ready to hand straight to Intune.
5. Opens the result in File Explorer so you can see exactly what was produced.

## Staying up to date

The app checks for a newer version automatically every time it starts (after its splash screen, before the main window appears). If a newer, verified build is available, it downloads, verifies it, and installs it automatically — no prompt, no manual download. You'll occasionally notice startup take a couple of seconds longer than usual; that's the update happening.

## Troubleshooting

- **"You are required to..." messages on Save** — every field the message names is mandatory; the tool won't build a package with any of them missing.
- **Vendor/Product/Version/Filename replacement warnings** — these are configured once in Settings (see [First launch](#first-launch)); if you see this, open Settings and fill in the missing replacement character.
- **"Serious error, to fix, connect to the internet!" no longer appears** — older versions required internet just to start. Current versions only need it for the update check and don't block you from working offline once past that.
- **MSI fields don't populate** — make sure the MSI file isn't corrupted or DRM-protected; the tool reads its property table directly and will show an error if that fails, rather than silently leaving fields blank.
