# Haloy Agent Skills

A collection of LLM skills for [Haloy](https://haloy.dev), the cloudless deployment tool.

Skills are packaged instructions that extend AI coding agents (Claude Code, Cursor, Copilot, etc.) with specialized capabilities. This repository follows the [Agent Skills format](https://agentskills.io).

## Available Skills

| Skill | Description |
|-------|-------------|
| [dockerize](./dockerize) | Create production-ready Dockerfiles for haloy deployment |
| [haloy-config](./haloy-config) | Create haloy.yaml configuration files for deploying applications with haloy |

## Installation

Install a skill using the skills CLI:

```bash
npx skills add haloydev/agent-skills
```

For Claude.ai (without CLI), add the skill folder to project knowledge or paste the SKILL.md contents directly into your conversation.

## What is Haloy?

Haloy is a deployment platform that simplifies application deployment without vendor lock-in. Like docker-compose, but for production. Deploy to your own servers using a single YAML configuration file.

Key features:
- Docker-native (runs any containerized application)
- Zero-downtime deployments with rolling updates
- Automatic SSL/TLS certificate management
- Health checks with auto-recovery
- Instant rollbacks

Learn more at [haloy.dev](https://haloy.dev).

## Contributing

See [AGENTS.md](./AGENTS.md) for instructions on creating new skills.

## License

MIT
