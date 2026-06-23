# Unraid Dev Environment

This setup uses an Unraid server as the Linux development machine.

The preferred setup is to run Coder on Unraid. Students use a browser on their Windows PC, while the editor, terminal, Elixir, Phoenix, git commands, tests, and app servers run in Coder workspaces on Unraid.

## Why This Setup

This avoids installing development tools directly on the Windows PC.

The Windows PC needs:

- A modern browser
- A GitHub account, if the student wants their own GitHub repo

The Windows PC should not need Docker, WSL, Elixir, Phoenix, Node, VS Code, or SSH for normal student work.

The Unraid server needs:

- SSH enabled
- Docker enabled
- Git available on the host
- A persistent folder for student projects
- Coder installed and reachable from the local network

For the admin setup path, see [Coder On Unraid](CODER_ON_UNRAID.md).

## Important Unraid Warning

On Unraid, the root filesystem is not where project work should live.

Do not store projects in:

```text
/root
/tmp
/
```

Use a persistent share instead, such as:

```text
/mnt/user/ai-students
```

or a cache-backed share if one is available.

Because the initial Coder setup may require a root shell on Unraid, be extra careful:

- Work only inside the project folder.
- Do not edit Unraid system files.
- Do not change Docker containers unrelated to the project.
- Do not run destructive commands unless you fully understand them.
- Do not expose the app publicly until a security review has been done.

Docker access is powerful. Treat the Unraid host like production infrastructure, even for a learning project.

Students should normally work in browser-based Coder workspaces, not directly in the Unraid root shell.

## Suggested Folder Layout

Create one folder per student:

```text
/mnt/user/ai-students/student-a
/mnt/user/ai-students/student-b
/mnt/user/ai-students/student-c
```

Each student should clone their own GitHub repo into their folder:

```sh
cd /mnt/user/ai-students/student-a
git clone git@github.com:STUDENT_USERNAME/STUDENT_REPO.git
cd STUDENT_REPO
```

## Coder Browser Flow

On the Windows PC:

1. Open the Coder URL in a browser.
2. Sign in.
3. Create or open the student's workspace.
4. Open the browser IDE from the workspace page.
5. Open a terminal in the browser IDE.
6. Clone the student's repo, or create a new repo from the starter template first.
7. Ask the AI assistant to help build the first small feature.

Commands such as `mix test`, `mix phx.server`, and `git status` should run inside the Coder workspace container.

## First Prompt For The AI

Use this after opening the project in Coder:

```text
We are developing this app in a Coder workspace running on an Unraid server.

Please keep development tools and app dependencies inside this Coder workspace. Do not install Elixir, Phoenix, Node, databases, or app dependencies directly on the Unraid host unless I explicitly approve it.

The project should stay inside this repo folder. Before running commands that affect Docker, the host filesystem, networking, or public access, explain what you are about to do and why.
```

## Ports And App Access

During development, the app should only be exposed through Coder's workspace app or port access features, or on the local network if an adult has reviewed it.

Do not open router ports or expose the app publicly until the AI has performed a security review.

For Phoenix, the development server commonly uses:

```text
http://localhost:4000
```

When running through Coder, the workspace page can expose or link to the running app depending on how the workspace template is configured. The workspace template should set `PROXY_BASE_PATH` so apps can generate correct asset URLs, static URLs, API URLs, and websocket paths behind the Coder proxy.

Apps should read `PROXY_BASE_PATH` from the environment and fall back to `/` when it is not set. With the template configured, students should normally start Phoenix with:

```sh
mix phx.server
```

## Remote SSH Fallback

If Coder is not working yet, VS Code Remote SSH can still be used as a fallback.

In that mode, the Windows PC needs:

- VS Code
- The VS Code Remote - SSH extension
- The VS Code Dev Containers extension
- An SSH key that can connect to the Unraid server

This is less ideal for students because it reintroduces local setup and SSH troubleshooting.

## Alternative: Ubuntu Server VM

If the Unraid server has enough RAM in the future, a safer alternative is to run an Ubuntu Server VM on Unraid and use that as the development host.

That setup looks like:

```text
Windows PC
  -> VS Code Remote SSH
    -> Ubuntu Server VM on Unraid
      -> Docker
        -> VS Code dev container
```

The VM approach is cleaner because students do not need root access to the main Unraid host, and mistakes are easier to isolate or roll back. The direct Unraid setup is acceptable for this project, but it deserves extra caution.
