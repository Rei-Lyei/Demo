# ViralHub livestreams

## Start using the updated system

1. Extract the ZIP and open the ViralHub folder in VS Code.
2. Open `viralhub.html` with Live Server. Alternatively run `python -m http.server 5500` in the folder (Windows: `py -m http.server 5500`).
3. Open `http://localhost:5500/viralhub.html`, sign in, and choose Create → Go live.
4. Select Enable camera. Allow camera AND microphone access. Enter a stream title, select a category, then select Go live now.
5. Keep the broadcasting tab open. Camera and microphone toggles affect the broadcast and its recording. Flip camera works on devices with a second camera.
6. Other signed-in users can open Home → Live to watch, comment, and repeatedly tap the heart. Watch from another account to test the viewer experience.
7. Select End live. Then open Settings → Past Livestreams to replay or download your private recording.

The livestream database, row access policies, Realtime publications, and private `live-recordings` bucket have already been installed in your existing Supabase project. Do not run `supabase/livestream.sql` again on that project. It is included for reference and installation on a fresh project.

## Included functions

- Camera/microphone permission flow and preview.
- Native WebRTC broadcast to up to eight viewer connections per stream.
- Live directory, viewer count, elapsed time, share link, reconnect, and host return shortcut.
- Comments, repeated heart reactions, and animated hearts.
- Host comment removal and per-stream chat/reaction muting; authors can remove their own comments.
- Only one active livestream per account. Other accounts can stream at the same time.
- Recording parts upload while streaming. Recordings are private to the broadcaster.
- Browser IndexedDB retains failed uploads. Recover recording retries them, including after refreshing the page.
- Interrupted broadcasts disappear from the directory after their heartbeat expires. The owner can recover available recordings.

## Network relay: required before relying on mobile/campus networks

This version uses native peer-to-peer WebRTC. It does not use a paid streaming platform or an SFU. Without a TURN relay, restrictive firewalls and some mobile networks can prevent viewers from receiving video even if chat works. Same-computer and permissive-network tests can work with the included STUN setting; they do not prove all internet connections will work.

For reliable internet connectivity, configure a coturn-compatible TURN relay:

1. Obtain a TURN server with its REST/shared-secret authentication enabled. Running or purchasing that server is a separate hosting step and may cost money.
2. Deploy `supabase/turn-credentials.ts` as a Supabase Edge Function named `viralhub-turn`. This function authenticates every request with Supabase Auth before returning temporary relay credentials. If disabling gateway JWT verification for modern signing keys, retain the function's `getUser(token)` check.
3. Configure Edge Function secrets `TURN_SHARED_SECRET` and `TURN_URLS`. Example `TURN_URLS` value: `["turn:YOUR_RELAY_HOST:3478?transport=udp","turns:YOUR_RELAY_HOST:5349?transport=tcp"]`. Use your real relay endpoints and supported ports.
4. Set `turnFunction: 'viralhub-turn'` in `live-config.js`.
5. Test two real accounts across different networks, including mobile data. Confirm video/audio, reconnect, chat, reactions, ending, and replay.

Never put a TURN shared secret, provider API key, or Supabase service-role key in a browser file. Only temporary TURN credentials belong in the client. This helper uses coturn's HMAC credential convention; it is not a generic adapter for all commercial relay providers.

This is a small-stream implementation with an eight-connection ceiling, not a large-audience broadcast service. Scaling to many viewers needs an SFU/media provider. No streaming provider was purchased or configured by this update.

## Recording recovery

A recording is stored as ordered media parts, not a single uploaded video. The replay page downloads and combines those parts into a playable recording. Part filenames end in `.part`; use the app's Download recording button to get a `.webm` or `.mp4` file.

When an upload fails, select Recover recording from Past Livestreams on the original device and browser, using the same website origin. IndexedDB is local to that origin. Changing `localhost` to `127.0.0.1` or changing the port creates a different local recording store. Do not clear browser/site data before recovering failed uploads.

A browser crash, device shutdown, exhausted disk space, or force-closing the tab can lose the last unfinished recording part. Available saved parts are labeled as an incomplete recording. The app does not claim that an interrupted recording is complete. Browser background suspension can also pause recording/capture.

There is no fixed stream duration limit in this interface, but browser disk/memory, host upload speed, and Supabase storage/bandwidth quotas still apply. Each part must be under 6 MiB. Replay assembly uses memory proportional to recording size. Very long recordings may need a future server-side assembly/transcoding service. Supabase usage still counts toward your existing project limits.

MediaRecorder output support varies by browser. Chrome/Edge are recommended for the first test. Some WebM recordings do not contain finalized duration/seek indexes; sequential playback and download are available, while seeking may vary by player. Live video and replay are available only over HTTPS or localhost.

## Validation

See `VERIFICATION.md` for the exact checks performed and their limits. Test with your real accounts before public use; no account credentials were collected.

## Database changes

New tables are `vh_streams`, `vh_live_viewers`, `vh_live_comments`, `vh_live_likes`, `vh_live_signals`, `vh_live_bans`, and `vh_recording_parts`. The existing legacy live tables and their records were preserved. Live metadata is visible to signed-in viewers while a broadcast is active; ended stream metadata, archived chat/reactions, recording manifests, and recording objects are restricted to the owner. Signaling is visible only to its participants.
