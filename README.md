# slack-notify

Command line tool for sending slack messages

# Basic Usage

The 1st argument can start with an @ symobl.  This indicates the slack channel to use.  The remain arguments are the message to send.

```
SLACK_CHANNEL="#general" slack-notify say hello
# or
slack-notify @#general say hello
```

The script outputs the thread timestamp.  This timestamp can be passed back to slack-notify to create a thread of messages.

```
export SLACK_CHANNEL="#notifications"
export SLACK\_THREAD\_TS=$(slack-notify script started)
sleep 1
slack-notify download complete
sleep 1
slack-notify processing file
sleep 1
slack-notify finished
```

# Setup

Requires the Slack API token to be configured into an environment variable SLACK_TOKEN.

To create a Slack API token goto [https://api.slack.com/tutorials/tracks/getting-a-token](https://api.slack.com/tutorials/tracks/getting-a-token)
