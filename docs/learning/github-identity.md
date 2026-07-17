# GitHub Identity — matching commits to the project's branding

## What I spotted (Task 3)

While working through the governance documents, I noticed my commits were being stamped with my personal Gmail address as the author. On a **public, recruiter-facing repository**, the commit author is visible on every single commit — so the whole history was carrying a personal address that didn't match the project's professional branding.

## The two identities involved

It's worth being clear that there are *two* separate identities, and they're easy to confuse:

1. **Git commit identity** (`user.name` / `user.email`) — set by Git config on my machine. This is what gets baked into each commit as the author. It has nothing directly to do with GitHub.
2. **GitHub account email** — the verified email(s) attached to my GitHub account. For GitHub to *link* a commit to my profile (show my avatar, count the contribution), the commit's email must match a verified email on my account.

They interact: if I set the commit email to my domain address but *don't* also verify that address on GitHub, the commits won't link to my profile.

## What I did (fixed later)

1. Added `ioannis@theitchef.com` to my GitHub account and **verified** it (the verification link goes to the mailbox — which is why the mailbox had to be live first).
2. Set the identity **per-repo** (local config), not globally — so this project uses the domain address while my other repos keep their own default:
git config --local user.email "ioannis@theitchef.com"
git config --local user.name "Ioannis Mintzivyris"

## The catch worth remembering

Changing the identity only affects **future** commits. Every commit made before the change keeps its old author stamp permanently — authorship is baked in at commit time and isn't worth rewriting history to change. So the clean move is to set the right identity *early*, before a repo accumulates commits under the wrong one.

## One-line takeaway

Set a per-repo commit identity that matches the project (and verify that email on GitHub so commits link to your profile) — and do it early, because it only fixes future commits.