# Building Smart Agents with MCP and OpenAI *gpt-oss* 🔥

In this guide, we'll explore using MCP and demonstrate how to build AI agents using `gpt-oss` as the LLM backbone. We'll use Hugging Face's lightweight MCP clients:

1. `@huggingface/tiny-agents` (TypeScript)
2. `huggingface_hub[mcp]` (Python)

The key to building effective AI agents lies in their tools. MCP provides a standardized interface for tool interaction, making it simple to create powerful agents. 

Let's dive in by creating a web-savvy agent that can browse and search the internet for you.

## **Example**: Local Browser Agent

Let's build a browser agent that can browse and search the internet for you. We've already implemented this in `/browser-agent` directory in case you want to skip ahead.

### Step 0: Log in to Hugging Face

```bash
huggingface-cli login
```

This will ask you for your HF API token. You can get it from [here](https://huggingface.co/settings/tokens).

### Step 1: Define the Agent

Both the JS and Python Tiny agent clients are meant to be quite easy to play and experiment with. They expect a transparent `agent.json` which includes the details of which LLM should be used and what tools it should have access to.

Let's define our agent using OpenAI's latest `gpt-oss` 120B as the LLM and connect it to [this Playwright MCP server](https://github.com/microsoft/playwright-mcp) that provides browser automation capabilities to your agent. We've added this in `/browser-agent/agent.json` for you to use.

```json
{
	"model": "openai/gpt-oss-120b",
	"provider": "fireworks-ai",
	"servers": [
		{
			"type": "stdio",
			"command": "npx",
			"args": ["@playwright/mcp@latest"]
		}
	]
}
```

Optionally, we can define a System Prompt that helps steer the LLM. This is defined in `/browser-agent/PROMPT.md` for you to use.

> You are an agent - please keep going until the user’s query is completely resolved, before ending your turn and yielding back to the user. Only terminate your turn when you are sure that the problem is solved, or if you need more info from the user to solve the problem.
> 
> If you are not sure about anything pertaining to the user’s request, use your tools to read files and gather the relevant information: do NOT guess or make up an answer.
> 
> You MUST plan extensively before each function call, and reflect extensively on the outcomes of the previous function calls. DO NOT do this entire process by making function calls only, as this can impair your ability to solve the problem and think insightfully.
> 
> Help the User with their task.
```

That's it, let's take it out for a spin.

### Step 2: Run the agent

To run the agent in Python, we simply install the `tiny-agents` package, which is part of the `huggingface_hub` library.

```bash
pip install -U "huggingface_hub[mcp]>=0.32.0"
```

Followed by running our agent:

```bash
tiny-agents run ./browser-agent
```

![CLI Init Browser](assets/init-browser.png)

When the agent starts, you can chat with it to ask him to solve tasks. For example, try to ask it to find the top 10 Hugging Face models, and see if it's able to connect to the website using the Playwright MCP tool we configured!

> [!NOTE]
> You can do exactly the same thing with our JavaScript client as well.
>
> ```bash
> npx @huggingface/tiny-agents run ./browser-agent
> ```

![Hugging Face MCP Settings page](assets/output-browser-agent.png)

Voila, you now have a capable browser agent with you!

## **Example**: Accessing Hugging Face MCP Servers

Let's take it up a notch and give more creative freedom to our AI Agent, cue, [Hugging Face MCP Server](https://hf.co/mcp). The HF MCP server allows you to not only interact with the HF Hub but also with 1000s of AI spaces on [hf.co/spaces](https://hf.co/spaces). 

Let's get it set up!

## Step 1: Find an MCP Server

Head over to [hf.co/mcp](https://hf.co/mcp) and add the spaces/ demo that you want to be able to play with

![Hugging Face MCP Settings page](assets/hf-mcp.png)

For example, I've added the following space 

1. [evalstate/FLUX.1-Krea-dev](https://huggingface.co/spaces/evalstate/FLUX.1-Krea-dev) - a popular aesthetic text to image model by Black Forest Labs
2. [DVe0UTvm4/ltx-video-distilled](https://huggingface.co/spaces/DVe0UTvm4/ltx-video-distilled) - a popular image/ text to video by Lightricks

Next, let's update our `agent.json`:

```json
{
	"model": "openai/gpt-oss-120b",
	"provider": "fireworks-ai",
	"inputs": [
		{
			"type": "promptString",
			"id": "hf-token",
			"description": "Your Hugging Face Token",
			"password": true
		}
	],
	"servers": [
		{
			"type": "stdio",
			"command": "npx",
			"args": [
			"mcp-remote",
			"https://huggingface.co/mcp",
			"--header",
			"Authorization: Bearer ${HF_TOKEN}"
			],
			"env": {
				"HF_TOKEN": "${input:hf-token}"
			}
		}
	]
}
```

### Step 2: Run it!

Let's run it with Tiny agents just like we did with the local browser agent.

```bash
tiny-agents run ./hf-mcp-server
```

> [!NOTE]
> Again, you can do exactly the same thing with our JavaScript client as well.
> 
> ```bash
> npx @huggingface/tiny-agents run ./hf-mcp-server
> ```

[![Watch the demo video](https://img.youtube.com/vi/OEaPk3FoK7M/0.jpg)](https://youtu.be/OEaPk3FoK7M)

That's it! What would you build next with it?
