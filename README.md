# Desktop-Setup
This repo is a guide to setup desktop as a remote server for coding, entertainmant, etc. It includes two parts, server setup in `server.md` and user setup in `user.md`.

The idea is that each user using this desktop should create a separate account with user specific software suite. All user configuration files should be inside user's directory e.g., `/home/XXX`, `/home/XXX/.config` instead of system directory such as `/etc`, `/usr/local/`.

Whenever the system needs reinstallation/restore, only the system directories needs to be wiped out and follow `server.md` to newly install a fresh system.
