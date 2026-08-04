# Setting Up Kali Linux on an M1 MacBook Pro (The Honest Version)

If you've searched for "install Kali Linux on Apple Silicon," you've probably found a dozen guides that make it look simple. It mostly is, but a few things caught me off guard along the way, so I'm writing down what actually happened, bugs included, in case it saves someone else the same confusion.

## Why I'm doing this

I'm moving from IT support into cybersecurity, and one of the first steps was building a home lab. Step one of that lab: get Kali Linux running somewhere I can safely practice on. I've got an M1 Pro MacBook, so this is that specific setup.

## The tools

- **UTM** (free, open source virtualization app built for Apple Silicon)
- **Kali Linux ARM64 Installer image** (from kali.org)

## Mistake #1: Wrong download page

Kali's site has multiple tabs for downloads: Installer, Pre-built VMs, ARM, Mobile, Cloud, and so on. My first instinct was to check "Pre-built VMs," since that sounded like the fastest route. Those are x86-64 images though, built for Intel and AMD processors, not Apple Silicon.

Next stop was the "ARM" tab, which also turned out to be the wrong one. That section is for physical ARM hardware like Raspberry Pi devices, not for running Kali as a VM on an M1 Mac.

**The right place:** the "Installer" tab, choosing the Apple Silicon (ARM64) image specifically. This gives you an ISO file rather than a ready-made VM, so UTM installs Kali from scratch. A few more steps than a pre-built image, but it's the correct path for Apple Silicon.

## Setting up the VM

Basic specs I used:

- 2 CPU cores
- Around 4GB RAM
- 64GB storage (generous, but I had the space and didn't want to think about resizing later)

## Mistake #2: The black screen bug

After booting the VM for the first time, I got a black screen with a blinking cursor. Nothing else. I let it sit for a couple of minutes assuming it just needed time, but it was actually a known UTM issue with Kali specifically, not something wrong on my end.

**The fix:**

1. Shut down the VM
2. Go into VM Settings, under Devices
3. Remove the Display device
4. Add a new Serial device in its place
5. Save and restart the VM

This swaps how UTM shows the VM's output, from a graphical display (buggy during Kali's install) to a text based serial console (reliable). Once I did this, the installer booted properly into Kali's text based install menu.

## The actual install

From here it was fairly standard Debian-based installer steps:

- Guided partitioning, using the entire virtual disk (safe, since this only touches the VM's virtual disk, not the Mac's real storage)
- Default software selection: Xfce desktop environment plus the "top10" tool collection

This stage took a while, it's downloading and installing packages, so it's a good point to walk away for ten or twenty minutes.

## Mistake #3: The install loop

After the install finished, I restarted the VM expecting to see Kali. Instead, it booted straight back into the installer again.

This wasn't a failed install, it was the VM still pointing at the installer ISO as its boot device instead of the newly installed system on the virtual disk.

**The fix:** in UTM, the ISO wasn't listed under a typical CD/DVD device (which is what most guides assume). On this ARM64 VM, it showed up as a USB Drive instead. Deleting that drive removed the installer from the boot sequence, so the VM booted into the actual installed system on restart.

## Mistake #4: Stuck in text mode

Once past the boot loop, I was in Kali, but still in the plain text console from the Serial device fix earlier, not the graphical desktop.

**The fix:** shut down, go back into Settings, add the Display device back (the one removed earlier to fix the black screen bug). The bug was specific to the install phase, so a standard display works fine now that Kali is actually installed. Boot again, and this time the graphical Xfce login screen showed up properly.

## What I'd tell someone starting this today

- Go straight to the Installer tab on kali.org, ARM64 image, don't bother with Pre-built VMs or the ARM tab if you're on Apple Silicon
- If you hit a black screen during install, it's the Serial device fix, not a broken download
- If it loops back to the installer after finishing, check for the ISO under USB Drive rather than CD/DVD
- Swap the Display device back once installation is actually complete

None of this is complicated once you know it, but I spent a good chunk of an evening figuring out things that could've been a five minute fix with the right information up front. Hopefully this saves someone else that evening.

---

*Next up: setting up a target environment (likely OWASP Juice Shop via Docker, since Metasploitable2 doesn't have ARM support) to actually start practicing against.*
