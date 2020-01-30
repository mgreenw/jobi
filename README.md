# jobi

Tiny logging utility with events and streams.

```js
const logger = require('jobi');
logger.warn(`Soon we'll be jobiing our logs 🤯`);
```

## Lore

Jobi is a fictional tree from Thomas the Tank Engine that is _logged_. That's all folks.

![Jumping Jobi Wood](./assets/JumpingJobiWood.jpg)

## Features

- Pretty (default), JSON, and custom formats
- Configurable `stdout` and `stderr` streams
- Singleton by default
- Shared or local format and level
- `Error` stack serialization

## Levels

**stdout**
- `trace`
- `debug`
- `info`

**stderr**
- `warn`
- `error`
- `critical`

Each value implicitly includes all levels above itself – so, for example, when
your app is run with `NODE_DEBUG=warn node app.js`, all `warn`, `error`, and
`critical` logs will be sent to `stdout`/`stderr`. In the same example,
`logger.info()` and `logger.debug()` would effectively be no-ops.

## API

#### `logger.log(level, message, [...interpolationValues])`
- `level` (string): one of the levels defined above
- `message` (string): an optional message. If the message is a format string, it will use the `interpolationValues` as the format parameters.
- `[...interpolationValues]` (...any): optional values to use to format the `message` using `util.format()`. Otherwise, these values will be appended to the `message`.

#### `logger.<level>(message, [interpolationValues])`

The same as `logger.log`, but without the need to pass the `level` as a parameter. Prefer using these to `logger.log`.

#### `logger.format` (Log => string)

Get the current log format **OR** set the current log format for this `Jobi` instance. By default, a `Jobi` instance will use the shared log format. Set `logger.format = null` to resume using the shared log format.

`Log`:
- `timestamp` (string)
- `level` (string)
- `message` (string | undefined)


#### `Jobi.level` (string)

Get the shared log level **OR** set the shared log level.

#### `Jobi.format` (Log => string)

Get the shared log format **OR** set the shared log format.

#### `new Jobi(opts)`

##### `opts`
- `[format]` (string | (Log => string))
- `[stdout]` (stream.Writable)
- `[stderr]` (stream.Writable)


##### Example
```js
logger.log('info', 'This is an info log');

// Preferred
logger.info('This is another info log');
```

**NOTE**: `logger.log` has the same argument signature as `console.log`.

### Configuration

#### Log Level

`jobi` will read the initial log level from the `NODE_DEBUG` environment variable.

The `NODE_DEBUG` environment variable can actually contain *multiple* flags,
but the one with the **lowest** priority level will win. For example,
`NODE_DEBUG=debug,info,critical node app.js` will use `debug` as the log level,
since it automatically includes the other levels.

**NOTE**: If the log level is not set, `jobi` will not write any logs.

#### Log Format

`jobi` will read the initial log format from the `JOBI_FORMAT` environment variable.

Possible formats:
- `pretty`
- `json`

### Events

Each log level will emit an event of the same name *if the log level is high enough*.
For example, `logger.critical('foo');` will emit a `'critical'` event whose
callback argument will be of type `Log`.

This way, applications can hook in to the logging system and respond however
they want (post to Slack, send to a logging service, etc.).

### Streaming

By default, logs are written to either `process.stdout` or `process.stderr`.

Apps can optionally overwrite `logger.stdout` and `logger.stderr` with other
instances of `stream.Writable` in order to stream logs to the filesystem,
via HTTP, to a database, etc.

### Node.js Compatibility

`jobi` requires >= Node.js 8.3.

### Examples

##### Logging

```js
const logger = require('jobi');

logger.critical( 'this is a %s with some %s', 'log', 'formatting' );
```

##### Event binding

```js
const logger = require('jobi');

logger.on( 'critical', msg => slack.notify( msg ) );
logger.on( 'error', (msg, log) => {
  slack.notify(msg);
  console.error(log.stack);
})
```

##### Streaming

```js
const logger = require('jobi');
const fs     = require('fs');

const file = fs.createWriteStream('./log.txt');
logger.stdout = file;
logger.stderr = file;

logger.info('blah blah blah');
```

##### Custom Jobi Instance

```js
const { Jobi } = require('jobi')
const format = log => '>> ' + log.message || 'No message';

const logger = new Jobi({ format });
logger.info('Hello world'); // ">> Hello world"
```
