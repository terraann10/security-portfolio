# OWASP Juice Shop: First Steps and the Defender's View

**Date:** 21 September 2026
**Lab:** Kali Linux (attacker) and Debian "Docker-lab" (target) on VirtualBox, connected over an Internal Network (`intlab`)
**Target:** OWASP Juice Shop running in Docker on the Debian VM, reached from Kali at `http://192.168.50.11:3000`

## Goal

Get the lab running again after an environment change, complete a first Juice Shop challenge, and check what the target actually records, so the exercise is useful from a defender's point of view and not only an attacker's.

## What I did

1. Brought the lab back after my external drive changed letter (D: to E:). VirtualBox stores absolute paths, so the VMs showed as inaccessible until they were re-attached.
2. Confirmed the network survived: static IPs on `intlab`, internet via the separate NAT adapter, and VM-to-VM ping.
3. Ran a system update on the Debian VM. The first `apt upgrade` failed with an interrupted dpkg error, which points to a half-finished earlier install. The fix suggested by the error message is `sudo dpkg --configure -a`. I ran it, then re-ran `apt upgrade`, and the update completed successfully.
4. Found the Juice Shop container stopped (`Exited (255)`), most likely because the VM went down uncleanly. Restarted it with `docker start juice-shop` and confirmed it was up with `docker ps`.
5. Loaded the shop from Kali's browser to confirm the whole lab worked end to end.
6. Solved my first challenge, **Score Board** (1 star): a page that exists in the app but is not linked from any menu.

## What the target logged (defender's view)

`docker logs juice-shop` showed:

- A normal startup sequence ending in `Server listening on port 3000`.
- Warnings that six challenges will not work as intended without an Alchemy API key or a local LLM server. These are missing optional dependencies, not faults in the lab.
- A line recording that the Score Board challenge was solved, followed by a "cheat score" line comparing my solve time (12 minutes) with the expected time (about 1 minute) and noting that no hints were used.

**Finding:** the log contains the challenge-solved lines but no entries for the individual page requests I made. I confirmed this by running `docker logs juice-shop | grep -i GET`, which returned nothing. So the app announces that a challenge was completed but, in this default setup, does not record the requests that led there.

## Why it matters for security operations

- An analyst can only investigate what is recorded. If an application does not log requests by default, visibility has to come from somewhere else, such as a reverse proxy or web server log, a WAF, or network monitoring.
- Attempts to reach hidden pages tend to show up as a burst of requests to unusual URLs. Knowing the attacker's side makes that pattern easier to spot.

## Troubleshooting notes

| Symptom | Cause | Fix |
|---|---|---|
| VMs inaccessible after drive letter change | VirtualBox stores absolute paths to VM files | Restore the original letter, or re-add each VM from its `.vbox` file at the new path |
| `apt upgrade` fails with "dpkg was interrupted" | Earlier install or upgrade did not finish | `sudo dpkg --configure -a`, then re-run `apt upgrade` |
| Juice Shop container `Exited (255)` | Host stopped abruptly | `docker start juice-shop` |

## Next steps

- Confirm whether request logging exists by default and, if not, look at adding a reverse proxy in front of the container to capture requests.
- Attempt a further 1-star challenge and record both the attack and what it looked like in the logs.
