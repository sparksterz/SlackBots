# SlackBots
WAT?
----
It first helps to understand what "Brian Bot" is. Brian Bot is a custom generated "Hubot". Hubot is an open source bot that can support custom scripts and adapters which allow it to plug in to multiple different communication clients. It can host itself on a node.js infrastructure and uses coffeescript to program it's commands.

Slack helps enable Hubot integrations as well by allowing you to add them as part of their "Bot" identities in your team. Giving Hubot an API key and a public environment to run in is all that's needed for your bot to be active in your Slack team.

Here is what was used for the Brian Bot's stack:
* Node JS (Server)
* NPM (Package manager for Hubot scripts)
* Redis (Hubot dependency)
* Hubot + Slack Adapter (Uses websockets to communicate)
* Heroku (Deployment)

There are some drawbacks to this completely free stack which I'll talk about at the end of the Post, but in the meantime I'll cover the How from a high level, and get gradually more detailed all the way to shipping the code live.

How? (Overview)
--------------
So first things first you need a decent amount of prerequisites to actually set all this stuff up. I did this from a windows perspective, but it's arguably easier to do it on OSX, or Linux.
* Node JS
* NPM
* Redis
* Python 2.7.x
* Git  
* Heroku Toolbelt
For Node JS it should be a simple install, one for Chocolately undoubtably exists, but I already had it.

NPM is also pretty simple to install with the instructions on their site. If you're looking to make your own custom slackbot scripts you'll want to register for an NPM account online so that you can do an npm install of the scripts to your bot. Otherwise, if you're not interested in making your own script, there are already many to chose from so it's not required.

Redis does not like to work on Windows...apparently there's a fork of an older version which at least allowed me to build what I needed to with Hubot. It's on Chocolatey. Do yourself a favor and just pull that down through there if you're on Windows.

Python 2.7.x...I know what you're thinking, because I was thinking the same thing. Really? Python 2.7.x...for some reason it's a crux of the open source community that just refuses to let it die. You need this I believe to install the slack adapter for Hubot properly.

Git, this is actually how Heroku integrates with your deployment. Basically Heroku will take anything you push to the head of it's master branch as "SHIP IT".  So in this case you will have a specific Heroku remote to setup. You can set up multiple remotes too in case you'd rather develop in your own git sandbox before pushing it live.

Heroku toolbelt, this is used as a means of authentication and helping to set up Git for your authentication, and deployment. It also will integrate it's own logging after you push your code so you can see deployment time issues in the console. You'll obviously need a Heroku account to do this, but go ahead and sign up, it's free!

