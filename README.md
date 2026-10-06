# elastic-fruit-protocol

The wire protocol between an [elastic-fruit-runner](https://github.com/boring-design/elastic-fruit-runner) agent and an Elastic Fruit Cloud server. This repository holds the protobuf definition and the generated Go code, and nothing else, so that the agent and the server can both import it without depending on each other.

## What is in here

* `proto/agent/v1/agent.proto` defines `AgentService`.
* `gen/agent/v1` holds the generated protobuf messages.
* `gen/agent/v1/agentv1connect` holds the generated [Connect](https://connectrpc.com) client and handler.

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

## License

Apache-2.0, see `LICENSE`.
