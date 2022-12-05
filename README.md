# Using Webhook in Project for Event-Driven Communication.
Webhooks can also be used to trigger event-driven wrokflows. For example, you can set up a hook that runs a redeploy script for your project, whenever you push changes to specified branch of your project.

Setting up [Webhook](https://github.com/adnanh/webhook) for linux

## Step 1: Install the webhook
Following command will install community packaged version of [Webhook](https://github.com/adnanh/webhook)

```terminal 
sudo apt install webhook
```

## Step 2: Configure a hook file
Webhook supports files written in JSON or YAML format. We will focus on JSON example here. 

Make a hooks.json file at the same location where you have your project 
```terminal
nano hooks.json
```
In this file we will configure hook(s) that we want our webhook to trigger on specified events.


A hook named `redeploy-webhook` is defined below that will run a redeploy script located in `/path to your project directory/scripts`.

```json
[
    {
        "id": "redeploy-webhook",
        "execute-command": "/path to your project directory/scripts/redeploy.sh",
        "command-working-directory": "/path to your project directory/project name",
        "pass-arguments-to-command": [
            {
                "source": "payload",
                "name": "head_commit.id"
            },
            {
                "source": "payload",
                "name": "head_commit.message"
            },
            {
                "source": "payload",
                "name": "head_commit.author.name"
            },
            {
                "source": "payload",
                "name": "head_commit.author.email"
            }
        ],
        "trigger-rule": {
            "and": [
                {
                    "match": {
                        "type": "payload-hash-sha256",
                        "secret": "your secret value",
                        "parameter": {
                            "source": "header",
                            "name": "X-Hub-Signature-256"
                        }
                    }
                },
                {
                    "match": {
                        "type": "value",
                        "value": "refs/heads/master",
                        "parameter": {
                            "source": "payload",
                            "name": "ref"
                        }
                    }
                }
            ]
        }
    }
]

```
**Only id and execute-command parameters are requireed in hooks.json file other parameters are optional.**

- id - specifies the *name* of your hook. This value is used to create the HTTP endpoint `http://yourserver:port/hooks/your-hook-id`

- execute-command - specifies the command that you want to be executed when the hook is triggered.

- command-working-directory - specifies the working directory that will be used for the script when it's executed.

- pass-arguments-to-command - specifies the list of arguments that will be passed to the command. [Referencing Request Values Page](https://github.com/adnanh/webhook/blob/master/docs/Referencing-Request-Values.md)

- trigger-rule - specifies the exact circumstances under which the hook would be triggered. Please Check out [Hook Rules Page](https://github.com/adnanh/webhook/blob/master/docs/Hook-Rules.md) for detailed list of Available rules and their usage. Here we have used `AND` rule it evaluates to true iff all sub rules evaluate to true. `secret` used in trigger rule is used to set a secret value to add privacy to your hook. 
To see the detailed description of what properties a hook can contain [Hooks Defination](https://github.com/adnanh/webhook/blob/master/docs/Hook-Definition.md)

***Bash script that you want to run by your hook should have  `#!/bin/sh`  shebang on top.***

## Step 3: Run your webhook
Now its time to start your webhook.

`{path where your webhook exe is} -hooks {hook file name} -verbose`
```terminal
    webhook -hooks hooks.json -verbose
```	
Webhook will start up on default port 9000 and will provide you with one HTTP endpoint `http://yourserver:9000/hooks/{id}`. This endpoint will be used to add a webhook to your git repository.

## Step 4: Add the webhook to your git repository

To add a webhook in your repository follow the steps given below

- Go to the settings page of your repository or organization.

- Click Webhooks

- click Add webhook

- Copy the HTTP endpoint you got in step3 to `Payload URL`.

***Note: If you are in your local environment then the HTTP endpoint you got in step 3 will not be reachable by github as it is not public. So all you have to do is to expose your local environment to the internet. You can use a proxy agent `Ngrok` for this purpose. Type `ngrok http {port number}` on your terminal. It will generate a public address `https://{hexa-numbers}.ap.ngrok.io` copy it and replace it with `http://yourserver:9000/` in your github repository webhook.***

- Add the secret value(that you used in your hook file).

For any further issues while adding a webhook in project repository [Visit Github Docs](https://docs.github.com/en/developers/webhooks-and-events/webhooks/creating-webhooks)


Congrats!!! you are ready to go just push something to your specified repository, it will invoke the hook to run the command in your hooks.json.
