---
title: '🚀 Deploy Poll App to Rocket.chat'
date: 2020-01-29 13:31:39
tags: [rocketchat]
published: true
hideInList: false
feature: 
---
You don't have to connect to rocket.chat cloud to install apps from the market place. If you have the app's code, you can deploy the app yourself.

Here is an example to deploy `poll` app:

1, Install the `rc-apps` tool: `npm install -g @rocket.chat/apps-cli`. To confirm `rc-apps` has been successfully installed, run `rc-apps -v`
2, To install the `poll` app, go to [https://github.com/sampaiodiego/rocket.chat.app-poll](https://github.com/sampaiodiego/rocket.chat.app-poll)
3, Clone the repo: [https://github.com/sampaiodiego/rocket.chat.app-poll.git](https://github.com/sampaiodiego/rocket.chat.app-poll.git)
4, cd to the repo, and install dependencies with `npm install` and `npm i @types/node`
5, Build the `poll` package: `rc-apps package`
6, Deploy: `rc-apps deploy`. When run this command, it will prompt to ask you input your rocket.chat server url, admin's username and password, like this:
```
What is the server's url (include https)?: https://rc.example.com
What is the username?: admin
And, what is the password?: *********
deploying your app... deployed!
```

### Reference
* [https://rocket.chat/docs/developer-guides/developing-apps/getting-started/](https://rocket.chat/docs/developer-guides/developing-apps/getting-started/)
* [https://github.com/sampaiodiego/rocket.chat.app-poll](https://github.com/sampaiodiego/rocket.chat.app-poll)