# Coder VM

This is the preferred "local cloud" setup for this project.

Students use a browser on their own laptop or desktop. Coder runs on an Ubuntu Server VM named `coder`. Each student gets a Coder workspace with a browser IDE, terminal, project files, Phoenix server, and AI coding tools.

## Target Shape

```text
Student's laptop browser
  -> Coder on the Ubuntu VM
    -> student workspace container
      -> browser IDE opened at /home/coder/project
      -> terminal
      -> git repo
      -> Elixir, Phoenix, Node, SQLite
      -> Codex CLI or approved AI coding tool
      -> Phoenix app preview
```

The goal is for the student's laptop to need only a browser.

## Why Coder

Coder gives you:

- A browser-based development environment.
- One workspace per student.
- A terminal that runs in the workspace container, not on the student's laptop.
- Port access for previewing the Phoenix app.
- A clean boundary between project files and VM or host administration.

It does not remove all risk. Coder still uses Docker to create workspaces, so the VM should be treated as shared development infrastructure.

## First Pilot

Start with one student workspace before scaling this to everyone.

Pilot checklist:

- Install Coder on the Ubuntu VM.
- Create one admin account.
- Create one student user.
- Create one Docker-based workspace template.
- Build a workspace image with Elixir, Erlang, Node, SQLite, git, Codex CLI, and a browser IDE.
- Configure the template to clone the student's project repo into `~/project`.
- Configure code-server to open `~/project` directly.
- Confirm `~/project/AGENTS.md` exists and `cd ~/project && git status` works.
- Confirm `echo "$PROXY_BASE_PATH"` prints the Coder proxy path.
- Ask the AI to create the first Phoenix LiveView app inside `~/project`, not directly under `/home/coder`.
- Confirm Phoenix preview works through Coder.
- Confirm the AI can run checks and commit working checkpoints.

## VM Rules

Keep Coder data on persistent VM storage, not temporary directories.

Suggested VM-side locations:

```text
/home/larry/coder
/home/larry/coder-images
```

Keep student project work inside Coder workspaces. Students should not need SSH or sudo access to the VM for normal app development.

Avoid exposing Coder directly to the public internet for this beginner setup. Keep it on the local network, behind Tailscale, or behind another access method you already trust.

## Installing Coder

Use Coder's official Docker installation path on the Ubuntu VM.

Coder's Docker docs say the Docker install requires Docker, a Linux machine, and available CPU and memory. The official Docker Compose example includes Coder and PostgreSQL.

High-level install flow:

1. Create a persistent Coder folder on the VM.
2. Download or create a Coder Docker Compose file there.
3. Set Coder's access URL to the URL students will use.
4. Start the Coder stack.
5. Open the Coder URL in a browser.
6. Create the first admin account.
7. Create users and a workspace template.

Reference: <https://coder.com/docs/install/docker>

## Workspace Image

For this project, the workspace image should include:

- Git
- Elixir
- Erlang/OTP
- Phoenix installer
- Node.js
- SQLite
- Build tools
- Codex CLI or another approved AI coding tool, if available and safe to authenticate

Keep project dependencies inside the workspace container.

Do not install Phoenix, Node packages, app databases, or AI tool credentials directly on the VM unless you are intentionally updating the shared workspace image.

## Workspace Template

Use a Docker-based Coder workspace template.

### Project Repository Clone

The workspace home directory, `/home/coder`, is not the project. Treat it like a persistent home folder for tool settings and caches. The project should live in a git repo at:

```text
/home/coder/project
```

For a real student, use that student's GitHub repo created from `ljm42/ai-web-app-starter`. For an admin smoke test, the default starter repo is acceptable, but remove its `origin` so test work is not pushed back to the starter repo.

Add a repo URL parameter:

```hcl
data "coder_parameter" "project_repo_url" {
  name         = "project_repo_url"
  display_name = "Project Git Repository"
  description  = "Student repo URL. Use a repo created from ljm42/ai-web-app-starter when possible."
  type         = "string"
  default      = "https://github.com/ljm42/ai-web-app-starter.git"
  mutable      = true
  icon         = "/icon/git.svg"
  order        = 1
}
```

Add these locals near the existing `locals` block:

```hcl
locals {
  username         = data.coder_workspace_owner.me.name
  project_dir      = "/home/coder/project"
  starter_repo_url = "https://github.com/ljm42/ai-web-app-starter.git"
  proxy_base_path  = "/@${data.coder_workspace_owner.me.name}/${data.coder_workspace.me.name}.main/apps/code-server/proxy/4000"
}
```

Then add the first-start clone to the `coder_agent.main` `startup_script`, after the `/etc/skel` copy:

