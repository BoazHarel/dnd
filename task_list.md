# MVP Task List: POV D&D Game

This document tracks the incremental tasks for building the Minimum Viable Product (MVP) of the chat-based multiplayer D&D game with unique player perspectives.

## Milestone 1: The "Hello World" Bot
- [ ] **Task 1.1**: Create a minimal Discord bot in Node.js/TypeScript that connects to your test server.
- [ ] **Task 1.2**: Make the bot respond with "Hello World!" to any message sent in a specific channel.
- [ ] **Task 1.3**: Implement a simple slash command (e.g., `/ping`) that replies with "Pong!".

## Milestone 2: Private Channels & Basic State
- [ ] **Task 2.1**: Implement a `/join` command that creates a private channel for the player who uses it, accessible only by them and the bot.
- [ ] **Task 2.2**: Set up a local Firebase project and connect a Cloud Function to handle a test command from the bot.
- [ ] **Task 2.3**: Save a "player joined" state to Firestore when the `/join` command is used.

## Milestone 3: Gemini Integration (Text)
- [ ] **Task 3.1**: Connect the Gemini API to a Cloud Function and make it respond to a simple prompt.
- [ ] **Task 3.2**: Implement `/startgame [theme]`. Gemini generates the game setting and lore based on the theme and posts it to the shared channel.
- [ ] **Task 3.3**: Implement interactive character creation via DMs or private channels, where Gemini asks questions to determine traits and saves them to Firestore.

## Milestone 4: The Core Loop (Split POV)
- [ ] **Task 4.1**: Implement turn-waiting logic. The bot listens for actions in both private channels and waits until both players have responded.
- [ ] **Task 4.2**: Send the actions and state to Gemini, asking for a JSON response with the ground truth and separate POVs for each player.
- [ ] **Task 4.3**: Deliver the unique POV descriptions back to each player's private channel.

## Milestone 5: Visuals
- [ ] **Task 5.1**: Test Gemini's native image generation with a simple static prompt and send the image to Discord.
- [ ] **Task 5.2**: Integrate image generation into the core loop, generating a unique image for each player's POV.
