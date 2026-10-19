# Installing DeepR

DeepR is a small translator for macOS: select text, press Cmd+C twice, and a popup shows the translation; a two-pane window works like DeepL's. It is a personal fork of [MoePeek](https://github.com/cosZone/MoePeek) shared with a few friends and colleagues (issue #4). This page is for people who want to **use** DeepR (in Russian: [INSTALL.ru.md](INSTALL.ru.md)); building it from source is described in `docs/deep-r/README.md` inside each release's source archive.

> **Status:** no release is published yet. The publisher's Apple Developer ID is being set up; until then the installer refuses to install. This page describes the process as it will work.

## What you need

- A Mac with **Apple Silicon** (M1 or later). Intel Macs are not supported.
- **macOS 15** or later.
- From the [DeepR releases page](https://github.com/vokhmin/deepr-releases/releases): `DeepR-<version>.zip`, `install-deepr.sh` and their `.sha256` files.
- Nothing else: no Xcode, Homebrew or developer tools. Apple Translation works without any account; Gemini is optional and needs your own key.

## Install

1. Download `DeepR-<version>.zip`, `DeepR-<version>.zip.sha256`, `install-deepr.sh` and `install-deepr.sh.sha256` into the same folder, for example `~/Downloads`, all from the same release on the releases page.
2. Open Terminal and check the installer itself before running it (a shell script has no signature of its own):

   ```sh
   cd ~/Downloads
   shasum -a 256 -c install-deepr.sh.sha256
   ```

   It must print `install-deepr.sh: OK`. Compare the hash in that file with the one the sharer published, too: a checksum proves the file is the one that source published, not who wrote it.
3. Run the installer, with the version from your downloaded file name in place of `0.21.0` (an example only):

   ```sh
   bash install-deepr.sh "DeepR-0.21.0.zip" --open
   ```

   The installer checks your Mac, the checksum, that the app is DeepR (`dev.av.deepr`), that it is signed by the expected publisher and that Apple notarized it. It installs to `~/Applications/DeepR.app` (set `DEEPR_INSTALL_DIR` to choose another folder) and opens it. If any check fails it stops and installs nothing. It never changes macOS security settings.
4. macOS may ask once whether to open an app downloaded from the internet: choose **Open**.

You can also install by hand: unzip the archive and move `DeepR.app` to `~/Applications` or `/Applications`. The installer only adds the checks.

## First launch

1. DeepR lives in the menu bar (and, by default, in the Dock). The onboarding window guides you through the permissions.
2. **Accessibility** (required): System Settings > Privacy & Security > Accessibility, switch on **DeepR**. Without it Cmd+C+C and the shortcuts cannot read your selection.
3. **Screen Recording** (optional): only for translating text in screenshots (OCR). You can skip it.
4. **Apple language packs**: Settings > Services > Apple Translation > Language Packs, download the pairs you need (for example English and Russian), or System Settings > General > Language & Region > Translation Languages.
5. **Gemini** (optional): Settings > Services > Gemini, paste your own key from [Google AI Studio](https://aistudio.google.com/apikey). DeepR works without it, using Apple Translation.
6. **Safari** (optional): when DeepR cannot read a selection in Safari directly, it may ask to control Safari (macOS asks you under Privacy & Security > Automation), and Safari's **Allow JavaScript from Apple Events** must be on for that fallback (Safari > Settings > Advanced: Show features for web developers; then Safari > Settings > Developer). You do not need it: copying the text (Cmd+C+C) works without it.

## Using DeepR

| Action | What happens |
| --- | --- |
| **Cmd+C+C** (Cmd+C twice, at most 0.8 s apart) | translates the selected text in a popup |
| **Opt+D** | translates the selected text |
| **Opt+V** | translates what is on the clipboard |
| click the DeepR icon in the Dock | opens the translator window |
| right-click the Dock icon | shows the DeepR menu |

By default Russian text is translated to English and everything else to Russian; change it in Settings. If Cmd+C+C or Opt+D does not work in some app (for example Slack or some terminals), copy the text with Cmd+C and press Opt+V, or paste it into the translator window.

## Update

Run the installer again with the new release. It quits the running DeepR at the install location, replaces it, and keeps your settings. macOS is expected to keep the Accessibility permission for a release from the same publisher; check after updating, and switch DeepR on again in Accessibility if the shortcuts stop working.

## Uninstall

- **Remove the app**: quit DeepR (menu bar > Quit) and move `~/Applications/DeepR.app` to the Trash.
- **Remove your data too** (optional, separate step): settings in `~/Library/Preferences/dev.av.deepr.plist` (including a Gemini key if you entered one) and `~/Library/Application Support/DeepR`. Also switch DeepR off or remove it in System Settings > Privacy & Security > Accessibility.

## Source code

DeepR is licensed under the [GNU AGPL-3.0](https://www.gnu.org/licenses/agpl-3.0.html) (the full text is in each release's source archive). Every release comes with `DeepR-<version>-source.tar.gz`, the exact source it was built from, including the build scripts and license.

## Install with an AI agent

You can ask an AI agent on your Mac (for example Claude Code or Codex) to install DeepR, for example: "Install DeepR from ~/Downloads following https://github.com/vokhmin/deepr-releases/blob/main/INSTALL.md, the AI agent section". The agent follows this section; if it cannot open the link, give it the text of this guide.

**The agent may:**
- check the Mac: `uname -m` must be `arm64`, `sw_vers -productVersion` must be 15 or later; check whether `~/Applications/DeepR.app` already exists;
- verify the installer first with `shasum -a 256 -c install-deepr.sh.sha256` (it must print OK; the hash must match the value from the source the user named), and only then run `bash install-deepr.sh "DeepR-<version>.zip"` with the real file name in the quotes (add `--open` if the user wants it started), with the archive and its `.sha256` file in the same folder;
- report the installer's output to the user.

**The agent must stop and tell the user when:**
- the installer's own checksum does not match, or the installer reports any failed check (checksum, bundle id, publisher, notarization, architecture, macOS version) or refuses because no release is published; it must not work around a failed check, re-sign the app, remove the quarantine attribute, change Gatekeeper or other security settings, or download the files from anywhere other than the source the user named;
- DeepR is running and does not quit.

**Only the user can do, so the agent lists them at the end:**
- answer the macOS prompt to open the app the first time;
- switch on DeepR in System Settings > Privacy & Security > Accessibility (and optionally Screen Recording);
- download Apple language packs if macOS asks;
- enter their own Gemini key, if they want Gemini (the agent must not ask for it or write it anywhere).

The agent must not delete user settings or data as part of an install or update, and must not uninstall DeepR unless the user asks; removing data is a separate, explicit request.
