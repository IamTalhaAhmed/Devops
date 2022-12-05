# Using Github Webhook for Auto Deployment on a Linux Server 
A webhook allows APIs to do event-driven communication. It can also be used to set up auto deployment on server. For example, redeploying code automatically whenever changes are pushed to a specific branch.

## Step 1: Tool to handle webhook requests
We need an API endpoint to handle requests from github. You can make your own program for this, but we are going to use [webhook](https://github.com/adnanh/webhook) by [adnanh](https://github.com/adnanh) for this purpose.

```terminal 
sudo apt install webhook
```

## Step 2: Configure Webhook
We need to configure our tool to handle requests as we need. It needs a configuration file in JSON or YAML format.

In this file we will configure hook(s) that we want our webhook to trigger on specified events.


>hooks.json
>```json
>[
>    {
>        "id": "redeploy-webhook",
>        "execute-command": "/path to your project directory/scripts/redeploy.sh",
>        "command-working-directory": "/path to your project directory/project name",
>        "pass-arguments-to-command": [
>            {
>                "source": "payload",
>                "name": "head_commit.id"
>            },
>            {
>                "source": "payload",
>                "name": "head_commit.message"
>            },
>            {
>                "source": "payload",
>                "name": "head_commit.author.name"
>            },
>            {
>               "source": "payload",
>               "name": "head_commit.author.email"
>            }
>        ],
>        "trigger-rule": {
>            "and": [
>                {
>                    "match": {
>                    "type": "payload-hash-sha256",
>                    "secret": "your secret value",
>                    "parameter": {
>                        "source": "header",
>                        "name": "X-Hub-Signature-256"
>                        }
>                    }
>                },
>                {
>                    "match": {
>                    "type": "value",
>                    "value": "refs/heads/master",
>                    "parameter": {
>                        "source": "payload",
>                        "name": "ref"
>                        }
>                    }
>                }
>            ]
>        }
>    }
>]

>```
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

***Note: Webhook tool should stay active to trigger the specified scripts when a certain event occurs, so you want it to be running all the time. For this purpose you can make a `service file` in systemd.***
###Configuring a Service File
1. Move to /etc/systemd/system
```
cd /etc/systemd/system
```

2. Create a service file using
```
sudo nano service-filename.service
```

3. Configure your service file
```
[Unit]
Description=Run Webhook Tool

[Service]
User=<username e.g talha>
WorkingDirectory=<path of directory that contains hooks file e.g /home/talha/projectdir>
ExecStart=<command/script you want to run e.g webhook -hooks hooks.json -verbose>
Restart=always

[Install]
WantedBy=multi-user.target   
```

4. Reload the service files to include the new service 
```
sudo systemctl daemon-reload
```

5. Start your service
```
sudo systemctl start your-service.service
```

6. To check the status of your service
```
sudo systemctl status example.service
```

7. To enable your service on every reboot
```
sudo systemctl enable example.service 
```


## Step 4: Add the webhook to your git repository

To add a webhook in your repository follow the steps given below

- Go to the settings page of your repository or organization.

- Click Webhooks

- click Add webhook

- Copy the HTTP endpoint you got in step3 to `Payload URL`.

***Note: If you are in your local environment then the HTTP endpoint you got in step 3 will not be reachable by github as it is not public. So all you have to do is to expose your local environment to the internet. You can use a proxy agent `Ngrok` for this purpose. Type `ngrok http {port number}` on your terminal. It will generate a public address `https://{hexa-numbers}.ap.ngrok.io` copy it and replace it with `http://yourserver:9000/` in your github repository webhook.***

- Add the secret value(that you used in your hook file).

For any further issues while adding a webhook in project repository [Visit Github Docs](https://docs.github.com/en/developers/webhooks-and-events/webhooks/creating-webhooks)

### That's it
**Its all done now. If now you push code to your master branch, it will be automatically redeployed.**