```sh
if [ ! -d "${local.project_dir}/.git" ]; then
  rm -rf "${local.project_dir}"
  git clone "${data.coder_parameter.project_repo_url.value}" "${local.project_dir}"

  if [ "${data.coder_parameter.project_repo_url.value}" = "${local.starter_repo_url}" ]; then
    git -C "${local.project_dir}" remote remove origin || true
  fi
fi
```

After creating or rebuilding a workspace, this should be true:

```sh
cd ~/project
ls AGENTS.md
git status
echo "$PROXY_BASE_PATH"
```

Create Phoenix apps inside `~/project`. Do not create app folders directly under `/home/coder`, because Codex will not see the starter `AGENTS.md` there and the app may not be inside the project git repo.

### Browser IDE Folder

Configure code-server to open `~/project` directly:

```hcl
module "code-server" {
  count  = data.coder_workspace.me.start_count
  source = "registry.coder.com/coder/code-server/coder"

  version = "~> 1.0"

  agent_id = coder_agent.main.id
  order    = 1
  folder   = local.project_dir
}
```

This lets VS Code see `AGENTS.md` and `.vscode/extensions.json` immediately.

### Proxy Base Path

Apps running behind the Coder path proxy need to generate asset, websocket, route, and API URLs with the workspace proxy prefix. Define this once in the Coder workspace container environment as `PROXY_BASE_PATH`.

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
    "PROXY_BASE_PATH=${local.proxy_base_path}",
  ]

  # ...
}
```

Do not move `CODER_AGENT_TOKEN` into the `coder_agent` `env` map. The Docker container needs that token before the Coder agent can start.

After publishing a new template version, restart or rebuild existing workspaces so new environment variables and startup behavior are present.

## Phoenix App Setup

Student commands can stay simple:

```sh
mix phx.server
```

If a one-off manual run is needed before the template is updated, prefix the command explicitly:

```sh
PROXY_BASE_PATH=/@USERNAME/WORKSPACE.main/apps/code-server/proxy/4000 mix phx.server
```

Ask the AI assistant to configure Phoenix to read `PROXY_BASE_PATH` in `config/runtime.exs`, apply it to both `url` and `static_url`, and use it when constructing LiveView socket paths instead of hardcoding `/live`. Other stacks should use the same variable for their asset, route, API, and websocket base paths.

When the AI creates a new Phoenix app with `mix phx.new`, it should make this proxy adaptation immediately, before asking the student to open the preview URL. The first commit for a new app should include both the Phoenix scaffold and the proxy-path fix.

## Codex In Coder

Use one of these:

- The official Codex VS Code extension, recommended by this repo in `.vscode/extensions.json`.
- Codex CLI inside the Coder workspace terminal, if the student can authenticate safely.
- Codex web/cloud against the student's GitHub repo, if that fits the student's account.
- Another approved AI coding tool available in the browser IDE.

Do not install random third-party "Codex web UI" containers that ask for OpenAI credentials.

### Codex Extension Sign-In

When the Codex IDE extension signs in from a browser-based Coder workspace, the login flow may redirect to a local callback URL like:

```text
http://localhost:1455/...
```

That `localhost` is inside the Coder workspace, not on the student's laptop. If the browser cannot open the callback URL, replace only this part:

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

1. Create their own GitHub repo from `ljm42/ai-web-app-starter`, if they have a GitHub account.
2. Create a Coder workspace using that repo URL, or use the default starter URL for a local-only smoke test.
3. Open the Coder URL in a browser on their laptop.
4. Sign in and open their workspace.
5. Open the browser IDE, which should open `~/project` directly.
6. Confirm `AGENTS.md` is visible and `git status` works inside `~/project`.
7. Work with the AI assistant in small steps.
8. Commit working checkpoints.
9. Push to their GitHub repo when ready.

Students should not need SSH or sudo access to the VM for normal app work.

## Good First Student Prompt

```text
We are working in a Coder workspace running on a shared Coder VM.

Before building anything, verify that we are working inside `~/project`, that `AGENTS.md` exists, that `git status` works, and that `PROXY_BASE_PATH` is set. If not, stop and help me fix the workspace setup.

I want to build a small Phoenix LiveView web app with AI assistance. Please create the app inside `~/project`, not directly under `/home/coder`.

Please keep all development tools, dependencies, and app data inside this workspace. Do not install anything on the Coder VM.

Help me choose a small first app idea, then build the first working feature.
```

## Open Questions For The Pilot

Before scaling beyond one workspace, decide:

- Will each student have a GitHub account?
- Will each student authenticate their own AI coding tool account?
- How much CPU and memory can each workspace use?
- Should workspaces shut down automatically when idle?
- Should app preview ports be private to each student or visible to all authenticated Coder users?
- Who owns backups of the Coder VM and workspace data?
