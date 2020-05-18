---
layout: post
title:  "It's all about automation!"
date:   2020-05-18 10:19:30 +0200
categories: nodejs selenium telegram
tags: [nodejs, javascript, selenium, telegram]
---
A few days ago I decided to write the script which would scrape information from specified sites and send me that via Telegram once a day. In this post I will cover how to simple achieve that.

First of all I will work with **[NodeJS](https://nodejs.org/en/)** and **[Selenium](https://www.selenium.dev/)**, so if you want to follow by this article it's strongly recommended to have them installed on your machine. There are many articles how to deal with Node, so I won't be write about it. However the installation of **Selenium** can be little tricky and I'm going to focus on that for a while.

## Selenium WebDriver
To get interested information from the specified site we are gonna to write a simple web crawler. For our needs we will use **Selenium WebDriver**. In order to run automated task on your browser you need to have browser-specific **WebDriver** binaries installed. I'm going to use Chrome, but you can choose whatever you want, just go to **[this site](https://www.selenium.dev/documentation/en/webdriver/driver_requirements/#quick-reference)** and download version you are interested in (also remember that you should choose the right version matching to your browser). After download let's extract the file in _C:\WebDriver\bin_ folder for example. Now we have to add this executable into environment variable **path**. We can do this in two different ways. First option is to use command prompt and type:
```
setx /m path "%path%;C:\WebDriver\bin\"
```

The second option is to do it manually. Here you are the quick info how to set it on Windows 10. Go to **System -> Advanced system settings ->** change tab to **Advanced -> Environment Variables ->** under **System variables** section scroll down and highlight the **Path** variable -> click the **Edit** button -> click the New button and paste _C:\WebDriver\bin_ path.

To check if everything is set correctly in the command prompt type:
```
chromedriver
```
and you should see something like this:
```
Starting ChromeDriver 2.25.426935 (820a95b0b81d33e42712f9198c215f703412e1a1) on port 9515
Only local connections are allowed.
```

## Let's get ready to rumble!

Okay, now the most interesting part is ahead of us. But before we will dive into the code, we have to set our goal. Let's say that we want get the random information from the wikipedia.org once a day and immediately send it via Telegram. So let's begin with create the folder and init npm config:
```
mkdir wiki-automation && cd wiki-automation
npm init
```
Create **index.js**:
```
touch index.js
```
Now install the **[selenium-webdriver](https://www.npmjs.com/package/selenium-webdriver)**:
```
npm install selenium-webdriver
```
And we are ready to make some code:
{% gist b66fb580c53553ff71e46c52c1aeba34 %}
`driver` is an instance of Chrome browser session, so we are gonna to use it whenever we want to make some action with browser.

If we run the above code, the Chrome browser will open, go to the specified address and quit. So far so good, now we can go deeper. To get link we could type some three letters in the search input and choose random link from the drop-down list. Let's do that in a few steps:

1. Create array of alphabet letters, and functions to get random string:

```javascript
const alphabet = 'abcdefghijklmnopqrstuvwxyz'.split('')

const getRandomVal = source => source[Math.floor(Math.random() * source.length)]

const randomString = length => [...Array(length)].map(_ => getRandomVal(alphabet)).join('')
```

2. Create a delay function, which will helps us to handle cases when DOM elements are not showing yet:

```javascript
const delay = ms => new Promise(resolve => setTimeout(() => resolve(), ms))
```

3. Now we want to go to **wikipedia** and type three random letters into search input:
```javascript
try {

await driver.get('https://en.wikipedia.org/')
await driver.findElement(By.id('searchInput')).sendKeys(randomString(3))

...
```

4. Target element with suggestion links and get an array of these links:
```javascript
const suggestionResults = await driver.wait(until.elementLocated(By.className('suggestions-results')));

await delay(2000)

const suggestionResultsList = await suggestionResults.findElements(By.className('mw-searchSuggest-link'))
```
As you can see I've used two different approaches to wait for DOM elements. But there is more. I encorage you to look on the official documentation if you're into it.
5. Get random link from the list:

```javascript
const link = await getRandomVal(suggestionResultsList).getAttribute('href')
}
```
We've made our tiny, smart web scraper, now we can go further :)

## Telegram Bot
So what is the Telegram afterall? Following the definition of Wikipedia:
> Telegram is a cloud-based instant messaging and voice over IP service.
If you don't use Telegram, you can choose any messenger that provides the API publicly, but in this post I will focus on the Telegram.

In the first step we have to create a Telegram Bot which allows us to get token and then make requests to Telegram API

Creating a new bot is pretty straightforward and you can do it in less than a minute. Just search for **BotFather** in the Telegram App and then type:
```
/newbot
```
After entering the name and nickname you'll get unique token. Before we go further you should add your bot on your friend list. To do that, search for your bot's name, click on it and click the Start button.

Now let's create a new group where we are gonna to post our daily task. Open the Telegram context menu, choose New Group and enter a name. To connect our bot with that group we have to add him as member. Open the group and click on the options on the right corner top:

![Add memmbers to Telegram's group](/assets/images/add_members_to_Telegrams_group.png)

Select _Add Members_, search for a name of your bot and click _Add_ button. For now, send some test message on this group and paste this link to your browser (remember to put up bot token):
```
https://api.telegram.org/bot{BOT_TOKEN_HERE}/getUpdates
```
In return you should get a **JSON** with a bunch of useful data, like so:
```
"chat":{"id":{CHAT_GROUP_ID},"title":{CHAT_GROUP_NAME},"type":"group","all_members_are_administrators":true},"date":1587291816,"new_chat_participant":{"id":{BOT_ID},"is_bot":true,"first_name":{BOT_NAME},"username":{BOT_USERNAME}
```
**CHAT_GROUP_ID** is that what we need in the next chapter.
>Sometimes can happen that you won't see JSON with that particular data at the first time. In that case, just remove bot from group members list and add it again, then post new message and refresh the page with above link.

## Let's get everything together
For now we are ready to connect our little web scraper with our Telegram Bot. To send request I will use **[axios](https://www.npmjs.com/package/axios)** http client, so let's install it first:
```
npm install --save axios
```
and add the following code to what we already have:
```javascript
axios.post('https://api.telegram.org/bot{BOT_TOKEN}/sendMessage', {
 chat_id: '-333449019',
 text: link
})
```
Now run the console in he directory of our **index.js** and type:
```
node index.js
```
And if everything is set properly we should get a message with random link! YAY! That was quite easy, isn't it? We can improve readability of our message by adding parse mode (there are two available mode: 'html' and 'markdown'). Let's do it:
```javascript
const message = `<a href='${link}'>Your Daily Wiki Link</a>`

axios.post('https://api.telegram.org/bot{BOT_TOKEN}/sendMessage', {
 chat_id: '-333449019',
 text: message,
 parse_mode: 'html'
})
```
That's it, we have it! Full repository of this example you **[can find here](https://github.com/Hilver/DailyWikiLink)** :) Now we can go to how to automate sending this task once a day.

## Task automation!
All what we want for now is to launch command `node index.js` in our CLI in particular intervals of time. There are many ways how to do that and mostly depend on what operating system you use. In this article I'll describe how to automate task in very easy way on Windows platform:

1. **Launch Task Scheduler**

The easiest way is to type Task Scheduler into the Start menu search.

2. **Create a task**

In the right part of window there is Action section, click on the Create Task button.

3. **General Settings**

Set a name of your task, select _Run with highest privileges_ and be sure that _Configure for_ option is matched with your system.

4. **Set triggers**

Change tab to _Triggers_, click the _New_ button and set configuration as you want.

5. **Specify an action**

This is probably the most confusing part, but we can easily go over it together. Click the _New_ button and in the _Program/script_ input add path to your CLI executable file. I will use Git Bash, so in my case the input is **_"C:\Program Files\Git\git-bash.exe"_**. Remember to add your path between quotation marks! Into the _Start_ in let's add path to folder where is our script (this time without quotation marks) and in _Add arguments_, copy and paste following command:
```
-c "node index.js"
```
Now save the configuration, click right mouse on your task and then Run. If everything is set correctly it should run our script and send your message with the link. Pretty nice, huh?

## Closing Words
I hope you found this article handy and inspired. And maybe you came up with an idea how to use that particular automation for your purposes! :) Here you have some links that you may still find useful:
* [Example GIT repository](https://github.com/Hilver/DailyWikiLink)
* [NodeJS main page](https://nodejs.org/en/)
* [Selenium dev](https://www.selenium.dev/)
* [Selenium WebDriver JS documentation](https://www.selenium.dev/selenium/docs/api/javascript/module/selenium-webdriver/)