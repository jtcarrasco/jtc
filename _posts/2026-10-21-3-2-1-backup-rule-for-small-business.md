---
layout: post
title: "The 3-2-1 Backup Rule: How to Actually Protect Your Business's Files"
description: "The simple backup rule photographers have trusted for years, what it looks like for a small business, and the one habit that keeps ransomware from taking your backups too."
date: 2026-10-21 08:00:00 -0700
published: true
image: /assets/blog/3-2-1-backup-rule-for-small-business.webp
tags: [tech-training, backup, security, small-business, beginner]
---

Nobody thinks about backups until the day they need one. A laptop gets lifted out of a car, a hard drive starts making that clicking noise it's never supposed to make, or ransomware locks every file on the office computer and asks for money to give them back. At that point the only question that matters is whether you have a copy somewhere else.

Well, here's the good news. You don't need an IT department for this. You need one simple rule and about an hour of setup.

## What "3-2-1" actually means

The rule comes from photographer [Peter Krogh](https://thedambook.com/), who laid it out in *The DAM Book* for people whose whole livelihood lived on hard drives. It's been the standard backup advice ever since, mostly because it's simple and it works:

- **3 copies of your data:** the original, plus two backups
- **2 different types of storage:** not two folders on the same computer (nice try)
- **1 copy offsite:** somewhere other than your office, in case of fire, theft or flood

That's it. If one bad day (spilled coffee, stolen laptop, house fire) can wipe out every copy you have, you don't have a backup. You have a false sense of security.

## The simple version for most small businesses

For a solo operator or a small office, 3-2-1 looks like this:

1. **Original:** the working copy on your computer.
2. **Local backup:** an external hard drive, or a NAS if you have one (more on that below). This is your fast restore when a drive dies.
3. **Offsite backup:** an automatic cloud backup service. This is the copy that survives if something happens to the building itself.

For the offsite copy, [Backblaze](https://www.backblaze.com/cloud-backup) is the easiest option I know of. It's $99 a year per computer, it runs quietly in the background, and it backs up everything on its own. No folders to remember, no manual uploads. If a laptop gets lost or stolen, you restore everything to a new machine, or Backblaze will even mail you a hard drive with your files on it.

## Got a few computers or a shared drive? Look at a NAS

Once your business has grown past one laptop (a couple of employees, or you're tired of hunting for which computer has the "current" version of the proposal), a **NAS** is worth a look. That stands for network-attached storage, which is a fancy way of saying a small server that lives in your back room.

Everyone saves to it like a shared drive. [Synology](https://www.synology.com/) makes small units for a few hundred dollars that are built to be set up without a networking background. Once it's running you get two things at once:

- Everyone works off the same files, so no more emailing documents back and forth
- You back up one device instead of five separate laptops

The NAS becomes your local copy (a different type of storage than the computers, check). Most of them can also sync to cloud storage like [Backblaze B2](https://www.backblaze.com/cloud-storage) overnight, which covers your offsite copy without anyone having to remember to do it.

This is a bigger step than running Backblaze on one laptop, and not everyone needs it. But if file sharing has become a daily headache, it pays for itself in sanity.

## Cloud storage is not the same as backup

This one trips people up constantly. Having your files in Dropbox, Google Drive or OneDrive is not the same as having a backup.

Those tools sync your files. They don't protect them. If ransomware scrambles the files on your computer, or you delete something and don't notice for a few weeks, the sync tool faithfully copies that damage up to the cloud too. A real backup keeps history, so you can go back to a version from before the problem. Backblaze, for example, keeps 30 days of file history by default, so an accidental delete or a ransomware attack doesn't mean the files are gone for good.

If you already use Google Workspace or Microsoft 365, that's a fine place for day-to-day file sharing. Just don't count it as your only backup.

## Keep one copy ransomware can't reach

Here's the catch the original rule didn't have to worry about: ransomware doesn't stop at the computer it lands on. It goes looking for anything else it can reach. An external drive that's always plugged in, or a NAS that's always on the network, gets encrypted right along with your laptop. Three copies don't help much if all three get scrambled by the same attack.

Two habits close that gap:

1. **Unplug the local backup when it's done.** A drive sitting in a drawer can't be infected. To make it painless, buy two drives and rotate them: one gets plugged in for the weekly copy, the other sits somewhere safe, and you swap them each time.
2. **Make sure your cloud backup keeps old versions.** If ransomware encrypts your files and the backup copies the encrypted versions up, version history is what gets you back to last Tuesday. Check that it's turned on and how far back it goes.

## What this looks like at my house

I'd be a hypocrite if I didn't show you mine, so here it is. Everything in our house (photos, documents, the family's shared files) lives on a Synology NAS. That's copy one. I back the NAS up to two 4TB external drives that I rotate: one gets loaded, then goes into a fireproof safe while the other comes out for the next round. That's copy two, and because one drive is always unplugged and locked away, it's also my ransomware insurance. Backblaze runs its own backup on a schedule, which is copy three, offsite, and I don't have to remember a thing.

The fireproof safe is the part people skip. It costs less than a nice dinner out, and it means a kitchen fire takes out the toaster... not ten years of family photos.

## Build it yourself: the 30-minute version

If you do nothing else after reading this, do this today:

1. Sign up for [Backblaze](https://www.backblaze.com/cloud-backup) on your main work computer.
2. Let the first backup run overnight. It can take a while the first time, so don't panic when it says "2 days remaining."
3. Buy a cheap external hard drive, copy your most important folders to it, then unplug it and keep it somewhere other than right next to the computer.

That's a real 3-2-1 setup: your computer, the external drive, and Backblaze offsite. About thirty minutes of actual effort, and it runs itself after that. Heck, set a monthly reminder to swap the drive and you're ahead of most small businesses out there.

Then go have some lunch. Your files will be fine.

---

*Not sure whether your business needs a NAS or just cloud backup? I offer one-on-one sessions in person (South Bay) and remotely. [Get in touch](/contact/) and we'll figure out the right setup together.*
