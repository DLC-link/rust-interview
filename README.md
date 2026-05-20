# rust-interview

## App brief — Canton Topology Viewer
Build a small internal tool in Rust that queries a Canton participant’s topology over gRPC, persists
it, and renders it in the browser. You will be given a bundle of Canton .proto files and a running
mock Canton at a known endpoint. Generate Rust bindings from the protos, connect, fetch the
topology (parties, participants, hosting relationships), store the snapshot in the database, expose
it over a REST API, and visualize it in a minimal frontend. The whole thing must run end-to-end with
docker compose up.
