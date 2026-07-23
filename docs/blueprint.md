# Predictor Sports Forecast Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Predictor is a Telegram bot that provides probabilistic match outcome forecasts (win/draw/loss percentages) for football and basketball to sports analysts. Analysts can receive predictions via private messages, group/channel posts, or scheduled daily summaries, and manage subscriptions to specific leagues/teams.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- sports analysts

## Success criteria

- Analysts can receive and manage probabilistic match forecasts for football and basketball
- Analysts can customize delivery preferences and watchlists
- Scheduled summaries are delivered at specified times

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu for onboarding and settings
- **/predict** (command, actor: user, command: /predict) — Request a prediction for a specific match or team
  - inputs: match or team identifier
  - outputs: probabilistic forecast with rationale
- **Manage Subscriptions** (button, actor: user, callback: subscriptions:manage) — Add/remove leagues/teams from watchlist
  - inputs: league/team identifiers
  - outputs: updated subscription status
- **Set Delivery Preferences** (button, actor: user, callback: delivery:preferences) — Configure how and when to receive predictions
  - inputs: delivery method, time
  - outputs: updated delivery settings
- **View History** (button, actor: user, callback: history:view) — See past predictions and forecasts
  - inputs: none
  - outputs: forecast history

## Flows

### Onboarding
_Trigger:_ /start

1. Welcome message
2. Request delivery preference
3. Select initial watchlist (default leagues)
4. Confirm settings

_Data touched:_ Analyst user, Subscription, Delivery setting

### On-demand Prediction
_Trigger:_ /predict

1. Prompt for match/team identifier
2. Validate identifier
3. Generate and display forecast
4. Offer to add to watchlist

_Data touched:_ Match, Forecast

### Scheduled Summary
_Trigger:_ daily schedule

1. Compile upcoming matches from subscriptions
2. Format summary with probabilities
3. Deliver to configured targets

_Data touched:_ Match, Forecast, Delivery setting

### Subscription Management
_Trigger:_ subscriptions:manage

1. Display current subscriptions
2. Prompt for additions/removals
3. Validate and update subscriptions

_Data touched:_ Subscription

### Error Handling
_Trigger:_ invalid input or data failure

1. Detect error condition
2. Generate human-readable error message
3. Offer to retry or adjust input

_Data touched:_ none

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Analyst user** _(retention: persistent)_ — Telegram user account with preferences and subscriptions
  - fields: Telegram id, delivery preferences, subscriptions
- **Match** _(retention: persistent)_ — Scheduled sports match with teams and competition
  - fields: teams, date/time, competition
- **Forecast** _(retention: persistent)_ — Probabilistic outcome prediction for a match
  - fields: win probability, draw probability, loss probability, confidence, timestamp
- **Subscription** _(retention: persistent)_ — Competitions/teams an analyst follows
  - fields: competitions, teams
- **Delivery setting** _(retention: persistent)_ — How and when predictions are delivered
  - fields: delivery method, target chat, schedule time

## Integrations

- **Telegram** (required) — Bot API messaging and scheduled delivery
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Admin notifications for delivery failures
- Add/remove seeded competitions
- Adjust data retention policy

## Notifications

- Scheduled daily summaries to configured targets
- Error notifications to admin chat

## Permissions & privacy

- Store user preferences and subscriptions for 12 months
- Only deliver forecasts to explicitly configured targets

## Edge cases

- Invalid match/team identifiers
- Failed delivery to group/channel
- No upcoming matches in subscriptions

## Required tests

- End-to-end onboarding flow with default subscriptions
- Scheduled summary delivery at configured time
- On-demand prediction with valid and invalid inputs

## Assumptions

- Seeded competitions include top 6 football leagues and NBA
- Prediction format includes win/draw/loss probabilities with optional rationale
- Data retention is 12 months by default
