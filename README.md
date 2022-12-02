# Using Webhook in Project for Event-Driven Communication.
Webhooks can also be used to trigger event-driven wrokflows. For example, you can set up a hook that runs a redeploy script for your project, whenever you push changes to the master branch of your project.

Setting up [Webhook](https://github.com/adnanh/webhook) for linux
## Step 1: Install the webhook
Following command will install community packaged version of [Webhook](https://github.com/adnanh/webhook)

```    
sudo apt-get install webhook
```

## Step 2: Configure a hook file
Webhook support files written in JSON or YAML format. We will focus on JSON example here. 

Make a hooks.json file at the same location where you have your project 
```
nano hooks.json
```
This file will contain an array of hooks the webhook will serve.


Simple hook named `redeploy-webhook` is defined that will run a redeploy script located in `/path to your project directory/scripts`.
```
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
                        "secret": "afdafdag",
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

- id - specifies the ID of your hook. This value is used to create the HTTP endpoint `http://yourserver:port/hooks/your-hook-id`

- execute-command - specifies the command that should be executed when the hook is triggered

- command-working-directory - specifies the working directory that will be used for the script when it's executed.

- pass-arguments-to-command - specifies the list of arguments
To see the detailed description of what properties a hook can contain [Hooks Defination](https://github.com/adnanh/webhook/blob/master/docs/Hook-Definition.md)

***Bash script that you want to run by your hook should have  `#!/bin/sh`  shebang on top.***
## Step 3: Run your webhook
In this step you will run your webhook to serve your hooks file.

`{path where your webhook exe is} -hooks {hook file name} -verbose` will run your webhook.
```
    webhook -hooks hooks.json -verbose
```	
It will start up on default port 9000 and will provide you with one HTTP endpoint `http://yourserver:9000/hooks/{id}`

*However, hook defined like this could pose a security threat to your system, because anyone who knows your endpoint, can send a request and execute your command. To prevent that, you can use the **"trigger-rule"** property for your hook, to specify the exact circumstances under which the hook would be triggered. For example, you can use them to add a secret that you must supply as a parameter in order to successfully trigger the hook. Please check out the Hook rules page for detailed list of available rules and their usage.*

## Step 4: Add the webhook to your git repository
Now you have to add your hook to your project repo. So to add a webhook in your repository
1. Go to the settings page of your repository or organization.
2. Click Webhooks
3. click Add webhook
4. Copy the HTTP endpoint you got in step3 to `Payload URL` and for setting up other options [click here](https://docs.github.com/en/developers/webhooks-and-events/webhooks/creating-webhooks)
5. Add Webhook
## Step 5: Make your local environment public
Mostly after correctly following the above steps you will get a red icon on github webhook page showing `Unable to connect to server`. This is because you are doing this on local server or local environment.

So all you have to do is to expose your local environment to the internet *(make it public so github can find it out)*. 

For this you can use a proxy agent `Ngrok`.

Type `ngrok http {port number}` on your terminal.

It will generate a public address `https://{hexa-numbers}.ap.ngrok.io` copy it and replace it with `http://yourserver:9000/` in your github repository webhook.

## Step 6:
Congrats!!! you are ready to go just push something to repo, it will invoke the hook to run the command in your hooks.json.
