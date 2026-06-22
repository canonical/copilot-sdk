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
3. On launch, the SDK configures `PATH` for the `copilot` binary
   and adds a `copilot-instructions.md` hint about the workshop environment.

### Start a coding session

Once the workshop is ready:

```bash
workshop shell
copilot
```

This opens an interactive Copilot session inside the workshop. You can ask
Copilot to read files, write code, run commands, and navigate your project.

### Authenticate with GitHub Copilot

To make your host Copilot credentials available inside the workshop,
you have two alternatives:

- Set the `GH_TOKEN` or `GITHUB_TOKEN` [environment variable](https://developers.openai.com/api/docs/quickstart/) inside the workshop.
  You can pass it using the `--env` option with `workshop run` or `workshop exec`,
  or by other means such as [direnv](https://direnv.net/).

- If neither variable is set, Copilot will prompt for an API token
  or offer browser-based login on first interactive use.
  The mount plug persists these credentials between workshop updates.

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
