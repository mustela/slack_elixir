# Slack

This is for creating Slack applications or bots in Elixir.

It uses **Socket Mode** to connect to Slack, which has some restrictions, so
please read up on that.

## Installation

Add `slack_elixir` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:slack_elixir, "~> 0.1.0"}
  ]
end
```

## Setup

You will need to:

  - Create a Slack app for your workspace
  - Add permissions (scopes)
  - Connect it to your workspace
  - Get an OAuth Bot Token (will have the scopes you defined)
  - Enable Socket Mode
  - Get an app-level token with `connections:write` scope
  - Add Event Subscriptions

Write the Bot module:
  
```elixir
defmodule MyApp.Slackbot do
  @behaviour Slack.Bot

  @impl true
  def handle_event("message", %{"app_id" => _}) do
    # Ignoring app/bot messages.
    # This is helpful if you have a bot that responds to messages,
    # and you don't want it to respond to itself.
    :ok
  end

  def handle_event("message", payload) do
    if String.match?(payload["text"], ~r/hello/i) do
      reply = "Hello! <@#{payload["user"]}>"
      {:reply, channel: payload["channel"], text: reply}
    else
      :ok
    end
  end

  def handle_event(type, payload) do
    :ok
  end
end
```

Then you start the Slack Supervisor in your application's supervision tree.

For example:

```elixir
  def start(_type, _args) do
    # Often, you'd fetch this from application env,
    # set in `config/runtime.exs`, instead of like this.
    config = [
      app_token: "MY_SLACK_APP_TOKEN",
      bot_token: "MY_SLACK_BOT_TOKEN",
      bot: MyApp.SlackBot
    ]

    children = [
      # ...
      {Slack.Supervisor, config}
    ]

    opts = [strategy: :one_for_one, name: MyApp.Supervisor]
    Supervisor.start_link(children, opts)
  end
```