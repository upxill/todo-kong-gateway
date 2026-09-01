# Todo API — Kong Gateway on Konnect

A declarative Kong Gateway configuration that fronts a sample TODO service (backed by [jsonplaceholder](https://jsonplaceholder.typicode.com)) and deploys it to **Kong Konnect** — Kong's managed cloud control plane — via decK, with a GitHub Actions pipeline that validates, diffs, and applies changes on every push.

## How this differs from my other Kong/gateway repos

This is the repo in the cluster that targets **Konnect** (Kong's SaaS control plane) rather than a self-run Kong node, and the CI pipeline is built around `deck gateway diff` for change previews before syncing — a workflow the other declarative-config repos (`kong-openapispec2proxy`, `kong-ai-gateway`) don't use. It also demonstrates a different plugin combination — `key-auth` plus `proxy-cache` — versus the AI-focused plugin pipeline in `kong-ai-gateway` or the MCP tool conversion in `kong-openapispec2mcp`. Where `Azure-Kong-AI-Gateway-IaC` provisions the infrastructure a data plane runs on, this repo is a small example of the service-level configuration Konnect would push down to that data plane.

## Tech Stack

- Kong Gateway, deployed to Kong Konnect (managed control plane)
- decK CLI (`validate`, `diff`, `sync`)
- `key-auth` and `proxy-cache` Kong plugins
- GitHub Actions (`kong/setup-deck@v1`, pinned to decK 1.39.3)

## How it works

1. **`services.yaml`** declares `todo-service`, pointing at `https://jsonplaceholder.typicode.com`, with a route (`todo-route`) exposing `/todos` (`strip_path: false` so sub-paths like `/todos/1` pass through).
2. **`consumers.yaml`** declares one consumer, `upxill-dev`, with a `key-auth` credential.
3. **`plugins-auth.yaml`** attaches `key-auth`, requiring an `apikey` header on every request.
4. **`plugins-cache.yaml`** attaches `proxy-cache` (in-memory strategy, 600s TTL, only caching `200` JSON responses) so repeated `GET /todos` calls are served from cache instead of hitting the upstream every time.
5. **CI/CD** (`.github/workflows/deploy-gateway.yaml`): on every push and PR to `main`, `deck gateway diff kong-gateway/` previews the change against the `quickstart` Konnect control plane; on a push to `main`, `deck gateway sync kong-gateway/` applies it, authenticated with a `DECK_KONNECT_TOKEN` repo secret.

## Getting Started

Prerequisites: [decK](https://docs.konghq.com/deck/) CLI and a Kong Konnect account/control plane.

```bash
# Install decK (match the pinned CI version, 1.39.3)
brew install deck
deck version

# Validate and preview locally
deck gateway validate kong-gateway/
deck gateway diff kong-gateway/

# Apply to Konnect (requires DECK_KONNECT_TOKEN)
export DECK_KONNECT_TOKEN=<your-konnect-token>
deck gateway sync kong-gateway/
```

For CI, set `DECK_KONNECT_TOKEN` as a repository secret; optionally set `DECK_KONNECT_CONTROL_PLANE_NAME` if you're not using the default `quickstart` control plane.

## Project Structure

```
.
├── README.md
├── .github/workflows/deploy-gateway.yaml   # decK validate/diff/sync CI pipeline against Konnect
└── kong-gateway/
    ├── services.yaml       # todo-service + /todos route
    ├── consumers.yaml      # upxill-dev consumer + key-auth credential
    ├── plugins-auth.yaml   # key-auth plugin config
    └── plugins-cache.yaml  # proxy-cache plugin config
```
