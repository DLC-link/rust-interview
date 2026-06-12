
**App brief — Decentralized Party Explorer**

Build a small internal tool in Rust that queries a live network's topology over gRPC and answers one question: **where do the decentralized parties live?** — which nodes host each one, with what permission. You'll be given a bundle of `.proto` files and a gRPC endpoint to a running network. Generate Rust bindings, fetch the decentralized parties, find their hosting nodes, persist your findings, expose them over a REST API, and visualize them in a minimal frontend. A table is acceptable; a party↔node graph is better. The whole thing must run end-to-end with `docker compose up`.

**Where to look**

The relevant proto is `topology_manager_read_service.proto`. Two RPCs matter:

1. `TopologyManagerReadService.ListDecentralizedNamespaceDefinition` — one result per decentralized party. On `item` you get `decentralized_namespace` (the party's namespace fingerprint). There are only a few hundred of these on the network.
2. `TopologyManagerReadService.ListPartyToParticipant` — the hosting data. Each result links a party to the nodes hosting it and their permission (submission / confirmation / observation). Party IDs have the form `name::namespace-fingerprint` — match the fingerprint against step 1.

Every `List*` RPC takes a `BaseQuery`; the minimum valid request body is:

```
{ "base_query": { "store": { "synchronizer": { "physical_id": "<id>" } }, "head_state": {} } }
```

Call `ListAvailableStores` first to discover the synchronizer's `physical_id`.

Note: `ListPartyToParticipant` returns the **full** hosting set for the network — tens of thousands of entries, almost all irrelevant to you. Decide what you fetch, what you keep, and what you persist.

**Stretch**

Decentralized parties are governed by multiple owner namespaces with a signing threshold — `item.owners` and `item.threshold` on the definition. Add the governance dimension to your view: for each decparty, show its owners and threshold alongside its hosting nodes.
