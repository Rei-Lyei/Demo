# New: livestreams

Read `LIVESTREAM_GUIDE.md` first for camera broadcasting, chat, repeated likes, and Settings → Past Livestreams. The livestream database/storage setup is already applied. Internet video connectivity still needs a TURN relay for restrictive networks.

# ViralHub — Supabase setup and testing

Your system is already configured for the ViralHub Supabase project (`wrnkilgtuhprfxoqlzqq`). You do not need a new project or database.

## 1. Open the updated system

1. Extract this ZIP.
2. Open the ViralHub folder in VS Code.
3. Open `viralhub.html` with Live Server. Alternatively, open a terminal inside the folder and run `python -m http.server 5500` (on Windows you can use `py -m http.server 5500`).
4. Visit `http://localhost:5500/viralhub.html`. Use the same URL consistently. Do not run it by double-clicking the HTML file.
5. Keep an internet connection: the app loads the pinned Supabase JavaScript SDK and connects to your hosted project.

## 2. Configure email confirmation

1. Open your ViralHub project in the Supabase dashboard.
2. Open Authentication → URL Configuration.
3. For local testing, set Site URL to `http://localhost:5500/viralhub.html` and add the exact same URL to Redirect URLs. If Live Server uses `127.0.0.1`, add that exact URL too.
4. When you publish later, add the actual website URL and set it as Site URL.
5. Keep email confirmation enabled. Register with an email you can open, click its confirmation link, then log in.
6. The app creates the `users` profile after an authenticated session exists. It does not attempt a profile insert while confirmation is pending. If the email does not arrive, check spam and Supabase Authentication logs/SMTP settings.

## 3. Test with your real account

1. Register with a unique username using 3–30 letters, numbers, or underscores and a password of at least eight characters.
2. Confirm your email and log in. Your name and username should appear in Profile and Settings.
3. Upload a small video. It should appear in Home, Discover, and your Posts grid.
4. Like it and open Liked videos. Post a comment and reopen the comments sheet.
5. Use another real account to test following, unfollowing, creator pages, followers, and mutual friends.
6. Change your bio and profile photo, save, then refresh. Both should remain saved.
7. Log out, log in as another user, and verify that account's profile loads.

## 4. What changed

- Removed six hardcoded creators, sample videos, sample comments, fake follower lists, fake profile thumbnails, fake notification entries, and the fake inbox badge.
- Discover lists database profiles, public video thumbnails, and hashtags found in real captions.
- Posts and Liked grids read the current account's database records.
- Followers, Following, and Friends use the existing `following` table. Friends are mutual follows.
- Fixed an undefined `user` variable that stopped profile loading.
- Login and restored sessions reload the current account's profile and data.
- Signup handles pending email confirmation and creates missing profiles after login.
- Profile saving now checks database errors. Profile email is synchronized from confirmed Supabase Auth email rather than prematurely writing a pending email change.
- Supabase SDK version is pinned to 2.117.3.

## 5. Dummy accounts: awaiting identification

No Supabase accounts or database rows have been deleted. These are the existing usernames found during inspection:

| Username | Uploaded videos |
| --- | ---: |
| testuser4 | 1 |
| kc_pogi | 0 |
| TestUser | 1 |
| Lyei | 1 |

Tell me exactly which usernames should be deleted. Their related videos, comments, likes, follows, reports, and stored uploads need a dependency review before deletion. A test-looking name alone is insufficient to identify a dummy account. Removing the hardcoded display users did not delete these real Supabase Auth records.

## 6. Remaining features and database concerns

- Notifications, messaging, and self-service account deletion are unfinished. Livestreaming now has a native WebRTC implementation; see `LIVESTREAM_GUIDE.md` for its networking setup and audience limit. No notifications table was added.
- Category choices are UI only; the existing video table has no category field. View counts are displayed from the database; this update does not add view tracking.
- Public `users` SELECT policies currently expose the whole users table, including email, phone number, and birthdate. Password values inspected are placeholders, not real passwords. Before public release, separate private account fields from public creator profiles and restrict access to the private fields. Client-side column selection alone cannot enforce privacy.
- The videos bucket is public. A private database row does not make a public Storage URL private. Private video support requires private storage and signed URLs.
- Storage upload ownership and username changes need further hardening: several foreign keys reference username, so renaming a username with related rows may be rejected. This update reports the error rather than claiming success.
- The livestream update added separate `vh_` tables, access policies, and a private recording bucket. Existing legacy policies/data were preserved. The database advisor still reports no policies for legacy `hashtag`, `sound`, and `live_like`; the new live feature uses its own tables.

## Verification

JavaScript syntax was checked. Database/project, table structure, existing accounts, RLS policies, and buckets were inspected through Supabase. Automated DOM checks use a simulated Supabase response and cover account initialization and email-confirmation handling. Actual login, email delivery, and upload testing with your account still need the steps above; your credentials were not requested or used.

Documentation: https://supabase.com/docs/guides/auth/passwords
Privacy/RLS: https://supabase.com/docs/guides/database/postgres/row-level-security
