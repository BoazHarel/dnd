# Implementation Plan: POV D&D Game (Discord Bot with Asymmetrical Perception)

This plan outlines the steps to build a 2-player multiplayer Dungeons and Dragons game where players interact via private Discord channels with an AI Dungeon Master (powered by Gemini), experiencing the game through asymmetrical information and filtered perceptions.

## User Review Required

> [!IMPORTANT]
> **Hosting & Bot Architecture (WebSockets vs. Firebase Functions)**
> Firebase Cloud Functions are designed for short-lived, stateless HTTP/event invocations. Running a long-running Discord WebSocket bot (using `discord.js` gateway to listen for raw player messages in channels) is not natively supported or practical in Firebase Cloud Functions.
> 
> We propose two main architectural choices:
> 1. **Option A (Recommended for simplicity & testing)**: Implement the Discord Bot as a long-running Node.js/TypeScript application (e.g., using `ts-node` or compiled JS) running in a persistent container, VPS, or local environment, using Firestore for state persistence.
> 2. **Option B (Serverless Webhooks)**: Deploy Firebase Cloud Functions using Discord's **Interactions Endpoint** (Webhooks). Players would interact using Slash Commands (`/action`, `/join`, etc.) instead of freeform text messages. This complies fully with serverless architecture but limits raw chat interactions.
> 
> *Our plan defaults to **Option A** (long-running bot process), as it matches the requirement of players chatting directly in private channels.*

## Open Questions

> [!NOTE]
> 1. **Image Generation Model**: The Gemini API offers Imagen 3 (via model ID `imagen-3.0-generate-002`) for image generation. Do you have a preferred model, or should we use the standard Imagen 3 API?
> 2. **Firebase Storage**: Since Gemini-generated images are returned as base64/bytes, we will need to host them to display them on Discord. We can upload them to Firebase Storage and get a public download URL. Is Firebase Storage setup acceptable?

## Proposed Changes

We will organize the project in the folder `/home/boaz/CascadeProjects/dnd` with the following structure:
- `firebase.json`, `firestore.rules`, `firestore.indexes.json` - Firestore configuration
- `src/` - Application source code
  - `config.ts` - Configuration & API clients (Gemini, Discord)
  - `db.ts` - Firestore operations and schema helper functions
  - `bot.ts` - Discord bot gateway events and interaction handlers
  - `game.ts` - Game state machine, turn resolution, and Gemini prompt integrations
  - `index.ts` - Main entrypoint to boot up the bot
- `package.json`, `tsconfig.json` - Dependency and TypeScript configuration

---

### [Component: Database Schema & Configuration]

We will initialize the database helpers, TypeScript interfaces, and Firestore security rules.

#### [NEW] [firestore.rules](file:///home/boaz/CascadeProjects/dnd/firestore.rules)
Basic locked-down firestore rules.

#### [NEW] [src/db.ts](file:///home/boaz/CascadeProjects/dnd/src/db.ts)
TypeScript interfaces matching the proposed Firestore schema:
- `Game`: tracks `theme`, `lore`, `groundTruth`, `currentTurn`, `status`.
- `Player`: tracks `id`, `gameId`, `channelId`, `name`, `traits`, `states`, `currentAction`.
- `Turn`: tracks actions, perceptions, and image URLs.
Helper functions to:
- `createGame(...)`
- `addPlayer(...)`
- `submitAction(...)`
- `getCurrentTurn(...)`
- `resolveTurn(...)`

---

### [Component: Discord Bot Service]

The bot gateway handles command registration, channel creation, and listening for message events.

#### [NEW] [src/bot.ts](file:///home/boaz/CascadeProjects/dnd/src/bot.ts)
Implementation of the Discord client using `discord.js`:
- `/startgame [theme]` slash command: Creates a new game instance.
- `/join` slash command: Registers player, creates a private text channel in the guild (readable only by the player and bot).
- Message listener: Listens to player messages in their respective private channels, saving them as their current turn actions.

---

### [Component: AI Dungeon Master (Gemini Loop)]

Handles prompt engineering, JSON response parsing, and image generation.

#### [NEW] [src/game.ts](file:///home/boaz/CascadeProjects/dnd/src/game.ts)
Logic to integrate with the `@google/genai` SDK:
- Lore generation for initialization.
- Character creation suggestions.
- Compound turn-resolution prompt:
  - Formulate structured prompt using previous ground truth, players' actions, and traits.
  - Call Gemini to get structured output (`groundTruth`, `playerA_POV`, `playerB_POV`, `imagePromptA`, `imagePromptB`).
  - Generate images using Imagen 3 model.
  - Upload images to Firebase Storage or send them as attachments directly to Discord.

---

### [Component: Main Entrypoint]

#### [NEW] [src/index.ts](file:///home/boaz/CascadeProjects/dnd/src/index.ts)
Initializes Firebase Admin SDK and starts the Discord client bot.

#### [NEW] [package.json](file:///home/boaz/CascadeProjects/dnd/package.json)
Manages dependencies: `discord.js`, `@google/genai`, `firebase-admin`, `dotenv`, `typescript`, `ts-node`, `@types/node`.

---

## Verification Plan

### Automated Tests
- We will set up local Firestore emulator to test database helpers without hitting live Firebase.
- We will write basic unit tests for state updates and JSON response parser.

### Manual Verification
- Start the bot locally using local credentials (`.env`).
- Connect two test users/accounts on a dedicated Discord server.
- Run `/startgame` and `/join` and verify private channel creation.
- Submit actions and verify that the bot responds with unique POVs and generated images for both players.
