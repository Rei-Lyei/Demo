# Livestream verification — 8 October 2026

## Passed

- JavaScript syntax checks for the existing script and new live module.
- Headless Chromium with real browser MediaDevices, MediaRecorder, IndexedDB and RTCPeerConnection APIs. A synthetic camera/microphone was used, not the user's physical devices.
- Host preview and camera/microphone permission denial feedback.
- Actual media received and played in a separate browser context through native WebRTC on the same test machine.
- Viewer comment delivered to host; seventeen repeated heart taps persisted and displayed.
- Host microphone toggle.
- Host ending a stream creates a recording archive; downloaded MediaRecorder parts concatenate into a playable replay.
- Simulated storage failure preserves recording parts in IndexedDB. Refreshing the browser, restoring uploads, and choosing Recover recording completes the archive. Its recovered recording plays.
- Desktop and phone layout screenshots inspected.
- Existing Supabase database tested with authenticated host/viewer roles inside a transaction: atomic likes, host-only edits, commenter identity, bounded reaction RPC arguments, authorized signaling, chat/reaction mute enforcement, private ended stream/chat metadata, private recording manifests, private Storage objects, and upload ownership. All fixture changes were rolled back.
- Every newly added table has RLS enabled. Security advisors report no new live-table findings. Performance advisors report no missing indexes or policy warnings on the new tables. Unused indexes on fresh tables are expected until real traffic arrives.

## Test limits and remaining setup

The browser tests used a shared simulated Supabase transport/storage backend to avoid creating user accounts or storing test videos in the real project. They used the actual WebRTC/recording browser APIs. Database authorization was tested separately against the real Supabase project. These checks do not prove end-to-end delivery through real Supabase sessions, real hardware, or different internet networks.

A TURN relay has not been purchased, deployed, or configured. Its authenticated Edge Function source and configuration hook are included. The default STUN-only connection cannot guarantee video delivery through restrictive campus networks, mobile carriers, or firewalls. Follow LIVESTREAM_GUIDE.md and test two real accounts on different networks before relying on livestreaming publicly.

This version caps streams at eight viewer connections and does not include an SFU, provider-managed server recording, background recording after the host tab closes, or automatic server-side transcoding. Recording uploads/replay consume the existing Supabase project's storage and bandwidth. Very large recordings can exceed the browser's available disk or replay memory.

## Existing project findings

These findings predate the livestream implementation and remain in the existing account/video system:

- Legacy hashtag, sound and live_like tables have RLS but no policies. The new implementation uses separate tables. Reference: https://supabase.com/docs/guides/database/database-linter?lint=0008_rls_enabled_no_policy
- Leaked-password protection is disabled. Reference: https://supabase.com/docs/guides/auth/password-security#password-strength-and-leaked-password-protection
- The earlier START_HERE.md documents public profile-table field exposure and existing public video Storage concerns. These require a separate review before public release.
