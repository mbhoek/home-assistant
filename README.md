# HomeAssistant

Stuff for my Home Assistant.

- [Blueprints](blueprints/README.md): reusable automations, such as layered room lighting with buttons and motion.

## Home Assistant MCP setup (long-lived token)

1. In Home Assistant: **Profile** -> **Long-Lived Access Tokens** -> **Create Token**.
2. In PowerShell, set the instance base URL and token for your current session:

   ```powershell
   $env:HOME_ASSISTANT_URL = "<your Home Assistant base URL>"
   $env:HOME_ASSISTANT_TOKEN = "your-long-lived-token"
   ```

3. The MCP server is configured in `.mcp.json` to use these environment
   variables for its endpoint and authorization header.
4. Start Copilot CLI from this repo and verify with:

   ```text
   /mcp
   ```

If you want the token to persist across PowerShell sessions, add it with `setx`:

```powershell
setx HOME_ASSISTANT_TOKEN "your-long-lived-token"
```
