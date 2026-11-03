# RustDesk Server on Wodby

What Wodby sets up for the open-source RustDesk server on this service. It runs the `wodby/rustdesk` image: the ID server (`hbbs`) and the relay (`hbbr`) in one container, plus a small web server on port 8080 for the domain's status page, which shows the ID and relay server addresses (not the key) and links to the setup guide.

## Make it reachable

RustDesk devices don't use the domain's HTTPS address. They connect to TCP and UDP ports, which must be published on the app environment:

1. Publish `id` and `id-udp` (both container port 21116). Devices sign in over UDP and connect over TCP on the same address and port, so both need the same public number. The second one published gets the first one's number automatically.
2. Publish `relay` (container port 21117). Every session goes through it.
3. Optionally publish `nat` (container port 21115) with the number one below the `id` number. Clients use it to show which devices are online. Sessions work without it.
4. Redeploy the service, so the server knows its public relay address.

A TCP and a UDP port can share a public number only on k3s and single-node clusters. On other clusters, `id-udp` gets a different number, and devices can't sign in.

## Set up devices

Run the service action "Show client settings". It prints the ID server, relay server and key to enter in every RustDesk client under Settings → Network → ID/Relay server. The host is the service's main domain, which points to the cluster's address. Leave the API server empty: the open-source server has no accounts or address book.

The key is the server's public key. Devices without it can't use the server. It is created on first start and stored on the volume.

## How sessions connect

Wodby's gateway hides each device's own address from the server, so devices can't connect directly. The service sets `ALWAYS_USE_RELAY=Y`, and every session goes through the relay, so its bandwidth comes from the cluster. The relay limits each session to 128 Mbit/s and all sessions together to 1024 Mbit/s by default. Change that with the `SINGLE_BANDWIDTH` and `TOTAL_BANDWIDTH` variables (Mbit/s). The server also limits how many devices can sign in from one address, and here every device shares the gateway's address. That suits up to a few hundred devices.

"Use WebSocket" in the client is not needed. Devices with it turned on can still connect to others through `https://<domain>/ws/id` and `/ws/relay`, but nobody can connect to them: the open-source server signs devices in only over UDP.

## Data

The `data` volume is mounted at `/data`. It holds the key pair (`id_ed25519`, `id_ed25519.pub`) and the device database (`db_v2.sqlite3`). Losing the key pair means every device has to be set up again with a new key.

Run one replica only.

## Check the result

The container log shows `Key: ...` from both servers at start. `curl -fsS http://localhost:8080/healthz` inside the container answers `OK`.
