# RustDesk Server service for Kubernetes on Wodby

Run the open-source RustDesk server as a reusable Kubernetes application service with Wodby.

This repository defines the Wodby service manifests and operational
configuration for RustDesk Server.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [RustDesk Server application stack](https://github.com/wodby/stack-rustdesk)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `rustdesk` |
| Type | Application service |
| Versions | `1` by default |
| Workloads | `main` (Statefulset, primary) |
| Containers | `rustdesk` using `wodby/rustdesk` |
| Endpoints | `rustdesk`: HTTP 8080 (main), TCP 21116, UDP 21116, TCP 21117, TCP 21115 |
| Service links | None |
| Application build | Not buildable from application source |
| Helm | chart `oci://registry-1.docker.io/wodby/stateful`; version `0.2.0` |
| Configuration and operations | 1 volumes, 1 actions |

## Connect devices

RustDesk devices connect to published TCP and UDP ports, not to the HTTPS domain:

1. Publish the `id` and `id-udp` ports. They need the same public port number, which Wodby assigns
   automatically on k3s and single-node clusters.
2. Publish the `relay` port.
3. Redeploy the service.
4. Run the **Show client settings** action and enter the ID server, relay server and key in each RustDesk client
   under **Settings > Network > ID/Relay server**.

Every session goes through the relay, because the cluster's gateway hides each device's own address.

## Use this service

Use this service through [RustDesk Server application stack](https://github.com/wodby/stack-rustdesk), or reference
`rustdesk` from a custom Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id> --include GUIDANCE.md
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
