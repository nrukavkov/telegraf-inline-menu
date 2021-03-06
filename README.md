# Telegraf Inline Menu

[![NPM Version](https://img.shields.io/npm/v/telegraf-inline-menu.svg)](https://www.npmjs.com/package/telegraf-inline-menu)
[![node](https://img.shields.io/node/v/telegraf-inline-menu.svg)](https://www.npmjs.com/package/telegraf-inline-menu)
[![Build Status](https://travis-ci.org/EdJoPaTo/telegraf-inline-menu.svg?branch=master)](https://travis-ci.org/EdJoPaTo/telegraf-inline-menu)
[![Dependency Status](https://david-dm.org/EdJoPaTo/telegraf-inline-menu/status.svg)](https://david-dm.org/EdJoPaTo/telegraf-inline-menu)
[![Peer Dependency Status](https://david-dm.org/EdJoPaTo/telegraf-inline-menu/peer-status.svg)](https://david-dm.org/EdJoPaTo/telegraf-inline-menu?type=peer)
[![Dev Dependency Status](https://david-dm.org/EdJoPaTo/telegraf-inline-menu/dev-status.svg)](https://david-dm.org/EdJoPaTo/telegraf-inline-menu?type=dev)

This menu library is made to easily create an inline menu for your Telegram bot.

![Example shown as a gif](media/example-food.gif)

# Installation

```
$ npm install telegraf-inline-menu
```

or using `yarn`:

```
$ yarn add telegraf-inline-menu
```

Consider using TypeScript with this library as it helps with finding some common mistakes faster.

A good starting point for TypeScript and Telegraf bots might be this repo: [EdJoPaTo/telegram-typescript-bot-template](https://github.com/EdJoPaTo/telegram-typescript-bot-template)

# Examples

## Basic Example

```ts
const {Telegraf} = require('telegraf')
const {MenuTemplate, MenuMiddleware} = require('telegraf-inline-menu')
// or
import {Telegraf} from 'telegraf'
import {MenuTemplate, MenuMiddleware} from 'telegraf-inline-menu'

const menuTemplate = new MenuTemplate<MyContext>(ctx => `Hey ${ctx.from.first_name}!`)

menuTemplate.interact('I am excited!', 'a', {
  do: async ctx => ctx.reply('As am I!')
})

const bot = new Telegraf(process.env.BOT_TOKEN)

const menuMiddleware = new MenuMiddleware('/', menuTemplate)
bot.command('start', ctx => menuMiddleware.replyToContext(ctx))
bot.use(menuMiddleware)

bot.launch()
```

## More interesting one

![Example shown as a gif](media/example-food.gif)

Look at the code here: [TypeScript](examples/main-typescript.ts) / [JavaScript (consider using TypeScript)](examples/main-javascript.js)

# Migrate from version 4 to version 5

If your project still uses version 4 of this library see [v4 documentation](https://github.com/EdJoPaTo/telegraf-inline-menu/blob/v4.0.1/README.md) and consider refactoring to version 5.

List of things to migrate:

- `TelegrafInlineMenu` was splitted into multiple classes.
  When you used `new TelegrafInlineMenu(text)` you will use `new MenuTemplate(body)` now.
- Applying the menu to the bot via `bot.use` changed. This can now be done with the `MenuMiddleware`. Check the [Basic Example](#Basic-Example)
- `button` and `simpleButton` are combined and renamed into `interact`. See [How can I run a simple method when pressing a button?](#how-can-i-run-a-simple-method-when-pressing-a-button)
- `selectSubmenu` was renamed to `chooseIntoSubmenu`
- `select` was splitted into `choose` and `select`. See [Whats the difference between choose and select?](#Whats-the-difference-between-choose-and-select)
- `question` is moved into a seperate library. see [Didnt this menu had a question function?](#Didnt-this-menu-had-a-question-function)
- The menu does not automatically add back and main menu buttons anymore.
  Use `menuTemplate.manualRow(createBackMainMenuButtons())` for that at each menu which should include these buttons.
- `setCommand` and `replyMenuMiddleware` were replaced by multiple different functions. See [Can I send the menu manually?](#Can-I-send-the-menu-manually)

# How does it work

Telegrams inline keyboards have buttons.
These buttons have a text and callback data.

When a button is hit the callback data is sent to the bot.
You know this from Telegraf from `bot.action`.

This library both creates the buttons and listens for callback data events.
When a button is pressed and its callback data is occuring the function relevant to the button is executed.

In order to handle tree like menu structures with submenus the buttons itself use a tree like structure to differentiate between the buttons.
Imagine it as the file structure on a PC.

The main menu uses `/` as callback data.
A button in the main menu will use `/my-button` as callback data.
When you create a submenu it will end with a `/` again: `/my-submenu/`.

This way the library knows what do to when an action occurs:
If the callback data ends with a `/` it will show the corresponding menu.
If it does not end with a `/` it is an interaction to be executed.

You can use a Telegraf middleware in order to see which callback data is used when you hit a button:

```ts
bot.use((ctx, next) => {
	if (ctx.callbackQuery) {
		console.log('callback data just happened', ctx.callbackQuery.data)
	}

	return next()
})

bot.use(menuMiddleware)
```

You can also take a look on all the regular expressions the menu middleware is using to notice a button click with `console.log(menuMiddleware.tree())`.
Dont be scared by the output and try to find where you can find the structure in the sourcecode.
When you hit a button the specific callback data will be matched by one of the regular expressions.
Also try to create a new button and find it within the tree.

If you want to manually send your submenu `/my-submenu/` you have to supply the same path that is used when you press the button in the menu.

## Improve the docs

If you have any questions on how the library works head out to the issues and ask ahead.
You can also join the [Telegraf community chat](https://t.me/TelegrafJSChat) in order to talk about the questions on your mind.

When you think there is something to improve on this explanation, feel free to open a Pull Request!
I am already stuck in my bubble on how this is working.
You are the expert on getting the knowledge about this library.
Lets improve things together!

# FAQ

## Can I use HTML / MarkdownV2 in the message body?

Maybe this is also useful: [npm package telegram-format](https://github.com/EdJoPaTo/telegram-format)

```ts
const menuTemplate = new MenuTemplate<MyContext>(ctx => {
	const text = '_Hey_ *there*!'
	return {text, parse_mode: 'Markdown'}
})
```

## How can I run a simple method when pressing a button?

```ts
menuTemplate.interact('Text', 'unique', {
	do: async ctx => ctx.answerCbQuery('yaay')
})
```

## How does the menu update after running my interaction?

Return the relative path to the menu you wanna show.
This is '.' most of the times as you want to return to the current menu.

```ts
menuTemplate.interact('Text', 'unique', {
	do: async ctx => {
		await ctx.answerCbQuery('yaay')
		return '.'
	}
})
```

## How can I show an url button?
```ts
menuTemplate.url('Text', 'https://edjopato.de')
```

## How can I display two buttons in the same row?

Use `joinLastRow` in the second button

```ts
menuTemplate.interact('Text', 'unique', {
	do: async ctx => ctx.answerCbQuery('yaay')
})

menuTemplate.interact('Text', 'unique', {
	joinLastRow: true,
	do: async ctx => ctx.answerCbQuery('yaay')
})
```

## How can I toggle a value easily?

```ts
menuTemplate.toggle('Text', 'unique', {
	isSet: ctx => ctx.session.isFunny,
	set: (ctx, newState) => {
		ctx.session.isFunny = newState
	}
})
```

## How can I select one of many values?

```ts
menuTemplate.select('unique', ['human', 'bird'], {
	isSet: (ctx, key) => ctx.session.choice === key,
	set: (ctx, key) => {
		ctx.session.choice = key
	}
})
```

## How can I toggle many values?

```ts
menuTemplate.select('unique', ['has arms', 'has legs', 'has eyes', 'has wings'], {
	showFalseEmoji: true,
	isSet: (ctx, key) => Boolean(ctx.session.bodyparts[key]),
	set: (ctx, key, newState) => {
		ctx.session.bodyparts[key] = newState
	}
})
```

## How can I interact with many values based on the pressed button?

```ts
menuTemplate.choose('unique', ['walk', 'swim'], {
	do: async (ctx, key) => {
		await ctx.answerCbQuery(`Lets ${key}`)
		// You can also go back to the parent menu afterwards for some 'quick' interactions in submenus
		return '..'
	}
})
```

## Whats the difference between choose and select?

If you want to do something based on the choice use `menuTemplate.choose(…)`.
If you want to change the state of something, select one out of many options for example, use `menuTemplate.select(…)`.

`menuTemplate.select(…)` automatically updates the menu on pressing the button and shows what it currently selected.
`menuTemplate.choose(…)` runs the method you want to run.

## I have too much content for one message. Can I use a pagination?

`menuTemplate.pagination` is basically a glorified `choose`.
You can supply the amount of pages you have and whats your current page is and it tells you which page the user whats to see.
Splitting your content into pages is still your job to do.
This allows you for all kinds of variations on your side.

```ts
menuTemplate.pagination('unique', {
	getTotalPages: () => 42,
	getCurrentPage: context => context.session.page,
	setPage: (context, page) => {
		context.session.page = page
	}
})
```

## My choose/select has too many choices. Can I use a pagination?

If you dont use a pagination you might have noticed that not all of your choices are displayed.
Per default only the first page is shown.
You can select the amount of rows and columns via `maxRows` and `columns`.
The pagination works similar to `menuTemplate.pagination` but you do not need to supply the amount of total pages as this is calculated from your choices.

```ts
menuTemplate.choose('eat', ['cheese', 'bread', 'salad', 'tree', …], {
	columns: 1,
	maxRows: 2,
	getCurrentPage: context => context.session.page,
	setPage: (context, page) => {
		context.session.page = page
	}
})
```

## How can I use a submenu?

```ts
const submenu = new MenuTemplate<MyContext>('I am a submenu')
submenuTemplate.interact('Text', 'unique', {
	do: async ctx => ctx.answerCbQuery('You hit a button in a submenu')
})
submenuTemplate.manualRow(createBackMainMenuButtons())

menuTemplate.submenu('Text', 'unique', submenuTemplate)
```

## How can I use a submenu with many choices?

```ts
const submenuTemplate = new MenuTemplate<MyContext>(ctx => `You chose city ${ctx.match[1]}`)
submenuTemplate.interact('Text', 'unique', {
	do: async ctx => {
		console.log('Take a look at ctx.match. It contains the chosen city', ctx.match)
		return ctx.answerCbQuery('You hit a button in a submenu')
	}
})
submenuTemplate.manualRow(createBackMainMenuButtons())

menuTemplate.chooseIntoSubmenu('unique', ['Gotham', 'Mos Eisley', 'Springfield'], submenuTemplate)
```

## Can I close the menu?

You can delete the message like you would do with telegraf: `context.deleteMessage()`.
Keep in mind: You can not delete messages which are older than 48 hours.

`deleteMenuFromContext` tries to help you with that:
It tries to delete the menu.
If that does not work the keyboard is removed from the message so the user will not accidentally press something.

```ts
menuTemplate.interact('Delete the menu', 'unique', {
	do: async context => {
		await deleteMenuFromContext(context)
	}
})
```

## Can I send the menu manually?

If you want to send the root menu use `ctx => menuMiddleware.replyToContext(ctx)`

```ts
const menuMiddleware = new MenuMiddleware('/', menuTemplate)
bot.command('start', ctx => menuMiddleware.replyToContext(ctx))
```

You can also specify a path to the `replyToContext` function for the specific submenu you want to open.
See [How does it work](#how-does-it-work) to understand which path you have to supply as the last argument.

```ts
const menuMiddleware = new MenuMiddleware('/', menuTemplate)
bot.command('start', ctx => menuMiddleware.replyToContext(ctx, path))
```

You can also use sendMenu functions like `replyMenuToContext` to send a menu manually.

```ts
import {MenuTemplate, replyMenuToContext} from 'telegraf-inline-menu'
const settingsMenu = new MenuTemplate('Settings')
bot.command('settings', async ctx => replyMenuToContext(settingsMenu, ctx, '/settings/'))
```

## Can I send the menu from external events?

When sending from external events you still have to supply the context to the message or some parts of your menu might not work as expected!

See [How does it work](#how-does-it-work) to understand which path you have to supply as the last argument of `generateSendMenuToChatFunction`.

```ts
const sendMenuFunction = generateSendMenuToChatFunction(menu, '/settings/')

async function externalEventOccured() {
	await sendMenuFunction(bot.telegram, userId, context)
}
```

## Didnt this menu had a question function?

Yes. It was moved into a seperate library with version 5 as it made the source code overly complicated.

When you want to use it check [telegraf-stateless-question](https://github.com/EdJoPaTo/telegraf-stateless-question).

```ts
const myQuestion = new TelegrafStatelessQuestion<MyContext>('unique', async context => {
	const answer = context.message.text
	console.log('user responded with', answer)
	await replyMenuToContext(menuTemplate, context, '/use/path/from/where/you/came/')
})

bot.use(myQuestion.middleware())

menuTemplate.interact('Question', 'unique', {
	do: async context => {
		await myQuestion.replyWithMarkdown(context, 'Tell me the answer to the world and everything.')
	}
})
```

# Documentation

The methods should have explaining documentation by itself.
Also there should be multiple @example entries in the docs to see different ways of using the method.

If you think the jsdoc / readme can be improved just go ahead and create a Pull Request.
Lets improve things together!
