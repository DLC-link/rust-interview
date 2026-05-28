# rust-interview

## App brief — Canton Topology Viewer
Build a small internal tool in Rust that queries a Canton participant’s topology over gRPC, persists
it, and renders it in the browser. You will be given a bundle of Canton .proto files and a running
mock Canton at a known endpoint. Generate Rust bindings from the protos, connect, fetch the
network’s decentralized parties (“decparties”) and their owning namespaces, store the snapshot in
the database, expose it over a REST API, and visualize it in a minimal frontend. The whole thing
must run end-to-end with `docker compose up`.

### Where to look
The relevant proto is
`proto/community/com/digitalasset/canton/topology/admin/v30/topology_manager_read_service.proto`.
The RPC you want is `TopologyManagerReadService.ListDecentralizedNamespaceDefinition`. It returns
one `Result { context, item }` per decparty; on `item` you get `decentralized_namespace` (the
namespace fingerprint), `threshold` (signing threshold), and `owners` (the list of member
namespace fingerprints). Every `List*` RPC on this service takes a `BaseQuery` — the minimum valid
request body is:

```json
{ "base_query": { "store": { "synchronizer": { "physical_id": "<id>" } }, "head_state": {} } }
```

Use `TopologyManagerReadService.ListAvailableStores` first to discover the synchronizer’s
`physical_id`. We deliberately scope the visualization to decparties because there are only
~hundreds of them on the network — small enough to persist and render comfortably without
worrying about pagination or memory.

### Stretch
As a follow-up, fetch hosting rights with
`TopologyManagerReadService.ListPartyToParticipant` and cross-reference each decparty’s namespace
fingerprint against `item.party` (parties are `name::namespace-fingerprint`) to show which
participants host which decparties, and with what permission. Note this RPC returns the *full*
party-to-participant set for the network (tens of thousands of entries), so filter to decparties
before persisting — don’t store the whole thing.
