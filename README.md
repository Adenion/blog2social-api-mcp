# Blog2Social MCP Server

The **Blog2Social MCP Server** provides a hosted [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) interface for connecting AI assistants, AI agents, coding assistants and automation tools directly to Blog2Social.

With the MCP server, compatible AI applications can use Blog2Social functionality through natural-language instructions instead of requiring a custom Blog2Social API integration for each application.

The MCP server can be used to:

* view connected social media accounts
* discover supported social networks
* connect and disconnect social media accounts
* check Blog2Social credit balances
* inspect publishing capabilities and network-specific limits
* publish text, link, image and video posts
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

The Blog2Social MCP server currently provides the following tools.

## `connections.list`

**Read-only**

Returns the social media accounts currently connected to the authenticated Blog2Social account.

The response includes information such as:

* social network
* account type
* display name
* `client_user_network_id`

The `client_user_network_id` identifies the connected account and is used when publishing posts with `posts.create`.

Example use:

```text
Show me all social media accounts connected to Blog2Social.
```

---

## `connections.add`

**Writes**

Starts the authorization process for connecting a social network account.

The tool uses:

* `network_id` – identifies the social network
* `network_type_id` – identifies the account type

Supported account types are:

```text
0 = Profile
1 = Page
2 = Group
```

The tool normally returns an authorization link that the user opens to authorize the connection.

Pinterest is an exception and may require a custom application configuration instead of the standard authorization link.

Example use:

```text
Connect my LinkedIn account to Blog2Social.
```

The available networks and supported account types should be checked with `networks.list` before starting the connection process.

---

## `connections.remove`

**Destructive**

Removes an existing social network connection from the authenticated Blog2Social account.

The `client_user_network_id` from `connections.list` is required to identify the account to remove.

After removal, the account can no longer be used for publishing until it is connected again.

Example use:

```text
Disconnect my Facebook page from Blog2Social.
```

Because this operation is destructive, it should only be executed when the user explicitly requests the removal.

---

## `networks.list`

**Read-only**

Returns the social networks that can be connected to Blog2Social.

For each network, the response can include:

* network name
* network URL
* `network_id`
* supported profiles
* supported pages
* supported groups

The returned `network_id` can be used with `connections.add`.

Example use:

```text
Which social networks can I connect to Blog2Social?
```

---

## `networks.list_properties`

**Read-only**

Returns publishing capabilities and restrictions for a specific social network and account type.

This can include:

* supported media formats
* image support
* video support
* character limits
* other network-specific publishing requirements

This tool should be used when preparing content for a specific network to ensure that the content matches the network's requirements.

Example use:

```text
What are the publishing requirements for LinkedIn?
```

---

## `credits.balance`

**Read-only**

Returns the current credit balance of the authenticated Blog2Social API account.

The credit balance can be checked when a publishing operation cannot be completed or before performing an operation that requires credits.

Example use:

```text
How many Blog2Social credits do I have left?
```

---

## `posts.create`

**Writes**

Publishes content to a connected social media account.

The target account is identified using its `client_user_network_id`.

Supported post formats are:

```text
0 = Text / Link
1 = Image
2 = Video
```

Depending on the selected format, posts can contain text, links, images or videos.

For non-video posts, the response can contain a `publish_url`.

Example:

```text
Publish this article to my LinkedIn page:

https://example.com/my-article

Use the following text:

We have just published a new article about AI automation.
```

The AI client can use `connections.list` to identify the appropriate connected account and then use `posts.create` to publish the content.

---

## `videos.check`

**Read-only**

Checks the publishing status of a video post.

When `posts.create` publishes a video, the response can contain a `video_token`.

`videos.check` uses this token to check the current publishing status.

Possible states include:

```text
pending
failed
successful
```

When publishing succeeds, a `publish_url` can be returned.

Example use:

```text
Check whether my video post has finished publishing.
```

---

# Tool Overview

| Tool                       | Type        | Purpose                               |
| -------------------------- | ----------- | ------------------------------------- |
| `connections.list`         | Read-only   | List connected social accounts        |
| `connections.add`          | Writes      | Start connecting a social account     |
| `connections.remove`       | Destructive | Remove a connected social account     |
| `networks.list`            | Read-only   | List available social networks        |
| `networks.list_properties` | Read-only   | Get network publishing requirements   |
| `credits.balance`          | Read-only   | Check the current credit balance      |
| `posts.create`             | Writes      | Publish text, links, images or videos |
| `videos.check`             | Read-only   | Check video publishing status         |

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
How many Blog2Social credits do I have?
```

### Publish a link

```text
Publish this article to my LinkedIn page:

https://example.com/my-article

Use the following text:

We have just published a new article about AI automation.
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

Supported post types and media formats vary by network. Use `networks.list` and `networks.list_properties` to determine the capabilities available for a particular network and account type.

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
