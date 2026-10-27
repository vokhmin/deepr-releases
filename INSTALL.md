# Installing DeepRrr

DeepRrr is a small translator for macOS: select text, press Cmd+C twice, and a popup shows the translation; a two-pane window works like DeepL's. It is a personal fork of [MoePeek](https://github.com/cosZone/MoePeek) shared with a few friends and colleagues (issue #4). This page is for people who want to **use** DeepRrr (in Russian: [INSTALL.ru.md](INSTALL.ru.md)); building it from source is described in `docs/deep-r/README.md` inside each release's source archive.

> **Status:** no Apple-notarized release yet; the publisher's Apple Developer ID is still being set up, and until then the installer refuses to install. Meanwhile **preview builds** are published: signed with the publisher's own certificate but not checked by Apple. The current one, `0.21.1` preview, is out for a pilot install by friends; if something does not work, tell the publisher. See [Preview builds](#preview-builds-not-notarized-by-apple) below; the other sections (the `install-deepr.sh` installer) apply to the future notarized releases.

## Preview builds (not notarized by Apple)

Until the publisher has an Apple Developer ID certificate, DeepRrr is published as preview builds on the [releases page](https://github.com/vokhmin/deepr-releases/releases), marked **Pre-release** with `preview` in the name. A preview is signed with the publisher's own certificate but Apple has not checked it, so macOS blocks its first launch and you allow it by hand. Install it only if you trust the publisher, and only from that page.

**Certificate fingerprint.** The signature is checked against the publisher's certificate fingerprint `d7b862527611395df9bb15be986dfd655f32ea09`. The publisher sends it to you in a separate message; make sure the two match. If they do not, do not install.

The `install-deepr.sh` installer does not install previews: follow the steps below in **Terminal**. The examples use version `0.21.1`; use the version from your downloaded file name.

1. Download both files of one preview into `~/Downloads`: `DeepRrr-0.21.1-preview.zip` and `DeepRrr-0.21.1-preview.zip.sha256`.
2. Check the checksum; it must print `OK`:

   ```sh
   cd ~/Downloads
   shasum -a 256 -c "DeepRrr-0.21.1-preview.zip.sha256"
   ```

3. Unzip into a staging folder. If `~/Downloads/deepr-new` is left from last time, delete it first, otherwise the command stops:

   ```sh
   mkdir ~/Downloads/deepr-new && ditto -x -k "DeepRrr-0.21.1-preview.zip" ~/Downloads/deepr-new
   ```

4. Verify and install with one block: paste it into Terminal as a whole. It checks that the app is signed by the publisher's certificate, unmodified and really DeepRrr; then it moves the old copy aside (if there is one) and installs the new one as a whole. On any error it stops without going further and says why. If DeepRrr is running, quit it first (menu bar icon > Quit DeepRrr). It must end with `INSTALLED, SIGNATURE OK`:

   ```sh
   bash -eu <<'EOF'
   req='=identifier "dev.av.deepr" and certificate root = H"d7b862527611395df9bb15be986dfd655f32ea09"'
   new="$HOME/Downloads/deepr-new/DeepRrr.app"
   app="$HOME/Applications/DeepRrr.app"
   legacy="$HOME/Applications/DeepR.app"
   backup=""
   fail() { echo "$*" >&2; if [ -n "$backup" ]; then echo "Old copy kept at: $backup" >&2; fi; exit 1; }
   /usr/bin/codesign --verify --deep --strict -R "$req" "$new" || fail "SIGNATURE NOT VALID: do not launch anything, delete deepr-new and tell the publisher"
   [ "$(/usr/bin/defaults read "$new/Contents/Info.plist" CFBundleIdentifier)" = dev.av.deepr ] || fail "Wrong application id: stop and tell the publisher"
   if [ -e "$legacy" ] || [ -L "$legacy" ]; then
     if [ ! -L "$legacy" ] && [ -d "$legacy" ] && [ "$(/usr/bin/defaults read "$legacy/Contents/Info.plist" CFBundleIdentifier 2>/dev/null)" = dev.av.deepr ]; then
       fail "The old DeepR is at ~/Applications/DeepR.app. One-time migration: quit DeepR (menu bar icon > Quit DeepR), move ~/Applications/DeepR.app to the Trash and run this again"
     fi
     fail "Unexpected object at ~/Applications/DeepR.app: inspect it by hand, nothing was replaced"
   fi
   procs=$(/bin/ps -axo args=) || fail "Cannot list the running processes: nothing was replaced"
   case "$procs" in *"$legacy/Contents/MacOS/"*) fail "The old DeepR is still running: quit it (menu bar icon > Quit DeepR) and run this again" ;; esac
   case "$procs" in *"$app/Contents/MacOS/"*) fail "DeepRrr is running: quit it (menu bar icon > Quit DeepRrr) and run this again" ;; esac
   mkdir -p "$HOME/Applications"
   [ ! -L "$app" ] || fail "~/Applications/DeepRrr.app is a link, not the app: sort it out by hand"
   if [ -e "$app" ]; then
     [ -d "$app" ] && [ "$(/usr/bin/defaults read "$app/Contents/Info.plist" CFBundleIdentifier 2>/dev/null)" = dev.av.deepr ] || fail "~/Applications/DeepRrr.app is not DeepRrr: sort it out by hand"
     candidate="$HOME/Downloads/DeepRrr-previous-$(date +%Y%m%d-%H%M%S).app"
     [ ! -e "$candidate" ] && [ ! -L "$candidate" ] || fail "A backup with this name already exists: wait a minute and run again"
     mv "$app" "$candidate" || fail "Could not move the old copy aside: nothing was replaced"
     backup="$candidate"
   fi
   [ ! -e "$app" ] && [ ! -L "$app" ] || fail "The old copy was not moved aside: nothing was replaced"
   mv "$new" "$app" || fail "Could not install the new copy: restore the old one (step 7)"
   /usr/bin/codesign --verify --deep --strict -R "$req" "$app" || fail "The installed copy does not verify: restore the old copy (step 7)"
   echo "INSTALLED, SIGNATURE OK"
   if [ -n "$backup" ]; then echo "Old copy kept at: $backup"; fi
   EOF
   ```

   If it reports a signature or id error, do not launch anything, delete `deepr-new` and tell the publisher. Do not copy the new version over the old one in Finder.
5. First launch: `open ~/Applications/DeepRrr.app`. macOS says it cannot verify the app and does not open it: click **Done**. Open **System Settings > Privacy & Security**, scroll to **Security**, click **Open Anyway** next to the DeepRrr message, enter your Mac password, and choose **Open Anyway** again in the next dialog. If macOS says the app is **damaged** or contains **malware**, or there is no Open Anyway button, stop: delete DeepRrr and tell the publisher. Do not remove the quarantine with commands from the internet.
6. Continue with [First launch](#first-launch): Accessibility, Apple language packs.
7. Once the new version works, delete the copy the block kept (it printed the path, `~/Downloads/DeepRrr-previous-...app`) and `~/Downloads/deepr-new`. If it does not start: quit it, move `~/Applications/DeepRrr.app` to the Trash and move the kept copy back: `mv ~/Downloads/DeepRrr-previous-<date>.app ~/Applications/DeepRrr.app`.

**Updating** a preview: the same steps 1-7. The Accessibility permission is intended to carry over between builds with the same signature, but that is still being verified: after updating, check that Cmd+C+C works, and switch DeepRrr on again in Accessibility if not. macOS may ask for Open Anyway again.

**Moving to notarized releases.** Releases notarized by Apple will have a different signature. You will probably have to grant the permissions again; DeepRrr's settings and data stay.

## What you need

- A Mac with **Apple Silicon** (M1 or later). Intel Macs are not supported.
- **macOS 15** or later.
- From the [DeepRrr releases page](https://github.com/vokhmin/deepr-releases/releases): `DeepRrr-<version>.zip`, `install-deepr.sh` and their `.sha256` files.
- Nothing else: no Xcode, Homebrew or developer tools. Apple Translation works without any account; Gemini is optional and needs your own key.

## Install

1. Download `DeepRrr-<version>.zip`, `DeepRrr-<version>.zip.sha256`, `install-deepr.sh` and `install-deepr.sh.sha256` into the same folder, for example `~/Downloads`, all from the same release on the releases page.
2. Open Terminal and check the installer itself before running it (a shell script has no signature of its own):

   ```sh
   cd ~/Downloads
   shasum -a 256 -c install-deepr.sh.sha256
   ```

   It must print `install-deepr.sh: OK`. Compare the hash in that file with the one the sharer published, too: a checksum proves the file is the one that source published, not who wrote it.
3. Run the installer, with the version from your downloaded file name in place of `0.21.0` (an example only):

   ```sh
   bash install-deepr.sh "DeepRrr-0.21.0.zip" --open
   ```

   The installer checks your Mac, the checksum, that the app is DeepRrr (`dev.av.deepr`), that it is signed by the expected publisher and that Apple notarized it. It installs to `~/Applications/DeepRrr.app` (set `DEEPR_INSTALL_DIR` to choose another folder) and opens it. If any check fails it stops and installs nothing. It never changes macOS security settings.
4. macOS may ask once whether to open an app downloaded from the internet: choose **Open**.

You can also install by hand: unzip the archive and move `DeepRrr.app` to `~/Applications` or `/Applications`. The installer only adds the checks.

## First launch

1. DeepRrr lives in the menu bar (and, by default, in the Dock). The onboarding window guides you through the permissions.
2. **Accessibility** (required): System Settings > Privacy & Security > Accessibility, switch on **DeepRrr**. Without it Cmd+C+C and the shortcuts cannot read your selection.
3. **Screen Recording** (optional): only for translating text in screenshots (OCR). You can skip it.
4. **Apple language packs**: Settings > Services > Apple Translation > Language Packs, download the pairs you need (for example English and Russian), or System Settings > General > Language & Region > Translation Languages.
5. **Gemini** (optional): Settings > Services > Gemini, paste your own key from [Google AI Studio](https://aistudio.google.com/apikey). DeepRrr works without it, using Apple Translation.
6. **Safari** (optional): when DeepRrr cannot read a selection in Safari directly, it may ask to control Safari (macOS asks you under Privacy & Security > Automation), and Safari's **Allow JavaScript from Apple Events** must be on for that fallback (Safari > Settings > Advanced: Show features for web developers; then Safari > Settings > Developer). You do not need it: copying the text (Cmd+C+C) works without it.

## Using DeepRrr

| Action | What happens |
| --- | --- |
| **Cmd+C+C** (Cmd+C twice, at most 0.8 s apart) | translates the selected text in a popup |
| **Opt+D** | translates the selected text |
| **Opt+V** | translates what is on the clipboard |
| click the DeepRrr icon in the Dock | opens the translator window |
| right-click the Dock icon | shows the DeepRrr menu |

By default Russian text is translated to English and everything else to Russian; change it in Settings. If Cmd+C+C or Opt+D does not work in some app (for example Slack or some terminals), copy the text with Cmd+C and press Opt+V, or paste it into the translator window.

## Update

Run the installer again with the new release. It quits the running DeepRrr at the install location, replaces it, and keeps your settings. macOS is expected to keep the Accessibility permission for a release from the same publisher; check after updating, and switch DeepRrr on again in Accessibility if the shortcuts stop working.

## Uninstall

- **Remove the app**: quit DeepRrr (menu bar > Quit) and move `~/Applications/DeepRrr.app` to the Trash.
- **Remove your data too** (optional, separate step): settings in `~/Library/Preferences/dev.av.deepr.plist` (including a Gemini key if you entered one) and `~/Library/Application Support/DeepR`. Also switch DeepRrr off or remove it in System Settings > Privacy & Security > Accessibility.

## Source code

DeepRrr is licensed under the [GNU AGPL-3.0](https://www.gnu.org/licenses/agpl-3.0.html) (the full text is in each release's source archive). Every release comes with `DeepRrr-<version>-source.tar.gz`, the exact source it was built from, including the build scripts and license.

## Install with an AI agent

You can ask an AI agent on your Mac (for example Claude Code or Codex) to install DeepRrr, for example: "Install DeepRrr from ~/Downloads following https://github.com/vokhmin/deepr-releases/blob/main/INSTALL.md, the AI agent section". The agent follows this section; if it cannot open the link, give it the text of this guide.

**The agent may:**
- check the Mac: `uname -m` must be `arm64`, `sw_vers -productVersion` must be 15 or later; check whether `~/Applications/DeepRrr.app` already exists;
- verify the installer first with `shasum -a 256 -c install-deepr.sh.sha256` (it must print OK; the hash must match the value from the source the user named), and only then run `bash install-deepr.sh "DeepRrr-<version>.zip"` with the real file name in the quotes (add `--open` if the user wants it started), with the archive and its `.sha256` file in the same folder;
- report the installer's output to the user.

**The agent must stop and tell the user when:**
- the installer's own checksum does not match, or the installer reports any failed check (checksum, bundle id, publisher, notarization, architecture, macOS version) or refuses because no release is published; it must not work around a failed check, re-sign the app, remove the quarantine attribute, change Gatekeeper or other security settings, or download the files from anywhere other than the source the user named;
- DeepRrr is running and does not quit.

**Only the user can do, so the agent lists them at the end:**
- answer the macOS prompt to open the app the first time;
- switch on DeepRrr in System Settings > Privacy & Security > Accessibility (and optionally Screen Recording);
- download Apple language packs if macOS asks;
- enter their own Gemini key, if they want Gemini (the agent must not ask for it or write it anywhere).

The agent must not delete user settings or data as part of an install or update, and must not uninstall DeepRrr unless the user asks; removing data is a separate, explicit request.

**Preview builds through an agent.** The agent may do steps 1-4 and 7 of [Preview builds](#preview-builds-not-notarized-by-apple): download the two files from the releases page only, check the checksum and run the step 4 block unchanged (it must end with `INSTALLED, SIGNATURE OK`; on any other ending the agent stops and does not fix anything itself). The user compares the certificate fingerprint with the publisher's message. Step 5 (Open Anyway) and the permissions are the user's: the agent does not remove the quarantine (`xattr`), does not change Gatekeeper (`spctl`) and does not re-sign the app.
