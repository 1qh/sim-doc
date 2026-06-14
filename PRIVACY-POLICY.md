# PRIVACY-POLICY

User-facing privacy policy. Renders at `/privacy`. Atemporal, simple.

---

## What data we collect

**Nothing identifying.** There are no accounts, no login, no sign-in. We never ask for your name, email, or any personal information, and we have no server that stores anything about you.

**Snapshots** you save live in your own browser's localStorage on this device. They never leave your browser unless you choose to share one.

**Shared snapshots** are encoded entirely into the URL you copy — the state is compressed and packed into the link's fragment. Nothing about a shared snapshot is uploaded to or stored on any server.

**Aggregate visit counts**: we count page visits to understand which parts of the tool get used. No identifiers, no cookies, no cross-site tracking. Respects Do Not Track and Global Privacy Control. Powered by cookieless Plausible.

## What we don't do

- No accounts, no login, no sign-in
- No cookies for tracking
- No third-party analytics (no Google Analytics, no Mixpanel, no Segment, no Hotjar, no FullStory)
- No advertising
- No retargeting pixels
- No selling your data — we have no data about you to sell
- No emailing you
- No notification spam
- No session recording

## Where data lives

Your saved snapshots live in your browser's localStorage, on your device only. Shared snapshots live entirely inside the URLs you share. The only thing we run on a server is the static site delivery and cookieless aggregate visit counting. We use Cloudflare for content delivery (no personal data passes through Cloudflare beyond the standard HTTP request headers needed to route traffic).

## How long we keep data

- Your saved snapshots: kept in your browser until you delete them or clear your browser storage. We never see them.
- Shared snapshot URLs: persist as long as the link exists — the data is the link.
- Aggregate visit counts: kept indefinitely (no identifiers).

## Your control

- Your snapshots are yours, on your device. Delete any of them from the `/me` snapshot list.
- Clearing your browser's site data removes every saved snapshot.
- There is no account to delete, because there is no account.

## Snapshot content

A shared snapshot URL is readable by anyone you give the link to. Don't put personal information in your snapshot content if you plan to share it.

## Children

We don't target anyone under 13. We don't knowingly collect data from anyone under 13 — in fact we collect no personal data from anyone. If you're a parent with a concern, contact us via the disclosure address in `SECURITY.md`.

## Changes

Material changes to this policy land in the git repo with a commit. We don't have any way to contact you. The current state of this file is the current state of the policy.

## Contact

Reach the maintainer via the disclosure address in `SECURITY.md` for any privacy concern.
