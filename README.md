# GitHub Copilot CLI SDK for Workshop

This SDK provides the GitHub Copilot CLI for AI-assisted coding within a
workshop. The agent is sandboxed in the workshop container. Credentials are
persisted between workshop updates.

---

## Reference workshop

A minimal workshop:

```yaml
# workshop.yaml
name: copilot-cli-env
base: ubuntu@24.04
sdks:
  - name: copilot
    channel: latest/stable

actions:
  copilot-yolo: copilot --yolo --interactive="$@"

  copilot-yolo-prompt: copilot --yolo --prompt="$@"
```

This creates a basic Copilot environment.
The agent is sandboxed by the workshop,
so interactive and non-interactive actions can use the YOLO mode.

---

## Using the SDK

### Prerequisites, project layout

1. No prerequisite SDKs are required.
2. Place your project files in your project directory. No special layout is
   required; Copilot CLI works with any codebase.
3. On launch, the SDK puts a `copilot` wrapper on `PATH`
   and adds a `copilot-instructions.md` hint about the workshop environment.
   The SDK pins the Copilot CLI version, so the wrapper sets
   `COPILOT_AUTO_UPDATE=false` unless you set it yourself.

### Start a coding session

Once the workshop is ready:

```bash
workshop shell
copilot
```

This opens an interactive Copilot session inside the workshop. You can ask
Copilot to read files, write code, run commands, and navigate your project.

### Authenticate with GitHub Copilot

Copilot accepts fine-grained personal access tokens (`github_pat_...`) with
the "Copilot Requests" permission, and OAuth tokens from the Copilot CLI or
GitHub CLI (`gh auth token`). Classic personal access tokens (`ghp_...`)
are not supported.

To make your credentials available inside the workshop,
you have these alternatives:

- Set the `COPILOT_GITHUB_TOKEN`, `GH_TOKEN`, or `GITHUB_TOKEN`
  [environment variable](https://docs.github.com/copilot/how-tos/copilot-cli)
  inside the workshop.
  You can pass it using the `--env` option with `workshop run` or `workshop exec`,
  or by other means such as [direnv](https://direnv.net/).

- Connect a secret to the `github-token` plug;
  see [Use a token from the host keyring](#use-a-token-from-the-host-keyring).
  When `COPILOT_GITHUB_TOKEN` isn't set, the `copilot` wrapper reads the secret
  and exports it as `COPILOT_GITHUB_TOKEN` for the Copilot process only,
  so it takes precedence over `GH_TOKEN` and `GITHUB_TOKEN`.
  Note that commands Copilot runs inherit this variable.

- Otherwise, Copilot will prompt for an API token
  or offer browser-based login on first interactive use.
  The mount plug persists these credentials between workshop updates.

#### Use a token from the host keyring

1. Check whether the host keyring already holds a Copilot token.
   When the keyring is available, `copilot login` on the host
   [stores its OAuth token there](https://docs.github.com/copilot/how-tos/copilot-cli/set-up-copilot-cli/authenticate-copilot-cli)
   under the service name `copilot-cli`.
   To list matching items without printing the secret, run:

   ```bash
   secret-tool search --all service copilot-cli | grep -v '^secret = '
   ```

   If an item is listed, skip to the next step
   and use `service: copilot-cli` instead of `service: copilot`
   as the slot attributes,
   adding the item's other attributes if several items are listed.

2. Otherwise, store a token in the host keyring.
   Use a [fine-grained personal access token](https://github.com/settings/personal-access-tokens/new)
   with the "Copilot Requests" permission;
   `secret-tool` prompts for it, so paste the token there:

   ```bash
   secret-tool store --label="copilot" --collection=default service copilot
   ```

   To use the OAuth token of your GitHub CLI login instead, pipe it in:
   `gh auth token | tr -d '\n' | secret-tool store --label="copilot" --collection=default service copilot`.
   To check that it's stored, run `secret-tool lookup service copilot >/dev/null && echo stored`.

3. Expose the keyring item through a `secret` slot on the system SDK
   in your workshop definition:

   ```yaml
   sdks:
     - name: system
       slots:
         copilot-token:
           interface: secret
           collection: default
           attributes:
             service: copilot
     - name: copilot
       channel: latest/stable
   ```

4. Once the workshop is launched, connect the slot to the `github-token` plug:

   ```bash
   workshop connect <workshop-name>/copilot:github-token :copilot-token
   ```

   The connection persists across `workshop refresh`;
   repeat it after `workshop restore` or after removing and launching
   the workshop again.
   To disconnect, use `workshop disconnect` with the same plug.

---

## Plugs (resources this SDK consumes)

### `copilot-config`

- Interface: `mount`
- Workshop target: `/home/workshop/.copilot`
- Purpose: Preserves Copilot's credentials and settings between workshop updates.
  You can also use `workshop remount` to control its contents on the host.
  To mount your existing `~/.copilot` settings into the workshop, stop
  the workshop first, remount, then start it again:

  ```bash
  workshop stop <workshop-name>
  workshop remount <workshop-name>/copilot:copilot-config ~/.copilot
  workshop start <workshop-name>
  ```

### `github-token`

- Interface: `secret`
- Purpose: Provides a GitHub token for Copilot from the host's secret service.
  The `copilot` wrapper reads it with `workshopctl get-secret copilot.github-token`
  and exports it as `COPILOT_GITHUB_TOKEN`,
  unless that variable is already set in the workshop.

## Slots (resources this SDK provides)

This SDK doesn't define any slots.

---

## Documentation and guidance

- [GitHub Copilot CLI documentation](https://docs.github.com/copilot/how-tos/copilot-cli)
- [Workshop documentation](https://ubuntu.com/workshop/docs/)

---

## Community and support

- GitHub Community:
  [GitHub Community Discussions](https://github.com/orgs/community/discussions)
- Workshop forum:
  [Discourse](https://discourse.ubuntu.com/)
- Please review our
  [Code of Conduct](https://ubuntu.com/community/ethos/code-of-conduct) before
  participating.

---

## Contributions

All contributions, including code, documentation updates, and issue reports,
are welcome!

- See `CONTRIBUTING.md` for guidelines.
- Open issues or pull requests on the official repository.

---

## License and copyright

Copyright 2026 Canonical Ltd.

This program is free software: you can redistribute it and/or modify it under
the terms of the
[GNU Lesser General Public License version 2.1 (LGPLv2.1)](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html)
as published by the Free Software Foundation.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the GNU Lesser General Public License for more details.

[GitHub Copilot CLI](https://github.com/features/copilot/cli) is subject to
[GitHub Copilot CLI License](https://github.com/github/copilot-cli/blob/main/LICENSE.md).