How? (Install)
-------------
* [Node JS](https://nodejs.org/en/download/), after doing that installing npm is as easy as opening an admin comand prompt and typing in: ```npm install npm -g```
* [redis](http://www.chocolatey.org/packages/redis-64/)/[chocolatey](https://chocolatey.org/) To install redis for windows, you'll need to install chocolatey first. I'll leave the install implementation up to which you prefer.  You can install redis afterward with the command ```choco install redis-64```
* [Python](https://www.python.org/download/releases/2.7.8/) you can get from here. Important note when installing is to make sure to toggle the setting to add python to your Windows PATH variables. Otherwise your Hubot Slack Adapter install will probably still fail.
* [Git](https://git-scm.com/download/win) can be installed in many ways and in fact if you don't have it at all, there is an integrated version as part of the Heroku toolbelt if you're not comfortable installing it. You'll also want git in your PATH variables, but if you want to install it for more than just Heroku, you should install this.
* [Heroku toolbelt](https://toolbelt.heroku.com/) You'll want to create a free account on Heroku if you haven't yet done it. Unlike for npm, this one is not an optional step.

How? (Building a bot)
Alright at this point environment setup is assumed! Go ahead and make a folder directory for your development. We'll make a bot directory in a later step. Go ahead and open a command prompt with admin rights and change to your working directory. 

Once there, run the following command: ```npm install -g hubot coffee-script yo generator-hubot```

This will grab start the hubot generator from npm. Go ahead and create a folder for your bot now, and make sure to dive into that directory in your command prompt.

Next you can trigger the hubot generator with the simple command of ```yo hubot```.
You can then answer a bunch of questions about your bot in the command prompt and when finished it will go and generate one for you.

Then we need to install a slack adapter to allow for Slack integration. Inside the same prompt call: ```npm install hubot-slack --save```. 

Surprisingly, that's all you need to configure your own hubot. By default a few different packages come with hubot's generated template. Those can be seen in the external-scripts.json file. You can also remove undesired integrations from appearing by removing these entries.

If you want to add your own, you can go hunting on npm for custom hubot scripts. Otherwise I'll cover how I made my own which you can skip over if you're not interested.

How? ([Optional] Make a custom script)
-------------------------------------
Let's be honest, you haven't gone through all this effort to make a bot just so you can use a couple extra not out of the box slack integrations. You want to make your own! In order to do that, you'll want to setup another development folder outside of your bot to create, register and deploy your own npm package. At this point, if you haven't signed up for an npm account on their website, you'll need to do so.

Alright, I'll go over the process I went through to make [hubot-tableflip](https://github.com/sparksterz/hubot-tableflip). The easiest way is to use the [generator-hubot-script](https://www.npmjs.com/package/generator-hubot-script). This you can install in your custom script's directory and run it and it will supply you with all the files you should need.
After that, you get to make it do things! Hop into the src folder and you'll see a [generatedName].coffee file. This is how you script the bot. It will have a couple examples of how to reply to a statement, or listen for a specific reference.
In my case I wanted it to respond with the response of an http get, so the coffee script version of that for node is actually quite simple:
```
module.exports = (robot) ->
  robot.respond /flip the table/i, (msg) ->
    msg.http("http://tableflipper.com/json")
      .get() (err, res, body) ->
        msg.send JSON.parse(body).gif
```
So basically what this does is makes a request to: http://tableflipper.com/json
If you go to that URL in the browser, this is what you'll see:
```{   "gif": "http://tableflipper.com/fatguy.gif" }```
As the call back to the get command I parse my response which is my 3rd parameter and get the gif property. I then send a message with the value of the gif property as my message. It's that simple!

Next you need to actually publish it so that it can be added to your hubot. npm makes it SUPER easy to publish your package. You can open the package.json and fill it out with some details. NPM follows semantic versioning so be sure to assign it an appropriate version number. If you've used the generator, your dependencies should be all set unless you have neglected to add some more of your own. They also have a git link if you decide to publish your code, it's not mandatory, but people always love a good open source script ;)
Once you have your package.json file looking good, open a command prompt in the root of your custom script folder. At this point you can call ```npm login``` 
After you're logged in, you can do ```npm publish``` and it's as simple as that.

Of course you probably didn't make a script just so others could use it, you probably want to use it yourself! So in this case you can change your command prompt directory back to your bot's directory and call: ```npm install SCRIPTNAME --save``` Then, be sure to add it to your external-scripts.json array. You probably already have a few in the list so it'll probably look something like this:
```[ "...","...","SCRIPTNAME"]```

SHIP IT! (Configuring the app space)
-----------------------------------
Wow, you've done it. You made your very own Slack bot. Let's get it up and running. First things first, let's configure Slack to have a Hubot integration go to your company's Slack integrations area (https://mycompany.slack.com/services/new) and add Hubot! Once added you'll get an API token and some settings to make your bot more shall we say "personable and accessible".

Slack should be all ready and set to listen to a public server communicating with that key to your company slack. Now we need to deploy it. You can deploy it internally on a node js server with this command ```HUBOT_SLACK_TOKEN=oxbox-bjidosajioj8232137-3hjkjase89132 ./bin/hubot --adapter slack``` Where the string of random characters is the slack hubot integration api key.

I didn't really want to maintain a constant running command prompt to host a bot, so I threw mine up on Heroku! First things first, get the command prompt over to the root of your bot folder and log in to heroku with the command: ```heroku login```

Next we need to create your Heroku app. We'll want to configure git inside of your root bot folder first, so let's make sure that's all good to go:
```
git init 
git add . 
git commit -m "My bot!"
```
Now you can call ```heroku create my-app-name``` and it will make you an app space and set your remote git server name to "heroku" and set up the "master" branch. This is a node js app when/if it asks. You may also  remember I mentioned that redis was a dependency of Hubot, so we need to add a redis setup to our app space. You'll want to log into the Heroku web interface https://dashboard.heroku.com/apps and find your app and add what they call an "add-on". This is where things can potentially cost money and Heroku wants to make sure you're liable to foot the bill and not them...Luckily there is an implementation called "Redis Cloud" They have a completely free option which will store 30MB of data for your app. More than enough for our Slackbot though, so no worries. They will still force you to add payment information on file though, not much of a way to get around that.

Now that you've added your redis integration, you can hop over to the settings tab  you'll see an environment for configuration variables. Go ahead and reveal those as you'll need to add a couple which the slack adapter will depend on. You'll probably see a REDISCLOUD_URL. That's fine, no need to touch that one. You will however need to add these:
```
HEROKU_URL https://my-app.herokuapp.com
HUBOT_SLACK_TOKEN oxbox-bjidosajioj8232137-3hjkjase89132
```
Where my-app is the Heroku app name, and the string of random characters is the slack hubot integration api key. That should be all the configuration you need. Now let's get the code live.

SHIP IT! (Deploy the code)
-------------------------
Alright, the part you've all been waiting for. Let's get our code up there and deployed. If you still have that command prompt with Heroku logged in at your bot's root, go ahead and call git push heroku master
Then you'll have a screen full of deployment info. Hopefully you'll get success and see the bot pop up in your Slack channel. If you don't you can always call heroku logs and it will try to give your more info as to why it failed.
Otherwise that's it, every time you push to master, heroku will redploy your app. So make any changes to your bot, be sure to commit them and push to heroku master.

It's live! But at what cost?
---------------------------
So, we've done all this work and now someone just perpetually hosts this for our amusement? Well...not exactly, because you, probably like myself have made a free Heroku account and possibly have never even touched some of these tools, I should let you know what I've found out.
Cut from Hubot's github:
Free dynos on Heroku will sleep after [30 minutes of inactivity](https://devcenter.heroku.com/articles/dyno-sleeping). That means your hubot would leave the chat room and only rejoin when it does get traffic. This is extremely inconvenient since most interaction is done through chat, and hubot has to be online and in the room to respond to messages. To get around this, you can use the [hubot-heroku-keepalive script](https://github.com/hubot-scripts/hubot-heroku-keepalive), which will keep your free dyno alive for up to 18 hours/day. If you never want Hubot to sleep, you will need to [upgrade to Heroku's hobby plan](https://www.heroku.com/pricing).
Unfortunately this means, you'll have to reboot your slackbot each workday, but it's a small price to pay for free hosting ;) Otherwise you can get a paid account or convince someone you know to host your little time waster!
