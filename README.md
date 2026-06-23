# AI Web App Starter

This is a starter repo for building a small web app with help from AI.

You do not need to understand every line of code at the beginning. Your job is to describe what you want the app to do, ask the AI to build small pieces, review the results, and keep the project moving safely.

## Before You Start

Create your own repo from this template, then open it in your development environment.

If you are using the shared Coder VM, read [Coder VM](docs/CODER_VM.md). In Coder, the project should be cloned into `~/project`; make sure `~/project/AGENTS.md` exists before asking the AI to build the app.

If you are working on your own computer, clone your repo locally, open the project folder in your editor, and start a chat with your AI coding assistant.

## First AI Prompt

Copy this into your AI assistant:

```text
This is my own repo based on an AI web app starter template.

I want to build a small web app, but I have not chosen the idea yet.

Help me choose a beginner-friendly app idea. Ask me a few questions, then suggest three possible apps. For each idea, include the first small feature we should build.
```

If you already know your app idea, use this instead:

```text
This is my own repo based on an AI web app starter template.

I want to build: [describe your app idea here].

Help me turn this into a small first version. Ask any important questions, then propose the first feature to build.
```

## How To Work With The AI

Ask for one small feature at a time.

Good requests:

- Build the simplest version of this feature.
- Explain any risks before you add login, file uploads, payments, email, or public sharing.
- Review the code you just wrote for bugs and security issues.
- Run the tests and commit the working checkpoint.
- Explain only what I need to know to safely ask for the next change.

Avoid asking for a whole complicated app at once. Small steps are easier to review, test, and fix.

## Safety Checkpoints

Before adding any of these, ask for a risk review first:

- Login or passwords
- Admin pages
- Private user data
- File uploads
- Payments
- Email or text messages
- Public deployment
- External APIs or API keys
- Deleting user data

Use this prompt:

```text
Before implementing this, explain the security, privacy, and reliability risks. Then suggest the safest beginner-friendly version.
```

## Git Checkpoints

The AI should help you use git as a safety net.

After each working feature, ask:

```text
Please review this feature, run the relevant checks, and commit it with a clear message if everything looks good.
```

This makes it easier to recover if a later change breaks something.

## Replace This README Later

This README is only for starting the project.

Once your app has a name and a real first feature, ask the AI to replace this file with a README for your actual app:

```text
Please replace the starter README with a README for my app. Include what the app does, how to run it, and any important setup steps.
```
