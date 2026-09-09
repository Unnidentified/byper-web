#!/bin/bash
# byper installer — sets up the byper CLI + menu bar app on Apple Silicon Macs.
# Usage: curl -fsSL https://byper.org/install | bash
#
# What it does:
#   1. Checks the platform (Apple Silicon, macOS 11+, not Linux/Intel).
#   2. Downloads the latest byper-installer.pkg from GitHub Releases.
#   3. Verifies it, then opens it in the macOS Installer.
#   4. Prints the one-time SIP prerequisite (csrutil enable --without debug).
#
# The actual privileged install happens through the standard macOS installer
# UI (admin password prompt, Touch ID when offered) — piping to bash only
# stages the package and opens it, so nothing installs without your consent.

set -euo pipefail

REPO="Unnidentified/byper"
DL="https://github.com/${REPO}/releases/latest/download"

# ── 1. Platform checks ──────────────────────────────────────────────────────
if [[ "$(uname)" != "Darwin" ]]; then
    echo "byper: this installer is macOS-only." >&2
    exit 1
fi
if [[ "$(uname -m)" != "arm64" ]]; then
    echo "byper: Apple Silicon (arm64) only — no Intel build exists." >&2
    exit 1
fi
MAC_MAJOR=$(sw_vers -productVersion | cut -d. -f1)
if [[ "${MAC_MAJOR:-0}" -lt 11 ]]; then
    echo "byper: macOS 11 Big Sur or newer is required (you have $(sw_vers -productVersion))." >&2
    exit 1
fi

echo "==> byper: downloading the latest installer (Apple Silicon)…"

TMP_DIR=$(mktemp -d)
trap 'rm -rf "$TMP_DIR"' EXIT
TMP="$TMP_DIR/byper-installer.pkg"

# The bare .pkg download loses the Finder custom icon (HTTP strips resource
# forks) but the installer itself is byte-identical. Prefer the DMG when a
# desktop session is available so the badge survives; fall back to the pkg.
PKG_URL="${DL}/byper-installer.pkg"
if curl -fsSL -o "$TMP" "$PKG_URL"; then
    :
else
    echo "byper: download failed (no network, or the release moved)." >&2
    exit 1
fi

# Sanity: a valid pkg begins with the xar magic.
if [[ "$(head -c 4 "$TMP")" != "xar!" ]]; then
    echo "byper: downloaded file is not a valid installer package." >&2
    exit 1
fi

echo "==> byper: opening the installer (the macOS installer will ask for admin)."
open "$TMP"

cat <<'EOF'

==> Next steps:
    1. Choose "Install" in the installer window (your settings are kept).
    2. byper opens by itself when the install finishes.

    NOTE: bypass needs SIP's debug restriction relaxed ONCE in Recovery:
        csrutil enable --without debug
    (keeps filesystem/NVRAM protections; only the debugger attach policy is
    relaxed). Re-enable fully later with `csrutil enable` if you ever want to.

    Docs: https://github.com/Unnidentified/byper#readme
EOF
