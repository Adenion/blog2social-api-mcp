# Blog2Social MCP Server

[![HostedMCP checks](https://api.hostedmcp.org/v1/badge/server/adenion/blog2social-api-mcp.svg)](https://hostedmcp.org/servers/adenion/blog2social-api-mcp/checks)

The **Blog2Social MCP Server** provides a hosted [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) interface for connecting AI assistants, AI agents, coding assistants and automation tools directly to Blog2Social.

With the MCP server, compatible AI applications can use Blog2Social functionality through natural-language instructions instead of requiring a custom Blog2Social API integration for each application.

The MCP server can be used to:

* view connected social media accounts
* discover supported social networks
* connect and disconnect social media accounts
* check Blog2Social credit balances
* review aggregated account usage by day, endpoint and social network
* inspect publishing capabilities and network-specific limits
* publish text, link, image and video posts
* retrieve performance insights for published posts
* check the publishing status of video posts

The MCP server is hosted by Blog2Social. **No local MCP server installation is required.** Authentication is handled through OAuth.

## MCP Endpoint

```text
https://api.blog2social.com/mcp
```

The endpoint is the same for all supported MCP clients.

## How it works

The Blog2Social MCP Server acts as a bridge between an AI application and Blog2Social:

```text
AI Assistant / AI Agent
          |
          | MCP
          v
Blog2Social MCP Server
          |
          | Blog2Social API
          v
     Blog2Social
          |
          +-- Connected Social Accounts
          +-- Social Networks
          +-- Posts
          +-- Publishing
```

This means that an MCP-compatible application can interact with Blog2Social using standardized MCP tools rather than implementing the Blog2Social API itself.

For example, an AI assistant can be instructed to:

```text
Show me my connected social media accounts.
```

or:

```text
Publish this article to my LinkedIn page and Facebook page.
```

The AI application can then use the appropriate Blog2Social MCP tools to perform the requested operation.

## Authentication

Blog2Social MCP uses **OAuth** for authentication and authorization.

The typical authentication flow is:

```text
AI Application
      |
      v
Blog2Social MCP
      |
      v
Blog2Social OAuth
      |
      v
Login & Authorization
      |
      v
MCP Connection
      |
      v
Blog2Social MCP Tools
```

You do **not** need to enter your Blog2Social password into the AI application's MCP configuration.

After adding the MCP endpoint, the client redirects you through the Blog2Social OAuth flow. Log in to Blog2Social and explicitly authorize the connection.

> **Security:** Do not add your Blog2Social password, service token or access token to an MCP client configuration file. Authentication is handled through OAuth.

---

# Available Tools

The production Blog2Social MCP server exposes the following tools. Development-only tools are not listed here.

## `networks.list`

**Read-only**

Returns the social networks available to the authenticated Blog2Social API user, including network IDs and supported account types such as profiles, pages and groups.

Use this tool to find valid `network_id` and account type values before connecting an account.

Example:

```text
Which social networks and account types can I connect to Blog2Social?
```

---

## `connections.list`

**Read-only**

Returns the social media accounts connected to the authenticated Blog2Social user. Each entry includes the network, account type, display name and `client_user_network_id`.

Use `client_user_network_id` to identify the account for publishing or disconnection. This tool lists connected accounts; use `networks.list` for networks that can be connected.

Example:

```text
Show me the social media accounts connected to Blog2Social.
```

---

## `networks.properties`

**Read-only**

Returns publishing requirements and capabilities for the available network and account-type combinations, including supported formats, media capabilities and limits.

The response contains `network_id` and account-type information to identify the relevant entry. Use it before publishing; capabilities vary by network and account type.

Example:

```text
What formats and limits does Blog2Social report for LinkedIn?
```

---

## `connections.add`

**Writes**

Starts OAuth authorization for connecting an account. It requires `network_id` and `network_type_id` from `networks.list`; `language` is optional. Account types are `0` for profile, `1` for page and `2` for group.

The tool returns an `auth_link` that the user must open and approve. The connection is not complete until authorization finishes; use `connections.list` afterward to verify it. Pinterest also requires an active custom app assigned to the API service.

Example:

```text
Connect my LinkedIn page to Blog2Social.
```

---

## `connections.remove`

**Destructive**

Removes the connection identified by `client_user_network_id` from the authenticated Blog2Social user. Use the ID from `connections.list` and only call this tool when the user explicitly asks to disconnect that account.

After removal, the account cannot be used for publishing until it is connected again.

Example:

```text
Disconnect my Facebook page from Blog2Social.
```

---

## `credits.balance`

**Read-only**

Returns the current available credit balance for the authenticated Blog2Social API account.

Example:

```text
How many Blog2Social API credits are available?
```

---

## `account.overview`

**Read-only**

Returns the current credit balance and aggregated usage totals grouped by day, endpoint and social network. The optional `days` parameter is an integer from 1 to 30; when omitted, the endpoint default is used.

This is an account-level usage overview, not performance analytics for an individual post. Use `posts.insights` for post performance.

Example:

```text
Show my Blog2Social API usage for the last 14 days.
```

---

## `posts.publish_post`

**Writes**

Publishes one or more posts immediately to the connected account identified by `client_user_network_id`. This tool does not create drafts or schedule posts.

Each item in `b2s_posts` requires a `message` and `postFormat`:

```text
0 = text or link
1 = image
2 = video
```

Image and video posts also require `mediaObjects` with the matching `IMAGE` or `VIDEO` type and a publicly accessible HTTP or HTTPS `url`. An optional `customUrl` can be included where supported. Check `networks.properties` before publishing.

Only call this tool when the user explicitly requests immediate publication. Check all returned error fields; do not assume success or automatically retry an uncertain request. A successful response may include `publish_url` for `posts.insights`, or `video_token` for `videos.status`.

Example:

```text
Publish this article to my LinkedIn page:

https://example.com/my-article

Use the following text:

We have just published a new article about AI automation.
```

---

## `posts.insights`

**Read-only; may consume credits**

Retrieves the latest available performance insights and associated credit information for one of the user's published posts. The `post_url` argument must be the public `publish_url` returned after a successful `posts.publish_post`.

Insights are available only for supported networks and account types with tracking enabled. Available metrics vary; report only values returned by the tool. The request may deduct credits.

Example:

```text
Show me the performance insights for this published post:
https://example.com/published-post
```

---

## `videos.status`

**Read-only**

Checks video publication status using the `video_token` returned by `posts.publish_post`.

Possible states include `pending`, `failed` and `successful`. A `publish_url` may be returned after successful publication. If the status is pending, check again later; do not treat it as success.

Example:

```text
Check the publication status for this video token: [video_token]
```

---

# Tool Overview

| Tool                  | Type                           | Purpose                                     |
| --------------------- | ------------------------------ | ------------------------------------------- |
| `networks.list`       | Read-only                      | List networks and account types             |
| `connections.list`    | Read-only                      | List connected social accounts              |
| `networks.properties` | Read-only                      | Get network publishing requirements         |
| `connections.add`     | Writes                         | Start account authorization                 |
| `connections.remove`  | Destructive                    | Remove a connected account                  |
| `credits.balance`     | Read-only                      | Check the available credit balance          |
| `account.overview`    | Read-only                      | View aggregated account credit usage        |
| `posts.publish_post`  | Writes                         | Publish content immediately                 |
| `posts.insights`      | Read-only; may consume credits | Retrieve post performance insights          |
| `videos.status`       | Read-only                      | Check video publication status              |

---

# Installation

## No local installation required

The Blog2Social MCP Server is hosted by Blog2Social.

You do **not** need to:

* clone this repository to run an MCP server
* install Node.js
* install Python
* run a local MCP process
* configure a local database
* manually create an OAuth access token

Instead, configure your MCP-compatible client to use the hosted endpoint:

```text
https://api.blog2social.com/mcp
```

Then complete the Blog2Social OAuth authorization flow.

## Generic MCP configuration

For clients that accept a remote MCP server URL, the configuration is conceptually:

```json
{
  "name": "blog2social",
  "url": "https://api.blog2social.com/mcp"
}
```

The exact configuration format depends on the MCP client.

After adding the server, authenticate through OAuth when prompted.

---

# Claude

## Claude.ai

Claude can connect to the hosted Blog2Social MCP server through its custom connector functionality.

1. Open Claude.
2. Open **Settings** or **Connectors**.
3. Select **Add custom connector**.
4. Enter:

```text
https://api.blog2social.com/mcp
```

5. Add the connector.
6. Follow the Blog2Social OAuth flow.
7. Log in to your Blog2Social account.
8. Authorize the connection.

After authorization, Blog2Social is available as an MCP connector in Claude.

> The exact location and naming of connector settings can vary depending on the current Claude interface and account type.

## Claude Desktop

Claude Desktop supports MCP integrations.

Configure the Blog2Social MCP server using:

```text
https://api.blog2social.com/mcp
```

Complete the Blog2Social OAuth authorization when prompted.

After authorization, Claude Desktop can access the Blog2Social MCP tools available to the authenticated account.

---

# Cursor

Cursor supports MCP servers and can connect to the hosted Blog2Social MCP server.

Add the following remote MCP server to your Cursor MCP configuration:

```text
https://api.blog2social.com/mcp
```

For clients that use a JSON configuration, the configuration is typically structured around the remote URL:

```json
{
  "mcpServers": {
    "blog2social": {
      "url": "https://api.blog2social.com/mcp"
    }
  }
}
```

> Cursor's exact MCP configuration format may change between versions. If the installed version provides a dedicated MCP configuration UI, add the URL there instead.

Complete the Blog2Social OAuth flow when prompted.

---

# VS Code

VS Code environments that support MCP can connect to Blog2Social through the hosted MCP endpoint.

Use:

```text
https://api.blog2social.com/mcp
```

For a JSON-based MCP configuration, use a configuration equivalent to:

```json
{
  "servers": {
    "blog2social": {
      "url": "https://api.blog2social.com/mcp"
    }
  }
}
```

Add the endpoint to the MCP configuration and complete the Blog2Social OAuth authorization flow.

> The exact configuration location and schema can depend on the installed VS Code version and the MCP-capable extension or agent being used.

---

# ChatGPT

ChatGPT can connect to MCP servers through supported connector or app integration capabilities.

Use the Blog2Social MCP endpoint:

```text
https://api.blog2social.com/mcp
```

Follow the authentication flow provided by ChatGPT and authorize access to your Blog2Social account.

After authorization, ChatGPT can use the Blog2Social MCP tools available through the connection.

> The exact location and naming of MCP or connector settings may change as ChatGPT updates its MCP and app integration capabilities.

---

# GitHub Copilot

MCP-capable GitHub Copilot environments can connect to Blog2Social through the hosted MCP server.

Use:

```text
https://api.blog2social.com/mcp
```

Complete the Blog2Social OAuth authorization flow when requested.

After authorization, Copilot can use the Blog2Social MCP tools supported by the connection.

---

# n8n

n8n can use MCP-compatible connections to integrate Blog2Social into automated workflows.

Use:

```text
https://api.blog2social.com/mcp
```

Authenticate using the Blog2Social OAuth flow.

A typical workflow can look like:

```text
Trigger
   |
   v
AI / Content Generation
   |
   v
Blog2Social MCP
   |
   v
Create / Publish Content
   |
   v
Social Networks
```

This makes it possible to combine AI-powered content generation and Blog2Social publishing without implementing the Blog2Social REST API separately in every workflow.

---

# Other MCP Clients

The hosted Blog2Social MCP server can also be used with other MCP-compatible applications and development environments.

The current documentation lists support or compatibility information for:

* Claude
* Claude Desktop
* ChatGPT
* Cursor
* VS Code
* GitHub Copilot
* Cline
* Mistral
* n8n
* Notion
* Slackbot
* MCPJam

Support depends on the respective application's MCP and OAuth capabilities.

Blog2Social MCP is also available through MCP directories such as Smithery, Glama, MCPBundles and MCP Servers.

---

# Example Prompts

Once the MCP connection is configured, Blog2Social can be controlled using natural-language instructions.

### List connected accounts

```text
Show me all my connected social media accounts.
```

### Check available networks

```text
Which social networks can I connect to Blog2Social?
```

### Check publishing requirements

```text
What media formats and character limits are supported for LinkedIn?
```

### Check credits

```text
How many Blog2Social API credits are available?
```

### Review account usage

```text
Show my Blog2Social API usage for the last 14 days.
```

### Publish a link

```text
Publish this article to my LinkedIn page:

https://example.com/my-article

Use the following text:

We have just published a new article about AI automation.
```

### Review post performance

```text
Show me the performance insights for this published post:
https://example.com/published-post
```

### Publish an image

```text
Publish this image post to my Facebook page using the attached image
and this message:

Our latest article is now online.
```

### Check a video publication

```text
Check the publishing status of my latest video post.
```

### Manage connections

```text
Show me my connected accounts and tell me which one is my LinkedIn page.
```

---

# MCP vs. REST API

Blog2Social provides several integration options.

| Integration                 | Intended use                                            |
| --------------------------- | ------------------------------------------------------- |
| **MCP**                     | AI assistants, AI agents and natural-language workflows |
| **REST API**                | Custom applications and backend integrations            |
| **AI API Specification**    | AI-assisted API development and code generation         |
| **Automation integrations** | Automated workflows and integrations                    |

MCP is particularly useful when an AI application needs to interact with Blog2Social through standardized tools.

The REST API remains available when direct programmatic access is required.

---

# Supported Social Networks

The underlying Blog2Social API provides a unified interface for publishing across numerous social, blogging, business and community platforms, including:

* Facebook
* X
* LinkedIn
* Instagram
* Pinterest
* Reddit
* Tumblr
* Medium
* Telegram
* TikTok
* YouTube
* Vimeo
* Bluesky
* Threads
* Mastodon
* Discord
* Google Business Profile
* VK
* Xing
* Blogger
* DEV.to
* Flickr
* Diigo
* Bloglovin
* Torial
* Ravelry
* Instapaper
* Band
* HumHub

Supported post types and media formats vary by network. Use `networks.list` to find available networks and account types, then use `networks.properties` to review the reported requirements and capabilities.

---

# Security

Blog2Social MCP uses OAuth for authentication.

The AI application does not receive the Blog2Social password as part of an MCP request. The user authenticates with Blog2Social and explicitly authorizes the connection.

Always review the OAuth authorization request before approving access and only connect trusted MCP clients.

---

# Documentation

* **Blog2Social API Documentation:** https://docs.blog2social.com/api/
* **MCP Documentation:** https://docs.blog2social.com/api/#tag/Integrations/MCP
* **MCP Endpoint:** https://api.blog2social.com/mcp
* **AI API Specification:** https://docs.blog2social.com/api/ai-specification.json

## Repository purpose

This repository contains the metadata used to publish the Blog2Social MCP server to the official MCP Registry.

The MCP server itself is hosted by Blog2Social and does not need to be installed locally.
