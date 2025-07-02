# bloxd-code-loader

This script allows you to load and execute code dynamically from a chest in Bloxd.

The current version may stop working properly in the future!
This project uses translation, so the English may not be perfectly accurate!

## Features

* Execute code stored in chests
* Supports automatic code splitting and reconstruction
* Designed for modular code management within Bloxd

## How to Use

1. Store your code in a chest using the provided system.
2. The loader reads and reconstructs the code from chest slots.
3. The code is executed automatically within the Bloxd environment.

## Important Notes

* This system is intended for use **only within Bloxd**.
* Redistribution or public sharing of the code is **prohibited**.
* For personal, in-game use only.

---

## Data Code Management

This system provides functionality to manage structured data stored in chests, known as **data codes**.

### Available Functions

* `readDataCode(codeName)`
  Reads the loaded data for the specified code name. Returns the data object or `null` if not loaded.

* `accessDataCode(codeName, path)`
  Accesses a specific nested value within a data code.
  Supports dot and bracket notation in the path (e.g., `"stats.points"` or `"items[0].name"`).

* `setDataCodeValue(codeName, path, value)`
  Updates a value within the data code and writes the changes back to the associated chest slots.
  Returns `true` on success, `false` if the data is not loaded or an error occurs.

> **Note:**
> You can also access data like this:
> `globalDataStore.hogehoge`
> The globalDataStore contains the values ​​read from the data code.
> But for safe modification and proper chest synchronization, always use `setDataCodeValue`.

---

## Callback System

The loader includes a unified callback mechanism for handling Bloxd events.

### Definitions Controlling Callback Behavior

#### `callbackDefinitions`

Defines supported events and their expected return values.

```js
const callbackDefinitions = {
  onPlayerJoin: { returns: false },
  playerCommand: { returns: [true, false] },
  onPlayerChat: { returns: [false, null, "object", "string"] }
};
```
Most callbacks that have a return value are already added, so you just add them in the format `callback: {returns: false }`.

#### `registeredCallbacks`

Stores the actual registered functions for each event.

```js
const registeredCallbacks = {
  tick: [],
  playerCommand: [],
  onPlayerJoin: [],
  playerLeave: []
};
```
If you want to add a callback, add it in the form `callback: []`

### Important Rules

* Only events defined in both `callbackDefinitions` and `registeredCallbacks` are functional.
* Return values from callbacks are validated based on the `returns` field in `callbackDefinitions`.

---

## Command Handling

The loader supports custom player commands through the `playerCommand` callback.

### Built-in Commands

* `/loadcode <codeName>`
  Reloads a code module by its name. Requires authorization.

* `/unload <codeName>`
  Unloads a loaded code module. Requires authorization.

* `/reloadauto`
  Reloads all code modules marked as automatic. Requires authorization.

* `/loadedcodes`
  Lists all known code modules with their load status.

* `/removecode <codeName>`
  Completely removes a code module and clears its data from chests. Requires authorization.

> Unauthorized users receive a warning message with their player ID.

---

## Using `writeCodeToChest`

This function saves code or data to a chest in a safe, loader-compatible format.

### Syntax

```js
writeCodeToChest( 
  chestPos, // recommended: [thisPos[0], thisPos[1] + 1, thisPos[2]],
  `codeString`, // code to save (see note below) 
  {
    Creator: "YourName", 
    codeName: "CodeName",
    SaveDestination: [[thisPos[0], thisPos[1] + 1, thisPos[2]], // this structure do not change 
    codeType: "auto" // Options: "auto", "data", "manual" 
 }, 
 myId, // as is 
 false // Unicode encoding (leaving it false is strongly recommended; setting it to true will cause an error very likely to occur and the character count jumps by a factor of about 6) 
);
```

### Code String Rules

* Use a backtick string for multiline code.
* Place all callback functions inside a `callbacksToRegister` object.
* Always register callbacks at the end with:

```js
registerCallbacks("コード名", callbacksToRegister);
```

* Data can be accessed directly via `globalDataStore`,
  but for reliable updates, use `setDataCodeValue`.

### SaveDestination Rules

* Must be a **2D array** like `[[thisPos[0], thisPos[1] + 1, thisPos[2]]]`.
* Do **not** change this structure manually. Incorrect formats may break loader functionality.

### Unicode Encoding

* The last parameter controls Unicode encoding.
* **Keep it `false` unless you fully understand the risks.**
* Enabling it may cause unexpected errors in some cases.

---
### Known Defects

- Issue with onPlayerJoin callback not being executed when a player enters the lobby when there is no one in the lobby.
- 
  
*I haven't actually written any long code, so I'm sure there are many other bugs besides the ones mentioned above.*


---

# Bloxd Code Loader - Usage Examples

This is a collection of templates and sample codes for storing, loading, and executing code or data dynamically from chests within Bloxd.

---

## Template: Basic `writeCodeToChest` Usage

This is the basic template for writing code into a chest.

