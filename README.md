# Github integration for Mattermost

Inspired by [mattermost-integration-gitlab](https://github.com/NotSqrt/mattermost-integration-gitlab) this program creates a server using [flask](https://github.com/mitsuhiko/flask) that listens for incoming GitHub event webhooks. These are then processed, formatted, and eventually forwarded to Mattermost where they are displayed inside a specified channel.
![](preview.png)

## Requirements
- Python
- Flask (install with `pip install flask`)
- requests (install with `pip install requests`)

## Usage
Copy `config.template` to `config.py` and edit it with your details. For example:

```python
from os import environ
USERNAME = "Github"
ICON_URL = ""
MATTERMOST_WEBHOOK_URLS = {
    'default' : (environ['HOOK_URL'], environ['CHANNEL_NAME']),
}
SECRET = ""
SHOW_AVATARS = True
SERVER = {
    'hook': "/",
    'address': "0.0.0.0",
    'port': 5000,
}

```

GitHub messages can be delegated to different Mattermost hooks. The order is as
follows. First try to find a hook for the repositories full name.  If that
fails, try to find a hook for the organisation name. Otherwise use the default
hook. Repositories can be blacklisted by setting them to `None` instead of
`(url, channel)`.

The server is listening by default on address `0.0.0.0`, port `5000`, and
using `/` as base route.
Make sure to point your Github webhooks to `http://yourdomain.org:5000/`.

If you have a proxy/load-balancer in front of your machine, and do not want to
expose port 5000 to the outside, change the `SERVER['hook']` value and redirect it
to this service.
For example, if `SERVER['hook']` is `/hooks/github`, your Github webhooks
would be `http://yourdomain.org/hooks/github`.

If you don't want to use a secret set the field to `None`.

Start the server with `python server.py`.

### Deploying with Docker

To deploy with Docker, make sure you have Docker installed and run:

```
docker build --rm=true -t mm-github .
```
Environment variables 
HOOK_URL     : url to mattermost webhook
CHANNEL_NAME : channel which should be notified

If you want to run in background mode, change the option `--rm` for `-d`.


## Supported Events

Not all Github events are forwarded to Mattermost. Currently supported events are:

* Ping events (send when first adding the Github webhook)
* Commit pushes and comments
* Issues (open, close, comment)
* Pull Requests (create, merge, remove, comment)
* Create/Delete repositories
* Create/Delete branches and tags

All other events will report back to GitHub with `400 Not Implemented`.

## Known issues

- Channel names need to use the spelling that is used in their URL (the channel ID), e.g. instead
of `Town Square` it needs to be `town-square`.

- If you set a custom username (as shown in the default config), make sure you also set **Enable webhooks and slash commands to override usernames** under **Custom Integrations** in the System Console to **True**. Otherwise the bots username will be that of the person that setup the Mattermost integretation.
