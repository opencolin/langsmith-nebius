# Configure Nebius Token Factory as the default LangSmith provider

This guide walks through wiring **Nebius Token Factory** into LangSmith as an **OpenAI Compatible Endpoint**, then promoting it to the default model for Fleet (and any other LangSmith feature you want it to power).

You will end up with:

- A Model Configuration named **Nebius/Qwen** that points at `https://api.tokenfactory.nebius.com/v1/`
- A workspace secret named **`NEBIUS_API_KEY`** holding your Token Factory key
- Fleet (and Playground/Evaluators) using that configuration as the default model

Estimated time: ~3 minutes.

---

## Prerequisites

- A LangSmith workspace where you have permission to edit Settings → Integrations.
- A Nebius Token Factory API key. Create one at <https://tokenfactory.nebius.com/>.
- The exact Token Factory model ID you want to use (e.g. `Qwen/Qwen3.5-397B-A17B-fast`).

---

## Step 1 — Open Model configurations

In the LangSmith left sidebar, expand **Integrations** and click **Model configurations**.

![Step 1](1-click%20model%20configurations.png)

## Step 2 — Click Create

On the **Configurations** page, click the blue **+ Create** button.

![Step 2](2-click%20create%20under%20configurations.png)

## Step 3 — Select "OpenAI Compatible Endpoint" as the provider

In the **Provider** dropdown, scroll to the **Other** section and pick **OpenAI Compatible Endpoint**. (Nebius is not a first-class provider in LangSmith — that's fine, the OpenAI-compatible path is exactly what Token Factory expects.)

![Step 3](3-%20select%20openai%20compatibile%20endpoint.png)

## Step 4 — Type the model name

LangSmith's **Model** field is a free-text dropdown. Paste the Token Factory model ID directly — for example:

```
Qwen/Qwen3.5-397B-A17B-fast
```

If the model isn't in the autocomplete list, the helper text confirms you can type it in.

![Step 4](4-copy%20paste%20the%20model%20name.png)

## Step 5 — Set the API Key Name to `NEBIUS_API_KEY`

This is the **name of the workspace secret** that LangSmith will read at request time, not the key itself. Use:

```
NEBIUS_API_KEY
```

You'll create the matching secret in Step 14–16.

![Step 5](5-set%20api%20key%20name%20to%20NEBIUS_API_KEY.png)

## Step 6 — Set the Base URL to the Token Factory endpoint

Paste the Token Factory v1 endpoint into **Base URL**:

```
https://api.tokenfactory.nebius.com/v1/
```

The trailing slash matters — keep it.

![Step 6](6-copy%20paste%20the%20Token%20Factory%20URL.png)

## Step 7 — Scroll down to the Provider Config section

Below the Temperature/Max Tokens area you'll find a **Provider Config** block. Scroll down to it.

![Step 7](7-scroll%20down.png)

## Step 8 — Switch Provider API from Responses to Chat Completion

Token Factory implements the OpenAI **Chat Completions** API, **not** the newer Responses API. Change the dropdown from **Responses (Recommended)** to **Chat Completion**.

> Skipping this step is the most common cause of 404s and "unknown route" errors against Token Factory.

![Step 8](8-change%20Responses%20to%20Chat%20Completion.png)

## Step 9 — Save the configuration

Click **Save** at the bottom of the dialog.

![Step 9](9-click%20save.png)

## Step 10 — Confirm the config appears in the list

You should now see a single row in **Configurations** — provider `OpenAI Compa…`, model `Qwen/Qwen3.5-397B…`. Give it a recognizable name (e.g. **Nebius/Qwen**) if it isn't already named.

![Step 10](10-you'll%20see%20your%20config.png)

## Step 11 — Open the Available Models dropdown for Fleet

Scroll to **Feature Access** on the same Model configurations page. Each LangSmith feature (Playground, Evaluators, Fleet, Polly, Insights, Issues Agent…) has its own provider/model controls. Click the **Available Models** dropdown on the **Fleet** row.

![Step 11](11-select%20the%20available%20models%20drop%20down%20for%20Fleet.png)

## Step 12 — Enable Nebius/Qwen for Fleet

In the dropdown, scroll to the **Workspace models** section at the bottom and tick **Nebius/Qwen**. This makes the configuration *eligible* to be used by Fleet.

![Step 12](12-select%20Nebius%20as%20an%20available%20model.png)

## Step 13 — Set Nebius/Qwen as the Default Model

Now use the **Default Model** column on the Fleet row and pick **Nebius/Qwen**. From this point on, Fleet routes traffic to Token Factory unless overridden per-run.

Repeat Steps 11–13 for any other features you want backed by Nebius (Playground, Evaluators, etc. — they each have their own row).

![Step 13](13-select%20Nebius%20as%20the%20default%20model.png)

## Step 14 — Open Provider secrets

The configuration references a secret called `NEBIUS_API_KEY` but that secret doesn't exist yet. In the left sidebar under **Integrations**, click **Provider secrets**.

![Step 14](14-go%20to%20provider%20secrets.png)

## Step 15 — Click + Secret

On the **Workspace Secrets** page, click the blue **+ Secret** button.

![Step 15](15-add%20a%20Secret.png)

## Step 16 — Add the NEBIUS_API_KEY value

In the **Add secret** dialog:

- **Key:** `NEBIUS_API_KEY` (must match exactly what you typed in Step 5)
- **Value:** your Token Factory API key from <https://tokenfactory.nebius.com/>

Click **Save**.

![Step 16](16-enter%20the%20NEBIUS_API_KEY.png)

---

## Verify it works

1. Go to the **Playground** (or open a Fleet run).
2. Confirm the model selector shows **Nebius/Qwen** as the default.
3. Send a test message — a successful response means the Base URL, API key, and Chat Completion toggle are all wired correctly.

If you see an auth error, double-check that the secret **Key** in Step 16 matches the **API Key Name** in Step 5 character-for-character (`NEBIUS_API_KEY`). If you see a 404 or "unknown route", revisit Step 8 and make sure Provider API is set to **Chat Completion**.

---

## Notes

- The same configuration can serve any Token Factory model — just create additional Model Configurations with different model IDs and reuse the same `NEBIUS_API_KEY` secret.
- The **API Key Name** is a *reference* to a workspace secret, not the literal key. This is what lets you rotate the underlying token without editing every model configuration.
- "Default Model" is a per-feature setting. Setting it for Fleet does **not** automatically set it for Playground, Evaluators, Insights, etc. — repeat Steps 11–13 for each feature you care about.