```js
writeCodeToChest(
  [thisPos[0], thisPos[1] + 1, thisPos[2]],
  ``,
  {
    Creator: "",
    codeName: "",
    SaveDestination: [[thisPos[0], thisPos[1] + 1, thisPos[2]]],
    codeType: "auto"
  },
  myId,
  false
);
```

Copy this and fill in the code and metadata as needed.

---

## 1. Output "hello world" in chat

```js
writeCodeToChest(
  [thisPos[0], thisPos[1] + 1, thisPos[2]],
  `api.broadcastMessage("hello world!!")`,
  {
    Creator: "kentaki",
    codeName: "test",
    SaveDestination: [[thisPos[0], thisPos[1] + 1, thisPos[2]]],
    codeType: "auto"
  },
  myId,
  false
);
```

---

## 2. Log a message when a player clicks

```js
writeCodeToChest(
  [thisPos[0], thisPos[1] + 1, thisPos[2]],
  `const callbacksToRegister = {
    onPlayerClick: (playerId, wasAltClick) => {
      api.log("hogehoge");
    }
  };
  registerCallbacks("testCallbacks", callbacksToRegister);`,
  {
    Creator: "kentaki",
    codeName: "testCallbacks",
    SaveDestination: [[thisPos[0], thisPos[1] + 1, thisPos[2]]],
    codeType: "auto"
  },
  myId,
  false
);
```

*Don't forget to call `registerCallbacks`. If it doesn't work, try running `/loadcode testCallbacks` in-game.*

---

## 3. Prevent chatting when message is "test1"

```js
writeCodeToChest(
  [thisPos[0], thisPos[1] + 1, thisPos[2]],
  `const callbacksToRegister = {
    onPlayerChat: (playerId, chatMessage, channelName) => {
      if (chatMessage === "test1") {
        return false;
      }
    }
  };
  registerCallbacks("testchat", callbacksToRegister);`,
  {
    Creator: "kentaki",
    codeName: "testchat",
    SaveDestination: [[thisPos[0], thisPos[1] + 1, thisPos[2]]],
    codeType: "auto"
  },
  myId,
  false
);
```

*Note: The only supported return value for `onPlayerChat` to block a message is `false`.*

---

## 4. Store a function in `globalDataStore` and execute it with a command

```js
writeCodeToChest(
  [thisPos[0], thisPos[1] + 1, thisPos[2]],
  `
  globalDataStore.testFunc = function(playerId) {
    api.sendMessage(playerId, "hi!");
  };

  const callbacksToRegister = {
    playerCommand: (playerId, cmd) => {
      if (cmd === "hello") {
        globalDataStore.testFunc(playerId);
        return true;
      }
    }
  };
  registerCallbacks("testfunc", callbacksToRegister);
  `,
  {
    Creator: "kentaki",
    codeName: "testfunc",
    SaveDestination: [[thisPos[0], thisPos[1] + 1, thisPos[2]]],
    codeType: "auto"
  },
  myId,
  false
);
```

*Typing `/hello` will trigger the function and send a message to the player.*

---

## 5. Define data as a "data code" stored in a chest

```js
writeCodeToChest(
  [thisPos[0], thisPos[1] + 1, thisPos[2]],
  `globalDataStore.testData = {
    messages: {
      welcomeMessage: "hello!",
      goodbyeMessage: "See you next time!",
      errorMessage: "eee",
      successMessage: "Operation completed successfully.",
      infoMessage: "This is an informational message"
    }
  };`,
  {
    Creator: "kentaki",
    codeName: "testdata",
    SaveDestination: [[thisPos[0], thisPos[1] + 1, thisPos[2]]],
    codeType: "data"
  },
  myId,
  false
);
```
Please enter the data in this format
<details> <summary>This is a template for data definition code(Click to open)</summary>
  
```js
writeCodeToChest(
  [thisPos[0], thisPos[1]+1, thisPos[2]],
  `
  globalDataStore./*Please change it to the appropriate name*/ = {
    /*Place objects here.*/
  };
  `,
  {
    Creator: "kentaki",
    codeName: "testdata",
    SaveDestination: [[thisPos[0], thisPos[1]+1, thisPos[2]]],
    codeType: "data"
  },
  myId,
  false
);
```
</details>

---

## 6. Read, access, and update the data code

```js
writeCodeToChest(
  [thisPos[0], thisPos[1] + 1, thisPos[2]],
  `
  const messages = readDataCode("testdata");
  api.log(messages); // logs the entire object

  const welcomeMessage = accessDataCode("testdata", "messages.welcomeMessage");
  api.log(welcomeMessage); // logs "hello!"

  setDataCodeValue("testdata", "messages.errorMessage", "An error has occurred");
  `,
  {
    Creator: "kentaki",
    codeName: "testcall",
    SaveDestination: [[thisPos[0], thisPos[1] + 1, thisPos[2]]],
    codeType: "auto"
  },
  myId,
  false
);
```

---
