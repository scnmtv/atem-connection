# Atem connection lybrary with CCU data

This library is forked from Sofie: The Modern TV News Studio Automation System (ATEM connection library)
The official readme is below.

The CCU library add another object named Cameras to Atem.state.

```javascript
cameras: {
    '1': {
      bars: false,
      cameraID: 1,
      autoIris: false,
      calltally: false,
      tint: 0,
      gaindb: 0,
      shutterValue: 3,
      whiteBalanceValue: 5600,
      iris: 10240,
      liftR: 0,
      gammaR: 0,
      gainR: 2048,
      liftG: 0,
      gammaG: 0,
      gainG: 2048,
      liftB: 0,
      gammaB: 0,
      gainB: 2048,
      liftY: 0,
      gammaY: 0,
      gainY: 2048,
      lumMix: 2048,
      hue: 0,
      contrast: 2048,
      pivot: 1024,
      saturation: 2048,
      command: 'hue+sat'
    },
    ...
}
```

# Sofie: The Modern TV News Studio Automation System (ATEM connection library)

[![Node CI](https://github.com/nrkno/sofie-atem-connection/actions/workflows/node.yaml/badge.svg)](https://github.com/nrkno/sofie-atem-connection/actions/workflows/node.yaml)
[![codecov](https://codecov.io/gh/nrkno/sofie-atem-connection/branch/master/graph/badge.svg)](https://codecov.io/gh/nrkno/sofie-atem-connection/)
[![npm](https://img.shields.io/npm/v/atem-connection)](https://www.npmjs.com/package/atem-connection)

This library is used in the [**Sofie** TV News Studio Automation System](https://github.com/nrkno/Sofie-TV-automation/) for connecting to Blackmagic Design ATEM devices.

## Device support

| Version       | Level of Support                                                  |
| ------------- | ----------------------------------------------------------------- |
| v8.0 - v8.6   | Primary focus. Target for new features                            |
| v7.2          | Previous focus, should still work well                            |
| v7.3 - v7.5.2 | Community tested, we may accept PRs but will not be testing these |

Due to the nature of the ATEM firmware and its tendency to break things, it is likely that new firmwares will require updates to the library to be fully supported.
Additionally any newly unimplemented functionality will generally only be written and tested for the latest firmware, even if it was added in an older firmware.

We have no intention of dropping support for older firmwares, but we may do at some point. If we do this will be done in a major version bump, and will only be for significantly older firmware.

It is tested on a variety of models spanning every generation, and the aim is to provide support for every model.

Note: USB control of devices is not supported by this library.

## Technology highlights

- Typescript
- Jest
- standard-version
- codecov

## Installation

For usage by library consumers installation is as easy as:

```sh
yarn add atem-connection
```

For library developers installation steps are as following:

```sh
git clone https://github.com/nrkno/tv-automation-atem-connection
yarn
yarn build
```

We welcome any contributions. Please read [our guide](.github/CONTRIBUTING.md) on how to add new commands before raising a PR.

## Usage

```javascript
const { Atem } = require('atem-connection')
const myAtem = new Atem()
myAtem.on('info', console.log)
myAtem.on('error', console.error)

myAtem.connect('192.168.168.240')

myAtem.on('connected', () => {
	myAtem.changeProgramInput(3).then(() => {
		// Fired once the atem has acknowledged the command
		// Note: the state likely hasnt updated yet, but will follow shortly
		console.log('Program input set')
	})
	console.log(myAtem.state)
})

myAtem.on('stateChanged', (state, pathToChange) => {
	console.log(state) // catch the ATEM state.
})
```

### Documentation

You can find the generated type docs [here](https://nrkno.github.io/sofie-atem-connection/).

### Events

- `error`
  This event will be fired when an error occurs in the connection. Node requires you to handle these errors, or your application will quit.

- `info`
  This event will be fired when the connection provides some logging information.

- `debug`
  This event is not useful for most users. It will emit some messages useful to figure out why some data useful mostly to library developers.

- `connected`
  This event will be fired once we have connected with the ATEM.

- `disconnected`
  Whenever the connection to the ATEM fails and does not recover within 5 seconds this is called.

- `stateChanged(state, path)`
  Whenever a packet from the ATEM is received that changes the state, this event will be fired.
  The path parameter is a path into the state that represents the change, to allow for filtering of events. eg video.mixEffects.0.programInput

- `receivedCommand(command)`
  Whenever a packet from the ATEM is received that contains a command, this event will be fired.
  This should not be relied on in most usage, as the commands can and will have breaking changes in patch releases. This event is needed for some use cases, so if this is used you should likely pin the version down to a specific patch release to ensure nothing breaks.

## Debug

Set `debugBuffers=true` config option in order to see raw packets. This is especially useful for library developers.

```javascript
const myAtem = new Atem({ debugBuffers: true })
myAtem.on('info', console.log)
myAtem.on('debug', console.log)
myAtem.on('error', console.error)
```

```sh
SEND <Buffer 10 14 53 ab 00 00 00 00 00 3a 00 00 01 00 00 00 00 00 00 00>
SEND <Buffer 80 0c 53 ab 00 00 00 00 00 03 00 00>
SEND <Buffer 80 0c 53 ab 00 00 00 00 00 03 00 00>
SEND <Buffer 80 0c 80 0f 00 01 00 00 00 41 00 00>
RECV <Buffer 00 0c 90 60 5f 76 65 72 00 02 00 10>...
```

## Test

This module run tests with jest.

```sh
$ yarn unit
```

There is a suite of generated serialization tests, using [atem-connection-test-generator](https://github.com/LibAtem/atem-connection-test-generator) as the [LibAtem](https://github.com/LibAtem) project has tests to verify its serialization against the Blackmagic Atem SDK
