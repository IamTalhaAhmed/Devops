# Setting Up HookDeck Webhook [Visit Github](https://github.com/adnanh/webhook)
## WHAT ARE WEBHOOKS
A webhook is an HTTP-based callback function that allows lightweight, event-driven communication between 2 application programming interfaces (APIs). Webhooks are used by a wide variety of web apps to receive small amounts of data from other apps, but webhooks can also be used to trigger automation workflows in GitOps environments. For example, if you're using Github or Bitbucket, you can use webhook to set up a hook that runs a redeploy script for your project on your staging server, whenever you push changes to the master branch of your project.
## You can set up a HookDeck Webhook by following the steps given below.

## Step 1: Install the webhook
First of all you have to install the Webhook.If you are using Ubuntu linux (17.04 or later), following command will install community packaged version.

```    
sudo apt-get install webhook
```

## Step 2: Create a file hooks.json or hooks.yml
After installation of webhook you have to define some hooks that you want webhook to serve. Webhook supports YAML and JSON configration files but we will focus on JSON example here. 

So make a hooks.json file.
```
nano hooks.json
```
This file will contain an array of hooks the webhook will serve.

***Bash script that you want to run by your hook should have  `#!/bin/sh`  shebang on top.***

### Its time to set up your hooks.json: 

Simple hook named `redeploy-webhook` is defined that will run a redeploy script located in `/home/username/projectdir/scripts`.
```
[
  {
    "id": "redeploy-webhook",
    "execute-command": "/home/username/projectdir/scripts/redeploy.sh",
    "command-working-directory": "/var/webhook"
  }
]
```
**Only id and execute-command parameters are requireed in hooks.json file other parameters are optional.**

- id - specifies the ID of your hook. This value is used to create the HTTP endpoint `http://yourserver:port/hooks/your-hook-id`

- execute-command - specifies the command that should be executed when the hook is triggered

- command-working-directory - specifies the working directory that will be used for the script when it's executed.

To see the detailed description of what properties a hook can contain [Hooks Defination](https://github.com/adnanh/webhook/blob/master/docs/Hook-Definition.md)


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
