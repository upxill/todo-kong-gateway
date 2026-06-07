**Todo API - Kong Gateway**

A concise overview of the `todo-kong-gateway` configuration, architecture, CI/CD pipeline, and the components present in this folder.

**Overview**
- **Purpose:**: Declarative configuration for a Kong Gateway that fronts a sample TODO service (uses jsonplaceholder).
- **Deployment target:**: Konnect (Kong Cloud) via the `decK` CLI.

**Architecture**
- **Gateway:**: Kong Gateway configured declaratively under `kong-gateway/`.
- **Control plane:**: Konnect control plane where configuration is applied.
- **Sync tool:**: `decK` (deck) is used to validate, diff and apply configuration from the repository to Konnect.

**Components (files in this folder)**
- **Services:**: [todo-kong-gateway/kong-gateway/services.yaml](todo-kong-gateway/kong-gateway/services.yaml) — declares the `todo-service` pointing to `https://jsonplaceholder.typicode.com` and the route `/todos`.
- **Consumers:**: [todo-kong-gateway/kong-gateway/consumers.yaml](todo-kong-gateway/kong-gateway/consumers.yaml) — contains the `upxill-dev` consumer and a key-auth credential.
- **Authentication plugin:**: [todo-kong-gateway/kong-gateway/plugins-auth.yaml](todo-kong-gateway/kong-gateway/plugins-auth.yaml) — configures `key-auth` and key name(s).
- **Caching plugin:**: [todo-kong-gateway/kong-gateway/plugins-cache.yaml](todo-kong-gateway/kong-gateway/plugins-cache.yaml) — configures `proxy-cache` with memory strategy and TTL.
- **CI/CD workflow:**: [todo-kong-gateway/.github/workflows/deploy-gateway.yaml](todo-kong-gateway/.github/workflows/deploy-gateway.yaml) — GitHub Actions pipeline that validates, diffs, and applies the declarative configuration to Konnect using `decK`.

**CI/CD (GitHub Actions)**
- **Trigger:**: `push` to `main` (deploy) and `pull_request` to `main` (dry-run validations).
- **Key steps:**
  - **Checkout code**: `actions/checkout@v4`.
  - **Setup decK**: `kong/setup-deck@v1` (pinned `deck-version: '1.39.3'`).
  - **Validate**: `deck gateway validate kong-gateway/` — syntax/lint validation.
  - **Diff**: `deck gateway diff kong-gateway/` — preview configuration changes.
  - **Apply**: `deck gateway apply kong-gateway/` — run only on pushes to `main` and uses `DECK_KONNECT_TOKEN` secret.

**How to run locally**
- Install `decK` (match pinned version):

  ```bash
  # macOS (example via Homebrew)
  brew install deck
  deck version
  ```

- Validate locally:

  ```bash
  deck gateway validate kong-gateway/
  deck gateway diff kong-gateway/    # preview
  deck gateway apply kong-gateway/   # applies (requires DECK_KONNECT_TOKEN)
  ```

**Secrets / Environment**
- The workflow expects a secret named `DECK_KONNECT_TOKEN` in the repository settings to authenticate to Konnect.
- Optionally set `DECK_KONNECT_CONTROL_PLANE_NAME` if you use a custom control plane name.

**Notes & Recommendations**
- Keep `deck-version` pinned to match your local tooling to avoid unexpected behavior.
- Use PRs to run the diff/validate steps before merging to `main`.
- For production, consider adding automated tests or health checks that validate runtime behavior after sync.

**Files**
- [todo-kong-gateway/.github/workflows/deploy-gateway.yaml](todo-kong-gateway/.github/workflows/deploy-gateway.yaml)
- [todo-kong-gateway/kong-gateway/services.yaml](todo-kong-gateway/kong-gateway/services.yaml)
- [todo-kong-gateway/kong-gateway/consumers.yaml](todo-kong-gateway/kong-gateway/consumers.yaml)
- [todo-kong-gateway/kong-gateway/plugins-auth.yaml](todo-kong-gateway/kong-gateway/plugins-auth.yaml)
- [todo-kong-gateway/kong-gateway/plugins-cache.yaml](todo-kong-gateway/kong-gateway/plugins-cache.yaml)

**Next steps**
- Update consumer credentials and plugin tuning as needed for your environment.
- Ask for help converting this to a self-hosted Gateway deployment or adding automated tests.
