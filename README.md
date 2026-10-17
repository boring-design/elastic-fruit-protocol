# elastic-fruit-protocol

The wire protocols of Elastic Fruit Cloud: the agent protocol between an [elastic-fruit-runner](https://github.com/boring-design/elastic-fruit-runner) agent and an Elastic Fruit Cloud server, and the public Cloud API. This repository holds the protobuf definitions and the generated Go code, and nothing else, so that the agent, the server and API clients can import them without depending on each other.

## What is in here

* `proto/agent/v1/agent.proto` defines `AgentService`.
* `gen/agent/v1` holds the generated protobuf messages.
* `gen/agent/v1/agentv1connect` holds the generated [Connect](https://connectrpc.com) client and handler.
* `proto/cloud/v1/cloud.proto` defines `CloudService`, the Cloud API.
* `gen/cloud/v1` and `gen/cloud/v1/cloudv1connect` hold its generated messages, client and handler.

Every RPC is started by the agent. The server never dials the agent. Commands from the server to the agent travel back over the server stream opened by `WatchCommands`. See the comments in the proto file for the full contract.

## Using it from Go

```sh
go get github.com/boring-design/elastic-fruit-protocol@latest
```

```go
import (
    agentv1 "github.com/boring-design/elastic-fruit-protocol/gen/agent/v1"
    "github.com/boring-design/elastic-fruit-protocol/gen/agent/v1/agentv1connect"
)
```

The agent uses `agentv1connect.NewAgentServiceClient`. The server implements `agentv1connect.AgentServiceHandler` and mounts it with `agentv1connect.NewAgentServiceHandler`.

## Cloud API (`cloud.v1.CloudService`)

`CloudService` is the public API of Elastic Fruit Cloud. The dashboard, the [efc CLI and MCP server](https://github.com/boring-design/elastic-fruit-cli) and scripts all use it. Every method is a unary Connect call: `POST <API URL>/cloud.v1.CloudService/<Method>` with JSON or binary protobuf.

Calls carry either a browser session or a tenant API token as `Authorization: Bearer efc_<token>`. A token belongs to one tenant and has the scope `read` or `admin`. Requests may leave `tenant_id` empty, the server then uses the tenant of the token.

```go
import (
    cloudv1 "github.com/boring-design/elastic-fruit-protocol/gen/cloud/v1"
    "github.com/boring-design/elastic-fruit-protocol/gen/cloud/v1/cloudv1connect"
)
```

## Regenerating

Install [buf](https://buf.build/docs/installation), then run:

```sh
buf lint
buf generate
```

Generated code is committed. CI fails when the committed code does not match the proto file.

## Versioning

Once a version is tagged, changes to the proto must be additive. Add new fields, messages and RPCs. Do not rename, renumber or remove anything that exists in a tagged version. CI runs `buf breaking` against `main` to catch mistakes.

A new tag is cut whenever the proto changes. Both the agent and the server pin a tag in their `go.mod`.

v0.3.0 adds the Cloud API, `proto/cloud/v1/cloud.proto`.

v0.2.0 adds the `isolation` field on `EnrollRequest` and `HeartbeatRequest`, the `runtime` field on `StartRunner` and the `runtimes` list on `BackendCapability`, so the server can schedule pool runners on hosts that offer gVisor or VM isolation.

## License

Apache-2.0, see `LICENSE`.
