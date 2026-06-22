# Unraid Dev Environment

This setup uses an Unraid server as the Linux development machine.

The Windows PC runs VS Code, but Elixir, Phoenix, git commands, tests, and app servers run on Unraid inside a VS Code dev container.

## Why This Setup

This avoids installing development tools directly on the Windows PC.

The Windows PC needs:

- VS Code
- The VS Code Remote - SSH extension
- The VS Code Dev Containers extension
- An SSH key that can connect to the Unraid server

The Unraid server needs:

- SSH enabled
- Docker enabled
- Git available on the host
- A persistent folder for student projects

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

Because students may need to connect as `root`, be extra careful:

- Work only inside the project folder.
- Do not edit Unraid system files.
- Do not change Docker containers unrelated to the project.
- Do not run destructive commands unless you fully understand them.
- Do not expose the app publicly until a security review has been done.

Docker access is powerful. Treat the Unraid host like production infrastructure, even for a learning project.

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

## VS Code Flow

On the Windows PC:

1. Open VS Code.
2. Install the Remote - SSH extension.
3. Install the Dev Containers extension.
4. Connect to the Unraid server with Remote - SSH.
5. Open the student's project folder on Unraid.
6. Ask the AI assistant to add or use a `.devcontainer` setup for the project.
7. Run `Dev Containers: Reopen in Container`.

After that, commands such as `mix test`, `mix phx.server`, and `git status` should run inside the Linux dev container.

## First Prompt For The AI

Use this after opening the project on Unraid:

```text
We are developing this app on an Unraid server through VS Code Remote SSH.

Please use a VS Code dev container for development tools. Do not install Elixir, Phoenix, Node, databases, or app dependencies directly on the Unraid host unless I explicitly approve it.

The project should stay inside this repo folder. Before running commands that affect Docker, the host filesystem, networking, or public access, explain what you are about to do and why.
```

## Ports And App Access

During development, the app should only be exposed on the local network or through VS Code port forwarding.

Do not open router ports or expose the app publicly until the AI has performed a security review.

For Phoenix, the development server commonly uses:

```text
http://localhost:4000
```

When running through VS Code Remote SSH or a dev container, VS Code can forward that port back to the Windows PC.

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
