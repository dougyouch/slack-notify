# slack-notify

Command line tool for sending slack messages.

In my world, I quite often need to run long running scripts in screen(s).  This simple script comes in handy to get notifications about when my scripts complete.  So, I can move on to the next part of the process.

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
export SLACK_THREAD_TS=$(slack-notify script started)
sleep 1
slack-notify download complete
sleep 1
slack-notify processing file
sleep 1
slack-notify finished
```

# slack-message-formatter

Purpose is the conveniently replace usernames with Slack Profile IDs.

example:

```
slack-message-formatter Hey, @bob your download is ready https://example.com/download.zip
```

Outputs:
```
Hey, <@U888888IIIZ> your download is ready https://example.com/download.zip
```

### slack-message-formatter Requirements

A file that maps usernames to Slack Profile ID(s).  By default it checks for a /etc/slack-users.conf file.  To change the file's location set the SLACK\_USER\_CONF to the correct location.

example:
```
bob:U888888IIIZ
```

# Setup

Requires the Slack API token to be configured into an environment variable SLACK_TOKEN.

To create a Slack API token goto [https://api.slack.com/tutorials/tracks/getting-a-token](https://api.slack.com/tutorials/tracks/getting-a-token)

# Recommended Installation

Create a /etc/profile.d/slack.sh with

example:
```
export SLACK_TOKEN="xyzb-1111-1111-ZZ"
export SLACK_CHANNEL="#bot_notifications"
export SLACK_USERNAME=$(hostname -s)
export SLACK_ICON_EMOJI=":large_blue_circle:"
export SLACK_MESSAGE_PREFIX="$APP_ENV: "
```

Install the slack-notify script

```
curl -s https://raw.githubusercontent.com/dougyouch/slack-notify/main/bin/slack-notify -o /usr/local/bin/slack-notify
chmod 755 /usr/local/bin/slack-notify
```
