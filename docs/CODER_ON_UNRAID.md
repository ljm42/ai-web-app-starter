# Coder On Unraid

This is the preferred "local cloud" setup for this project.

Students use a browser on Windows. Coder runs on Unraid. Each student gets a Coder workspace with a browser IDE, terminal, project files, Phoenix server, and AI coding tools.

## Target Shape

```text
Windows browser
  -> Coder on Unraid
    -> student workspace container
      -> browser IDE
      -> terminal
      -> git repo
      -> Elixir, Phoenix, Node, SQLite
      -> Codex CLI or other approved AI coding tool
      -> Phoenix app preview
```

The goal is for the Windows PC to need only a browser.

## Why Coder

Coder gives you:

- A browser-based development environment.
- One workspace per student.
- A terminal that runs on Unraid instead of Windows.
- Port access for previewing the Phoenix app.
- A cleaner boundary between student workspaces and the Unraid host.

It does not remove all risk. Coder still needs Docker access to create workspaces, and Docker access on Unraid is powerful.

## First Pilot

Start with one student workspace before scaling this to everyone.

Pilot checklist:

- Install Coder on Unraid.
- Create one admin account.
- Create one student user.
- Create one Docker-based workspace template.
- Build a workspace image with Elixir, Erlang, Node, SQLite, git, and a browser IDE.
- Clone the student's GitHub repo inside the workspace.
- Run `mix phx.new` or ask the AI to create the first Phoenix LiveView app.
- Start Phoenix with `mix phx.server`; the workspace template should provide `CODER_PROXY_BASE_PATH` automatically.
- Open the app preview through Coder.
- Confirm the student can commit and push changes.

## Unraid Host Rules

Keep Coder data on persistent storage, not the Unraid root filesystem.

Suggested locations:

```text
/mnt/user/appdata/coder
/mnt/user/ai-students
```

Do not store Coder data or student projects in:

```text
/root
/tmp
/
```

Avoid exposing Coder directly to the public internet for this beginner setup. Keep it on the local network or behind an access method you already trust.

## Installing Coder

Use Coder's official Docker installation path.

Coder's Docker docs say the Docker install requires Docker, a Linux machine, and available CPU and memory. The official Docker Compose example includes Coder and PostgreSQL.

High-level install flow:

1. Create a persistent Coder folder on Unraid.
2. Download or create a Coder Docker Compose file there.
3. Set Coder's access URL to the local URL students will use.
4. Start the Coder stack.
5. Open the Coder URL in a browser.
6. Create the first admin account.
7. Create users and a workspace template.

Reference: <https://coder.com/docs/install/docker>

## Workspace Template

Use a Docker-based workspace template.

For this project, the workspace image should include:

- Git
- Elixir
- Erlang/OTP
- Phoenix installer
- Node.js
- SQLite
- Build tools
- A browser IDE such as code-server or OpenVSCode Server
- Codex CLI or another approved AI coding tool, if available and safe to authenticate

Keep project dependencies inside the workspace container.

Do not install Phoenix, Node packages, app databases, or AI tool credentials directly on the Unraid host.

### Coder Proxy Base Path

Apps running behind the Coder path proxy need to generate asset, websocket, route, and API URLs with the workspace proxy prefix. Define this once in the Coder workspace container environment as `CODER_PROXY_BASE_PATH`.

For the current Docker template shape, add a local value near the existing `locals` block:

```hcl
locals {
  username              = data.coder_workspace_owner.me.name
  coder_proxy_base_path = "/@${data.coder_workspace_owner.me.name}/${data.coder_workspace.me.name}.main/apps/code-server/proxy/4000"
}
```

Keep Git identity in the Coder agent environment:

```hcl
resource "coder_agent" "main" {
  # ...

  env = {
    GIT_AUTHOR_NAME     = coalesce(data.coder_workspace_owner.me.full_name, data.coder_workspace_owner.me.name)
    GIT_AUTHOR_EMAIL    = data.coder_workspace_owner.me.email
    GIT_COMMITTER_NAME  = coalesce(data.coder_workspace_owner.me.full_name, data.coder_workspace_owner.me.name)
    GIT_COMMITTER_EMAIL = data.coder_workspace_owner.me.email
  }

  # ...
}
```

