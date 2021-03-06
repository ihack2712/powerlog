# Powerlog

> A logger for Deno that sends your logs wherever you want them.

## Getting Started

Firstly,

You need to import the module into your project.

```ts
import { Powerlog } from "https://deno.land/x/powerlog/mod.ts";
```

Secondly,

You should import any built-in transports you want to use.

```ts
import {
	WriterTransport,         // For writing to an async writer (an object that has .write).
	StdTransport,            // For writing to stdout or stderr.
	FileTransport,           // For appending to a file.
	TcpTransport,            // For sending logs over a tcp connection or unix sock.
	DiscordWebhookTransport, // For sending discord webhook messages.
} from "https://deno.land/x/powerlog/mod.ts";
```

Thirdly,

You need a log levels enumerable.

```ts
enum MyLogLevels {
	debug,
	info,
	success,
	alert,
	warn,
	critical,
}
```

Fourthly,

You need to create a new powerlog instance.

```ts
const myLogger = Powerlog.get({
	levels: MyLogLevels,
	name: "My Logger"
});
```

Fifthly,

You can now also create the transports you wish to use.

```ts
const myStdoutTransport = new StdTransport({
	levels: MyLogLevels
});
```

```ts
const myFileTransport = new FileTransport({
	levels: MyLogLevels,
	filename: "my.log"
});
```

Sixthly,

Add the transports to your logger.

```ts
await myLogger.use(
	myStdoutTransport,
	myFileTransport
);
```

Seventhly,

You can now use any of the properties in `MyLogLevels` as loggging methods.

```ts
myLogger.debug("Hello World");
```

Lastly but not least,

When you're done with the logger always remember to dispose of its transports.

```ts
await myLogger.dispose();
```

## Formatting

When creating transports you will have the ability to add formatting to your messages, however, not all transports have that ability. The `WriterTransport` object does not have formatting capabilities. It does however have methods `<WriterTransport>.dataToByteArray` and `<WriterTransport>.dataToString`. You can extend the `WriterTransport` and modify those.

### Default Formatting

The default formatting applies to `FormatTransportBase`, `FileTransport`, `StdTransport` and `TcpTransport`.

You can on either of these's constructors add a formatting property which is a function that returns either a `string` or a `Uint8Array`, it can be both synchronous and asynchronous.

**Example**:

```ts
const transport = new StdTransport({
	formatter: data =>
		`[${data.timestamp.toJSON()}] (${data.name}) ${MyLogLevels[data.level]} ${sprintf(data.message, ...data.arguments)}`
});
```

You can also change the formatter later on with the use `<FormatTransportBase>.format(formatter)`.

**Example**:

```ts
transport.format(
	data =>
		`${MyLogLevels[data.level]} ${sprintf(data.message, ...data.arguments)}`
);
```

You can also apply a default format on the powerlog instance the same way as above. When adding transports with no formatters to a powerlog instance with a default formatter the powerlog instance will try to apply the default formatter onto the transports on initialization.

**Example**:

```ts
const myLogger = new Powerlog({
	levels: MyLogLevels,
	name: "My Logger",
	formatter: data =>
		`[${data.timestamp.toJSON()}] (${data.name}) ${MyLogLevels[data.level]} ${sprintf(data.message, ...data.arguments)}`
});
```

You can then also later change the default formatter by using `<Powerlog>.format(formatter)`.

**Note** that this will not replace the previous default formatters applied to the already existing transports. It will only be applied to new transports.

**Example**:

```ts
myLogger.format(
	data =>
		`${MyLogLevels[data.level]} ${sprintf(data.message, ...data.arguments)}`
);
```

### Discord Formatting

The discord transport uses its own formatter specification. The formatter function is expected to return a discord message object.

```ts
import type { IDiscordMessage } from "https://deno.land/x/powerlog/mod.ts";
import { DiscordWebhookTransport } from "https://deno.land/x/powerlog/mod.ts";
```

**Example**:

```ts
const myDiscordWebhookTransport = new DiscordWebhookTransport({
	levels: MyLogLevels,
	url: "webhook url",
	formatter: data => ({
		content: null,
		embeds: [
			{
				title: MyLogLevels[data.level],
				description: sprintf(data.message, ...data.arguments),
				timestamp: data.timestamp.toJSON()
			}
		]
	})
});
```

Again, this can later be changed using `<DiscordWebhookTransport>.format(formatter)`.
