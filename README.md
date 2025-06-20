# Karmabot for Slack

[![Build Status](https://github.com/deekim/glitter/actions/workflows/main.yml/badge.svg)](https://github.com/deekim/glitter/actions)
![License](https://img.shields.io/github/license/deekim/glitter)
![Python Version](https://img.shields.io/badge/python-3.8%2B-blue)

A simple, robust Slack Karma bot written in Python.

## Features
- Karma rate-limiting (default: 60 operations/hour per user)
- Separate karma for things, users, groups, and channels
- Top/Bottom lists
- Karma expiration
- General statistics
- Badges (optional)

## What is Karma?
Karma is a reputation system for Slack. Add `++` or `--` to the end of a subject to give or remove karma. Anyone (except bots) can give karma. It's a fun way to show appreciation in your workspace.

## Setup
1. **MongoDB:** Set up a persistent MongoDB instance.
2. **Metrics (optional):** If you want metrics, set up a service that accepts InfluxDB line protocol (see `docker-compose.yml` for an example).
3. **Slack App:**
   - Create an app at [api.slack.com/apps](https://api.slack.com/apps).
   - Add `/karma` and `/badge` commands, pointing to your bot's HTTP endpoint.
   - Enable event subscriptions for `app_mention`, `message.channels`, and `message.groups`.
   - Set OAuth permissions: `bot`, `commands`, `channels:write`, `chat:write:bot`, `im:write`, `usergroups:read`.
   - Invite the bot to channels you want to track.
4. **Configuration:**
   - Set environment variables (see below).
   - Start the bot (Docker recommended).

### Environment Variables
- `VERIFICATION_TOKEN`: Slack app verification token (required)
- `MONGODB`: MongoDB URI (default: `mongodb://localhost:27017`)
- `SLACK_EVENTS_ENDPOINT`: Slack events endpoint (default: `/slack_events`)
- `KARMA_RATE_LIMIT`: Karma ops/hour/user (default: `60`)
- `KARMA_TTL`: Karma expiration in days (default: `90`)
- `KARMA_COLOR`: Message color (default: `#af8b2d`)
- `FAKE_SLACK`: Set to `True` for testing (mocks Slack)

For multi-workspace support, see the original project for Vault integration details.

## Architecture
- **Flask** web service for Slack events
- **MongoDB** for storing karma operations
