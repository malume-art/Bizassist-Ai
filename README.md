# BizAssist AI — starter MVP

A starter web dashboard for multi-user business assistants with English/Chichewa support, OpenAI-powered replies, automatic/manual/hybrid modes, and WhatsApp QR/pairing-code linking through a WhatsApp Web-compatible library.

## Important WhatsApp note

This starter uses an **unofficial WhatsApp Web-compatible integration** for QR/pairing-code linking. It is not the official WhatsApp Business Platform, may break when WhatsApp changes, and can result in account restrictions. No implementation can guarantee a number will never be restricted. For a commercial product, review Meta's current terms and consider the official WhatsApp Business Platform before onboarding real businesses. Use a test account first and obtain consent from message recipients.

## Requirements

- Node.js 20 or newer
- npm
- An OpenAI API key (API usage is billed separately from a ChatGPT subscription)

## Run locally

1. Copy `server/.env.example` to `server/.env` and add your OpenAI API key.
2. From the project root:
   ```bash
   npm install
   npm install --prefix server
   npm install --prefix client
   npm run dev
   ```
3. Open `http://localhost:5173`.
4. Register a user, add business details, then connect WhatsApp from the Connections section.

The API runs at `http://localhost:4000`. SQLite data and WhatsApp session files are stored in `server/data/`.


## Deploy the dashboard on Vercel

This repository is configured so npm workspaces install the Vite/React client dependencies during Vercel's install step. `vercel.json` runs the root build and publishes `client/dist`.

1. Upload/import the **contents of this project folder** to Vercel (the folder containing `package.json` and `vercel.json`).
2. In Vercel, keep the Root Directory set to the project root. The build command and output directory are configured in `vercel.json`.
3. Deploy the API separately to a persistent Node.js host. The WhatsApp session service must not run only as short-lived serverless functions.
4. In Vercel → Settings → Environment Variables, add `VITE_API_URL` with your deployed backend API base URL, including `/api` (for example, `https://your-api-host.example/api`). Redeploy after adding it. If this variable is missing, the deployed frontend will show a configuration message instead of trying to call `localhost` on the visitor's phone/computer.
5. On the backend host, configure `OPENAI_API_KEY`, `JWT_SECRET`, and `CLIENT_ORIGIN` (the Vercel site origin), plus persistent storage for SQLite data and WhatsApp sessions.

**Note:** Vercel hosts the frontend in this setup; it does not automatically host the separate `server/` API or persist WhatsApp linked-device sessions. The frontend build can succeed while sign-in and WhatsApp features remain unavailable until the API is deployed and `VITE_API_URL` points to it. A browser message like `Failed to fetch` usually means the API is unreachable, the URL is wrong, the backend is stopped, or its CORS `CLIENT_ORIGIN` does not exactly match the deployed Vercel site origin.

## Production checklist before launch

This is a functional starter, not yet a hardened public SaaS. Before public deployment, add email verification, password reset, rate limiting, CSRF strategy appropriate to your auth design, audit logs, encrypted secrets at rest, backups, admin tooling, subscription billing, privacy/retention controls, abuse monitoring, and a persistent server/volume for WhatsApp sessions. Use HTTPS. Do not deploy the WhatsApp session service as short-lived serverless functions.

## Modes

- **Automatic:** AI replies are sent automatically.
- **Manual:** AI suggestions are saved for the owner to review; the owner sends them from the conversation panel.
- **Hybrid:** Routine messages can be answered automatically; messages that look like complaints or need human attention are left for review.

The AI is instructed to use business facts supplied by the owner and not invent prices, availability, policies, or booking confirmations.

## Docker deployment (single server)

1. Create a `.env` file in the project root with:
   ```env
   JWT_SECRET=replace-with-a-long-random-secret
   OPENAI_API_KEY=your-openai-api-key
   OPENAI_MODEL=gpt-4.1-mini
   ```
2. Run `docker compose up --build -d`.
3. Open `http://localhost:8080`.

The `bizassist_data` volume stores SQLite data and WhatsApp sessions. For an internet-facing deployment, configure HTTPS at a reverse proxy/load balancer, change `CLIENT_ORIGIN`, and review the production checklist above. Do not commit `.env` files.
