# About local sandboxing for GitHub Copilot

Local sandboxes provide isolated execution environments that let Copilot safely interact with code, tools, filesystem, and network resources securely on your local machine.

## Introduction

Copilot local sandboxes are the execution platform for secure sandboxed experiences in GitHub Copilot CLI and the GitHub Copilot app. Sandboxing provides isolation, portability, and policy controls as Copilot runs tools, executes commands, and modifies files.

Sandboxing is available in both surfaces, but settings are configured separately:

- In Copilot CLI, control local sandboxing with `/sandbox` commands and CLI settings.
- In the GitHub Copilot app, project settings define defaults and active sessions can override them.

Choose a sandbox type:

- **Local sandboxing**: Runs Copilot on your machine with restricted filesystem, network, and system access.
- **Cloud sandboxing**: Runs the entire session remotely in a fully isolated Linux environment.

## Local sandboxing

Local sandboxing restricts what Copilot commands can read, write, and reach. It is turned off by default. Before enabling it, shell commands run with the access of the current user.

Local sandboxing is powered by Microsoft eXecution Container (MXC). Copilot CLI declares a policy—readable and writable paths, network access, and other controls—and MXC applies it through the appropriate operating-system isolation backend.

Local sandboxing is lighter-weight isolation: it limits process access but does not run commands inside a separate virtual machine or container.

## Enabling local sandboxing

In a Copilot CLI session, run:

```shell
/sandbox enable
```

Copilot continues using local sandboxing in future sessions until `/sandbox disable` is run.

The GitHub Copilot app uses repository settings for the default of new local sessions. The default can be changed for an active session without changing the repository default.

## Configuring local sandboxing

Copilot CLI can control:

- **Filesystem**: Read-only or read/write access to paths, and denied paths.
- **Network**: Outbound and local network access, allowed hosts, and denied hosts.
- **Credentials**: Git and GitHub CLI authentication and masking of additional environment variables.
- **Subprocesses**: Whether local MCP and language servers run inside the sandbox.
- **Keychain**: Whether macOS keychain access is available.
- **Per-command exceptions**: Whether individual commands may run outside the sandbox.

The GitHub Copilot app repository settings expose filesystem, network, and credential controls. The app may also ask for approval before running an individual command outside the sandbox.

## Cross-platform support

- **macOS** uses Seatbelt and requires macOS 15 (Sequoia) or later.
- **Linux** uses bubblewrap 0.5.0 or later. `bwrap` must be on `PATH`.
- **Windows** requires a recent Windows 11 update that supports sandboxing.

### Linux requirements

For outbound traffic, Linux sandboxing may require:

- `slirp4netns`
- `unshare` and `nsenter` from util-linux 2.35 or later
- `iptables`, `ip6tables`, and their restore binaries
- Access to `/dev/net/tun`

### Proxy support

- On macOS and Linux, outbound connections are restricted to the built-in local proxy. Programs that ignore proxy settings cannot connect directly.
- On Windows, proxying and host rules require a supported Windows version and **Allow local network**. Windows programs must honor proxy settings.

In the app, denied paths fail closed if the active container capabilities cannot enforce them. On Linux, bubblewrap cannot independently control local network access for spawned shell, MCP, or language-server processes.

## Unsupported hosts

A saved Copilot CLI sandbox preference is turned off for the current session if the host does not support it. The session continues without an enforced sandbox and reports that shell commands and services will run unsandboxed.

Enterprise or device policy can require sandboxing. When `sandbox.failIfUnavailable` and `sandbox.enabled` are both enabled, Copilot can block model requests and tool execution if sandboxing cannot be enforced.

On unsupported hosts, the GitHub Copilot app may start a session configured for local sandboxing but fail when sandboxed processes are launched. Device policy may require the same fail-closed behavior.

## Authentication and billing

Sandboxing uses existing Copilot authentication. Separate cloud-provider credentials or infrastructure are not required.

Local sandboxing is included in the standard GitHub Copilot seat at no additional cost.
