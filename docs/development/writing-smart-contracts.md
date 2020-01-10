---
id: writing-smart-contracts
title: AssemblyScript
sidebar_label: AssemblyScript
---

## Contracts

What is a "contract?" It's the container for all the variables, functions and state of the blockchain portion of your application.

On NEAR, you just need to declare and export a function in TypeScript. Below is a fully functioning smart contract \(see it running [here](https://studio.nearprotocol.com/?f=67o8yzfco)\):

```typescript
export function hello(): string {
  return "Hello, World!";
}
```

You can also access the context in which the contract is being executed by using the `contractContext` object. This gives you access to variables like the `sender` or the contract's name via `contractName`.

See more about `context` in [autogenerated docs](runtime-ts/classes/context.md).

## View and Change functions

There are two types of functions that can interact with the blockchain -- "view" functions and "change" functions.
The difference, however, does not exist on the contract level. Rather, developers, if they wish to use view functions,
can mark certain functions to be "view functions" in the frontend or calling them through `near view` in [NEAR Shell](https://github.com/nearprotocol/near-shell).

1. **View** functions do not actually change the state of the blockchain. Imagine a function which, for instance, just checks the balance of a particular account. Because no state is changed, these functions are lightweight and generally safe.  \
  \
A view function will fail if it tries to change the state or calls the following binding methods that are exposed from nearcore:
   * `signer_account_id`
   * `signer_account_pk`
   * `predecessor_account_id`
   * `attached_deposit`
   * `prepaid_gas`
   * `used_gas`
   * `promise_create`
   * `promise_then`
   * `promise_and`
   * `promise_batch_create`
   * `promise_batch_then`
   * `promise_batch_action_create_account`
   * `promise_batch_action_deploy_account`
   * `promise_batch_action_function_call`
   * `promise_batch_action_transfer`
   * `promise_batch_action_stake`
   * `promise_batch_action_add_key_with_full_access`
   * `promise_batch_action_add_key_with_function_call`
   * `promise_batch_action_delete_key`
   * `promise_batch_action_delete_account`
   * `promise_results_count`
   * `promise_result`
   * `promise_return`

2. **Change** functions modify state, for example by updating the balance someone holds in their account. You need to be careful with these functions so they typically require explicit user authorization and are treated somewhat differently.

## Private and Public Functions

Methods with names prepended by an underscore `_` are not callable from outside the contract. All others are available publicly in the client.

```typescript
function _myPrivateFunction(someInput: string): void {
  // Code here!
}

export function myPublicFunction(someInput: string): void {
  // Code here!
}
```

## State

Like with web servers, function calls are stateless. Any state that you want to save to the blockchain needs to be explicitly saved by interacting with the `storage` object.

This object provides an interface to the blockchain storage. It is a standard key-value store where keys are strings and the values can be multiple types including `string`, `bytes`, `u64`. Anything else needs to be first converted into these types.

See [the storage docs](/docs/runtime-ts/classes/storage) and [collections](/docs/runtime-ts/modules/collections) for the full reference.

You can also save data using persistent collections discussed below.

### Collections

There are currently four types of collections. These all write and read from storage abstracting away a lot of what you might want to add to the storage object.

* PersistentVector
  * Acts like an array
  * You can create a vector like this:
    * `let vec = collections.vector<string>("v");`
* PersistentMap
  * Acts like maps you'd find in most languages
  * Yau can create a map like this:
    * `let m = collections.map<string, string>("m");`
    * You can use the map with `m.set(key, value)` and `m.get(key)`
* PersistentDeque
  * Implementation of a deque \(bidirectional queue\).
  * You can create a deque like this:
    * `let d = collections.deque<string>("d");`
* PersistentTopN
  * Used for creating ranked lists
  * You can create a TopN collection like this:
    * `let t = collections.topN<string>("t");`

The letter passed in as an argument \(e.g. `"v"` in the case of the vector\) is the key that gets assigned as a prefix to distinguish the collections from each other \(precisely because they're persistent\).

**NOTE**: if you're coming from JavaScript, you might not be familiar with the type declaration in the two brackets `<>`. In TypeScript, need to declare the types that any collection is going to take.

## Arrays

Arrays are similar to Arrays in other languages. One key difference is in how they are initialized, and what that means for your app. Check out more details in the [AssemblyScript docs](https://docs.assemblyscript.org/standard-library/array).

\(from the AssemblyScript documentation\):

```typescript
// The Array constructor implicitly sets `.length = 10`, leading to an array of
// ten times `null` not matching the value type `string`. So, this will error:
var arr = new Array<string>(10);
// arr.length == 10 -> ERROR

// To account for this, the .create method has been introduced that initializes
// the backing capacity normally but leaves `.length = 0`. So, this will work:
var arr = Array.create<string>(10);
// arr.length == 0 -> OK

// When pushing to the latter array or subsequently inserting elements into it,
// .length will automatically grow just like one would expect, with the backing
// buffer already properly sized (no resize will occur). So, this is fine:
for (let i = 0; i < 10; ++i) arr[i] = "notnull";
// arr.length == 10 -> OK
```

There is not currently syntactic sugar for array iterators like `map`.

## Iteration

Iteration follows the standard TypeScript format:

```typescript
// set i to a type u64
for (let i: u64 = startIndex; i < someValue; i++) {
  // do stuff
}
```

## Classes

Classes are normal TypeScript classes and more information can be found in the [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/classes.html). We don't have structs, we have TypeScript classes instead.

You will generally want to define your classes in a different file and then import them:

```typescript
// 1. define the class in the `assembly/model.ts` file
export class PostedMessage {
  sender: string;
  text: string;
}
```

```typescript
// 2. Import the class to your `assembly/main.ts` file
import { PostedMessage } from "./model";
```

There are no structs.

## Function Declarations and Return Values

Function declarations follow standard TypeScript conventions, including the parameters they take, optional arguments and return values. See the [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/functions.html) for more info.

## Events

Sometimes you want your front end to automatically update if something changes on the back end. For example, if you have a messaging app that should update your screen when your friend sends you a message. Currently, you will need to poll the chain to make this happen.

In the future, we may expose event emitters and listeners as syntactic sugar. If this is important to you, reach out [on Discord](http://near.chat).

## Math

Mathematical operations in TypeScript are done in the same way as JavaScript. See more in [this article](https://www.tutorialspoint.com/typescript/typescript_arithmetic_operators_examples.htm).

## Time

Time is one of the most difficult concepts in blockchains. In a single-server-based environment like web developers are used to, the server's \(or database's\) clock is ok to rely on for creating timestamps.

Because the blockchain's state is determined by a consensus among many nodes and must be deterministic and adversary-resistant, there is no way to settle on a "correct" clock time while code is being run.

You can pull timestamps from the client side \(eg the JavaScript running on the user's computer\) but that should come with all the usual warning about not trusting anything a client sends.

For less exact measures of time, the block index value is sufficient. This will look something like:

```typescript
// Note: Not implemented yet...
contractContext.blockIndex();
```

Some solutions to the time issue include using "trusted oracles" but that's outside the scope of this doc.