Pass the Coder agent token and proxy base path into the Docker workspace container:

```hcl
resource "docker_container" "workspace" {
  # ...

  env = [
    "CODER_AGENT_TOKEN=${coder_agent.main.token}",
    "CODER_PROXY_BASE_PATH=${local.coder_proxy_base_path}",
  ]

  # ...
}
```

Do not move `CODER_AGENT_TOKEN` into the `coder_agent` `env` map. The Docker container needs that token before the Coder agent can start.

After publishing the new template version, restart or rebuild existing workspaces so the new environment variable is present. Student commands can stay simple:

```sh
mix phx.server
```

If a one-off manual run is needed before the template is updated, prefix the command explicitly:

```sh
CODER_PROXY_BASE_PATH=/@USERNAME/WORKSPACE.main/apps/code-server/proxy/4000 mix phx.server
```

Ask the AI assistant to configure Phoenix to read `CODER_PROXY_BASE_PATH` in `config/runtime.exs`, apply it to both `url` and `static_url`, and use it when constructing LiveView socket paths instead of hardcoding `/live`. Other stacks should use the same variable for their asset, route, API, and websocket base paths.

## Codex In Coder

There does not appear to be an official self-hosted "Codex web UI Docker container" to install next to Coder.

Use one of these instead:

- Codex CLI inside the Coder workspace terminal, if the student can authenticate safely.
- Codex web/cloud against the student's GitHub repo, if that fits the student's account.
- Another approved AI coding tool available in the browser IDE.

Do not install random third-party "Codex web UI" containers that ask for OpenAI credentials.

### Codex Extension Sign-In

When the Codex IDE extension signs in from a browser-based Coder workspace, the login flow may redirect to a local callback URL like:

```text
http://localhost:1455/...
```

That `localhost` is inside the Coder workspace, not on the student's computer. If the browser cannot open the callback URL, replace only this part:

```text
http://localhost:1455
```

with the Coder proxy URL for that workspace:

```text
https://coder.example.ts.net/@USERNAME/WORKSPACE.main/apps/code-server/proxy/1455
```

Keep the rest of the callback path and query string exactly the same.

## App Preview

Phoenix commonly runs on:

```text
http://localhost:4000
```

Coder supports workspace port access and dashboard app links. For this project, prefer authenticated access only.

Do not make app preview ports public until the app has had a security review.

Reference: <https://coder.com/docs/user-guides/workspace-access/port-forwarding>

## Student Flow

Students should normally do this:

1. Open the Coder URL in a browser.
2. Sign in.
3. Open their workspace.
4. Open the browser IDE.
5. Open a terminal.
6. Work with the AI assistant in small steps.
7. Commit working checkpoints.
8. Push to their GitHub repo when ready.

They should not need an Unraid root shell for normal app work.

## Good First Student Prompt

```text
We are working in a Coder workspace running on an Unraid server.

I want to build a small Phoenix LiveView web app with AI assistance.

Please keep all development tools, dependencies, and app data inside this workspace. Do not install anything on the Unraid host.

Help me choose a small first app idea, then build the first working feature.
```

## Open Questions For The Pilot

Before scaling beyond one workspace, decide:

- Will each student have a GitHub account?
- Will each student authenticate their own AI coding tool account?
- How much CPU and memory can each workspace use?
- Should workspaces shut down automatically when idle?
- Should app preview ports be private to each student or visible to all authenticated Coder users?
- Who owns backups of `/mnt/user/appdata/coder` and `/mnt/user/ai-students`?

## Future Safer Alternative

If the Unraid server later has enough RAM, running Coder inside an Ubuntu Server VM is cleaner than running it directly on Unraid.

For now, direct-on-Unraid is acceptable for a small, supervised setup, as long as student work stays inside Coder workspaces and host-level changes are treated carefully.
