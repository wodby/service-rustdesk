# RustDesk Server on Wodby

What Wodby sets up for the open-source RustDesk server on this service. It runs the `wodby/rustdesk` image: the ID server (`hbbs`) and the relay (`hbbr`) in one container, plus a small web server on port 8080 that serves both over WebSocket on the service's main domain, a status page with setup steps, and a health check. The image patches the ID server to sign devices in over WebSocket, which the open-source release refuses.

## Set up devices

Devices connect over WebSocket to the main domain's HTTPS address, so nothing needs to be published and it works on every cluster.

Run the service action "Show client settings". It prints what to enter in the RustDesk app on every device, under Settings → Network:

1. Turn on **Use WebSocket**.
2. Under ID/Relay server: the domain as the **ID server**, **Relay server** empty, `https://<domain>` as the **API server**, and the **Key**.

The API server entry is what makes the app use secure WebSocket (`wss://`); without it the app tries plain `ws://` and the domain's HTTPS redirect breaks it. The server has no API, so the app's requests there get 404, which is harmless.

The key is the server's public key. Devices without it can't use the server. It is created on first start and stored on the volume. The status page on the domain shows the setup without the key.

## Devices without WebSocket (optional)

On k3s and single-node clusters, devices can also connect without WebSocket through published ports:

1. Publish `id` and `id-udp` (both container port 21116). Devices sign in over UDP and connect over TCP on the same address and port, so both need the same public number. The second one published gets the first one's number automatically. On other clusters, `id-udp` gets a different number and this mode doesn't work.
2. Publish `relay` (container port 21117).
3. Deploy the service again. "Show client settings" then also prints the port-based values.

`nat` (container port 21115) only shows which devices are online, and only when published with the number one below `id`. Sessions work without it.

## How sessions connect

Every session goes through the relay, so its bandwidth comes from the cluster. WebSocket sessions always relay, and Wodby's gateway hides devices' own addresses from port-based sessions, so the service also sets `ALWAYS_USE_RELAY=Y`. The relay limits each session to 128 Mbit/s and all sessions together to 1024 Mbit/s by default. Change that with the `SINGLE_BANDWIDTH` and `TOTAL_BANDWIDTH` variables (Mbit/s).

A device signed in over WebSocket gets a heartbeat every 10 seconds and counts as offline 45 seconds after it stops answering.

## Data

The `data` volume is mounted at `/data`. It holds the key pair (`id_ed25519`, `id_ed25519.pub`) and the device database (`db_v2.sqlite3`). Losing the key pair means every device has to be set up again with a new key.

Run one replica only.

## Check the result

The container log shows `Key: ...` from both servers at start and `Peer <id> signed in over websocket` for each device. `curl -fsS http://localhost:8080/healthz` inside the container answers `OK`.
