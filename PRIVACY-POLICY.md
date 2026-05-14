# PRIVACY-POLICY

User-facing privacy policy. Renders at `/privacy`. Atemporal, simple.

---

## What data we collect

**If you don't sign in**: nothing identifying. We store the snapshots you save (the contents of your sim state) under a content-addressed hash. Your browser keeps a list of those hashes in localStorage so you can find them again on this device.

**If you sign in**: your email address and display name from Google sign-in. We store this to give you a list of "your saves" across devices, nothing else.

**Aggregate visit counts**: we count page visits to understand which parts of the tool get used. No identifiers, no cookies, no cross-site tracking. Respects Do Not Track and Global Privacy Control. Powered by self-hosted Plausible.

## What we don't do

- No cookies for tracking
- No third-party analytics (no Google Analytics, no Mixpanel, no Segment, no Hotjar, no FullStory)
- No advertising
- No retargeting pixels
- No selling your data
- No emailing you
- No notification spam
- No session recording

## Where data lives

All data lives on our self-hosted servers. We use Cloudflare for content delivery (no personal data passes through Cloudflare beyond the standard HTTP request headers needed to route traffic).

## How long we keep data

- Anonymous snapshots: kept forever or until someone flags them as abuse. Content-addressed = deduplicated. Identical content = single stored copy.
- Signed-in user data: kept until you ask us to delete it. Email and display name only.
- Aggregate visit counts: kept indefinitely (no identifiers).

## Your rights

You can:
- Sign out at any time (button in nav)
- Delete your account (button in `/me`, removes user row, anonymizes any owned snapshots)
- Export your saved snapshots (download from `/me`)

## Snapshot content

Snapshots are public — anyone with the permalink can view them. Don't put personal information in your snapshot content. (We can't read snapshot bodies in normal operation — the canonical bytes are blake3-hashed and stored opaquely — but they're served on a public URL.)

## Abuse

If you find an abusive shared snapshot, the report button on the snapshot view flags it. Flagged snapshots return "gone" and are purged from edge caches. Admins review flagged queue.

## Children

We don't target anyone under 13. We don't knowingly collect data from anyone under 13. If you're a parent and believe we have such data, contact us via the disclosure address in `SECURITY.md`.

## Changes

Material changes to this policy land in the git repo with a commit. We don't have your email; we can't notify you. The current state of this file is the current state of the policy.

## Contact

Reach the maintainer via the disclosure address in `SECURITY.md` for any privacy concern.
