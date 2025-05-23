# DNS Monitor Bot - Discord Edition

A simple to configure, pre-built Cloudflare Worker that monitors DNS records for any list of user-specified domains and sends notifications via Discord when changes are detected.

The project is designed to stay comfortably within Cloudflare's free tier for its Worker and KV storage services.

<p align="center">
  <img src="images/example_alert.png" alt="Example Discord alert" />
  <br/>
  <i>Example Discord alert</i>
</p>

## Prerequisites

- [Node.js](https://nodejs.org/) (v20 or later)
- [npm](https://www.npmjs.com/) (comes with Node.js)
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/install-and-update/) (v4 or later)

## Setup

1. **Clone the repository:**

   ```bash
   git clone https://github.com/naouflex/dns-bot.git
   cd dns-bot
   ```

2. **Install dependencies:**

   ```bash
   npm install
   ```

3. **Configure your bot and secrets:**

   - Create a `.env` file in the project root and supply values:

     ```bash
     cp .env.example .env
     ```

   - Supply the same variables and values as GitHub Actions secrets within your repository's settings.[^1]

   - Update `config.json` with your settings:

     ```json
     {
       "domains": ["domain1.com", "domain2.com"],
       "cron": "*/5 * * * *",
       "kvNamespace": {
         "id": "your-kv-namespace-id"
       }
     }
     ```

   - Get your Cloudflare API token[^2]
   
   - Create a Discord webhook[^3]

   - Get your Discord role id from the Discord server you want to monitor[^4]

4. **Deploy the bot:**

   - **Option 1: Deploy locally**

     Run the deploy script:

     ```bash
     npm run deploy
     ```

     This will:

     - Set up the KV namespace if needed
     - Configure Discord webhook
     - Update the worker configuration
     - Deploy to Cloudflare Workers

   - **Option 2: Deploy via GitHub Actions**

     - Push your changes to the `main` branch.
     - The GitHub Action will automatically deploy the bot.

## Viewing Logs

To view the logs for your deployed worker:

1. Go to the [Cloudflare Dashboard](https://dash.cloudflare.com/).
2. Navigate to **Workers & Pages**.
3. Select your worker (`dns-bot`).
4. Click on **Logs** to view the worker's logs.

## Alerting

The DNS Monitor Bot sends several types of notifications to your Discord channel:

### DNS Change Detected (Orange)
Triggered when IP addresses for a domain change. Includes:
- Previous and new IP addresses
- TTL information
- DNS status
- SOA serial number, primary nameserver, and admin email
- Mentions the specified Discord role to alert team members

### DNS Zone Updated (Light Blue)
Triggered when the SOA record changes but IP addresses remain the same. Includes:
- Previous and new SOA serial numbers 
- Primary nameserver information
- Admin email
- Zone timing parameters (refresh, retry, expire, minimum TTL)

### DNS Authority Unreachable (Yellow)
Triggered when the DNS authority for a domain becomes unreachable. Includes:
- Domain name
- Status information
- DNS comments from the resolver

### Error Monitoring DNS (Red)
Triggered when there's an error checking a domain. Includes:
- Error details
- Timestamp of the error

### New Worker Deployment (Light Blue)
Triggered when a new version of the worker is deployed. Includes:
- Previous and new version IDs
- List of monitored domains
- Deployment timestamp

## Troubleshooting

- **Wrangler not found:** Ensure Wrangler is installed globally or use `npx wrangler`.
- **Deployment fails:** Check your API token and ensure all environment variables are set correctly.
- **No logs:** Ensure logging is enabled in your `wrangler.toml` file.
- **GitHub Actions fails:** Verify that all required secrets are set in your repository's Settings > Secrets and variables > Actions.
- **No Discord notifications:** Check that your webhook URL is correct and that the bot has permission to post in the channel.

## Footnotes

[^1]: Required secrets must be set in both your local `.env` file and GitHub Actions repository secrets. Go to your repository's Settings > Secrets and variables > Actions and add: `CLOUDFLARE_API_TOKEN`, `DISCORD_WEBHOOK_URL`, and `DISCORD_ROLE_ID`.

[^2]: To get your Cloudflare API token:

    1. Go to the [Cloudflare Dashboard](https://dash.cloudflare.com/)
    2. Navigate to **My Profile** > **API Tokens**
    3. Click **Create Token**
    4. Choose **Create Custom Token**
    5. Set the following permissions:
       - **Account** > **Workers** > **Edit**
       - **Zone** > **DNS** > **Read**
    6. Set the **Account Resources** to **All accounts**
    7. Set the **Zone Resources** to **All zones**
    8. Click **Continue to summary** and then **Create Token**

[^3]: To create a Discord webhook:

    1. Open Discord and go to the server where you want to receive notifications
    2. Go to Server Settings > Integrations > Webhooks
    3. Click "New Webhook"
    4. Give it a name (e.g., "DNS Monitor")
    5. Select the channel where notifications should be sent
    6. Click "Copy Webhook URL"
    7. Paste this URL as your `DISCORD_WEBHOOK_URL` in the `.env` file and GitHub secrets

[^4]: To get your Discord role id:

    1. Open Discord and go to the server where you want to receive notifications
    2. Go to Server Settings > Roles
    3. Hover over the role you want to monitor and click "Copy ID"
