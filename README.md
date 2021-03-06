# Portage

Fast pub/sub for JS

`npm install portage`

`bower install portage`

Works with AMD, CommonJS, global (as `portage`) and ES6.

## Intro

Portage is utilizing a [tree structure](https://github.com/EyalAr/FuzzyTree) to
[quickly](#benchmark) match publications with subscriptions; including support
for subscriptions with wild cards (see below).

Publications and subscriptions are segmented by channels. Channels are used to
publish messages on certain topics, and to subscribe to messages on certain
topics (or a pattern of topics).

A channel internally maintains a tree structure of subscriptions.

Topics are organized in a hierarchical manner. For example, `chat.new-message`
is a sub-topic of `chat`. This mostly affects the way the library efficiently
filters subscription handlers when a message is published, but also relates
to the usage of wild cards in subscriptions (see below).

Hubs are simply an aggregation of channels, which can easily be accessed (and
created on the fly) with a key.

## Hub API

```
var Hub = require('portage').Hub;
var myHub = new Hub();
```

### Get / create a channel

`var myChannel = myHub.channel(name)`

- `name {String}`: The channel name.

If channel `name` is requested for the first time, it is first created and then
returned. Otherwise the existing channel with that name is returned.

### Default global hub

`var portage = require('portage')`

`portage` is a `Hub` object which is created when the library is loaded.
`require('portage')` always returns the same default hub.

## Channel API

`var Channel = require('portage').Channel`

### Create a channel

`var myChannel = new Channel()`

### Publish data on a topic

`myChannel.publish(topic, data)`

- `topic {String}`: The topic of the publication. See topic structure below.
- `data {*}`: Data of the publication.

### Subscribe to a topic

`var s = myChannel.subscribe(pattern, callback)`

- `pattern {String}`: The pattern of topics of which publications to subscribe.
   See topic structure below.
- `callback {Function}`: Callback function to invoke when a message is published
   whose topic matches the pattern. The function is called with two arguments:
      0. `data`: The published data.
      0. `meta`: An object which contains the following properties:
         - `topic {String}`: The topic of the publication.
         - `called {Integer}`: The number of times this subscription has been
           invoked, **not** including the current time.
         - `limit {Integer}`: Invocation limit of this subscription, or null if
           no limit.
         - `last {Boolean}`: Flag if this invocation is the last.

**Return value** is an object with the following methods:

- `s.unsubscribe()`: removes this subscription
- `s.once()`: limits invocation of this subscription to one additional time,
  after which it is automatically removed.
- `s.limit(n)`: limits invocation of this subscription to `n` additional
  times, after which it is automatically removed.

**Note:** If `s.limit(3)` is called after the subscription was already called 5
times, it will be called up to 3 more times; up to a total of 8 times.

### Topics structure

Topics are strings which are divided into sections by the `'.'` separator.
For example, the topic `'chat.new-message.server'` has 3 sections: `'chat'`,
`'new-message'` and `'server'`.

On subscriptions, a topic pattern may be specified. A pattern can include wild
cards of 2 types:

- `'*'`: Non greedy wild card. Will match any one section. For example,
  `'chat.*.server'` will match both `'chat.new-message.server'` and
  `'chat.remove-message.server'`, **but not** `'chat.new-message.local'` and
  not `'chat.new.message.local'`.

- `'#'`: Greedy wild card. Will match one or more sections. For example,
  `'chat.#'` will match `'chat.new-message.server'`,
  `'chat.remove-message.server'` **and** `'chat.new-message.local'`. Basically
  it will match any topic that begins with `'chat.'`. `'chat.#.local'` will
  match any topic that begins with `'chat'` and ends with `'local'`. `'#.local'`
  will match any topic that ends with `'local'`.

## Benchmark

Benchmark against [Postal](https://github.com/postaljs/postal.js) is available
at the `benchmark/` folder.

Results have shown Portage to be more than **90% faster** than Postal.

To run:

```
npm install
npm run benchmark
```
