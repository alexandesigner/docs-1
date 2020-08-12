---
title: Signing transactions
description: Learn how to sign transactions using Blockstack Connect.
experience: advanced
duration: 30 minutes
---

## Introduction

With Connect, you can interact with the Stacks 2.0 blockchain. You can allow your users to send transactions
and interact with smart contracts.

## Transaction signing is still in progress

To use these features, make sure you install the `testnet` tag the `@blockstack/connect`
NPM package. You can do this by running `npm install --save @blockstack/connect@testnet`, or by specifying
`testnet` as the version in your package.json file.

The Stacks 2.0 blockchain is still in testnet, and our web app integration is also still in beta. In order to use
transaction signing in your application, you need to use the configuration `authOrigin` with `@blockstack/connect`.

```tsx
const options = {
  // your other options
  authOrigin: 'https://deploy-preview-301--stacks-authenticator.netlify.app',
};
```

## How it works

For your app's users to be able to execute a smart contract function, they need to sign and broadcast a transaction.
It's important that users remain in control of the private keys that sign these transactions. Connect provides an
easy-to-use workflow that allows your users to securely sign transactions.

Connect allows you to open the authenticator with parameters indicating the details of the transaction - like the smart
contract address, function name, and specific arguments. Your users get the chance to see these details, and then sign
and broadcast the transaction in a single click. Their transaction will be securely signed and broadcasted onto the
Stacks blockchain. After this is done, a callback is fired to allow you to update your app.

## Calling Clarity Contract Functions

Once you have a Clarity smart contract built and deployed, you'll naturally want to allow your app's users to interact
with it.

To initiate a contract call transaction, use the `openContractCall` function.

```tsx
import { openContractCall } from '@blockstack/connect';

// While in beta, you must provide this option:
const authOrigin = 'https://deploy-preview-301--stacks-authenticator.netlify.app';

// Here's an example of options:
const myStatus = 'hey there';
const options = {
  contractAddress: 'ST22T6ZS7HVWEMZHHFK77H4GTNDTWNPQAX8WZAKHJ',
  contractName: 'status',
  functionName: 'write-status!',
  functionArgs: [
    {
      type: 'buff',
      value: myStatus,
    },
  ],
  authOrigin,
  appDetails: {
    name: 'SuperApp',
    icon: 'https://example.com/icon.png',
  },
  finished: data => {
    console.log('TX ID:', data.txId);
    console.log('Raw TX:', data.txRaw);
  },
};

await openContractCall(opts);
```

When calling `openContractCall`, you need to specify a few details. Here is the exact interface that describes what
options you have:

```tsx
interface ContractCallOptions {
  contractAddress: string;
  functionName: string;
  contractName: string;
  functionArgs?: any[];
  authOrigin?: string;
  appDetails: {
    name: string;
    icon: string;
  };
  finished: (data: FinishedTxData) => void;
}
```

| parameter       | type     | optional | description                                                                                                                                                                   |
| --------------- | -------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| contractAddress | string   | false    | The Stacks address that published this contract                                                                                                                               |
| contractName    | string   | false    | The name that was used when publishing this contract                                                                                                                          |
| functionName    | string   | false    | The name of the function you're calling. This needs to be a [public function](/references/clarity-language#define-public).                                                    |
| functionArgs    | array    | false    | The arguments you're calling the function with. You'll need to provide the Clarity type with each argument. See the below section for details. Defaults to `[]`.              |
| appDetails      | object   | false    | A dictionary that includes `name` and `icon`                                                                                                                                  |
| finished        | function | false    | A callback that is fired when the transaction is signed and broadcasted. Your callback will receive an object back with a `txId` and a `txRaw`, both of which are strings.    |
| authOrigin      | string   | true     | The location of the authenticator. This is only necessary when developing the authenticator locally, or when using beta features. Defaults to `"https://app.blockstack.org"`. |

### Passing Clarity types with function arguments

To be able to serialize your transaction properly, you need to provide the appropriate Clarity type with each argument.
These types are named the same as they are in Clarity. The `value` that you pass must be a string. The types you can pass are:

- `uint` - i.e. `"240"`
- `int` - i.e. `"12"`
- `bool` - can be "true", "false", "0" or "1"
- `buff` - i.e. `"asdf"`
- `principal` - This can be a contract principal, or a standard principal.
  [Read more about principals](/core/smart/principals). Examples: `"ST22T6ZS7HVWEMZHHFK77H4GTNDTWNPQAX8WZAKHJ"`
  or `"ST22T6ZS7HVWEMZHHFK77H4GTNDTWNPQAX8WZAKHJ.my-contract"`.

Using these types, each argument is an object with the keys `type` and `value`. For example:

```tsx
const functionArguments = [
  {
    type: 'buff',
    value: 'hello, world',
  },
  {
    type: 'uint',
    value: '1',
  },
];
```

If you're using Typescript, these Clarity types can be imported as `ContractCallArgumentType` from `@blockstack/connect`.

## Stacks (STX) Token Transfers

STX token transfers can be initiated with the `openSTXTransfer` function.

```tsx
import { openSTXTransfer } from '@blockstack/connect';

// While in beta, you must provide this option:
const authOrigin = 'https://deploy-preview-301--stacks-authenticator.netlify.app';

openSTXTransfer({
  recipient: 'ST2EB9WEQNR9P0K28D2DC352TM75YG3K0GT7V13CV',
  amount: '100',
  memo: 'Testing STX Transfers!',
  authOrigin,
  appDetails: {
    name: 'SuperApp',
    icon: 'https://example.com/icon.png',
  },
  finished: data => {
    console.log(data.txId);
  },
});
```

When calling `openSTXTransfer`, you need to specify a few details. Here are the options you have:

```tsx
interface STXTransferOptions {
  recipient: string;
  amount: string;
  memo?: string;
  authOrigin?: string;
  appDetails: {
    name: string;
    icon: string;
  };
  finished: (data: FinishedTxData) => void;
}
```

| parameter  | type     | optional | description                                                                                                                                                                   |
| ---------- | -------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| recipient  | string   | false    | The STX Address for the recipient of this STX transfer                                                                                                                        |
| amount     | string   | false    | The amount of microstacks (µSTX) to be transferred. This argument is a string to prevent floating point errors. There are 1,000,000 µSTX per STX.                             |
| memo       | string   | true     | An optional memo to include in the transaction.                                                                                                                               |
| appDetails | object   | false    | A dictionary that includes `name` and `icon`                                                                                                                                  |
| finished   | function | false    | A callback that is fired when the transaction is signed and broadcasted. Your callback will receive an object back with a `txId` and a `txRaw`, both of which are strings.    |
| authOrigin | string   | true     | The location of the authenticator. This is only necessary when developing the authenticator locally, or when using beta features. Defaults to `"https://app.blockstack.org"`. |

## Deploying Clarity Contracts

To allow your app's users to deploy arbitrary Clarity contracts, use the `openContractDeploy` method.

```tsx
import { openContractDeploy } from '@blockstack/connect';

const codeBody = '(begin (print "hello, world"))';
// While in beta, you must provide this option:
const authOrigin = 'https://deploy-preview-301--stacks-authenticator.netlify.app';

openContractDeploy({
  contractName: 'my-contract-name',
  codeBody,
  authOrigin,
  appDetails: {
    name: 'SuperApp',
    icon: 'https://example.com/icon.png',
  },
  finished: data => {
    console.log(data.txId);
  },
});
```

Here is the interface for the options you can provide to `openContractDeploy`:

```tsx
interface ContractDeployOptions {
  codeBody: string;
  contractName: string;
  authOrigin?: string;
  appDetails: {
    name: string;
    icon: string;
  };
  finished: (data: FinishedTxData) => void;
}
```

| parameter    | type     | optional | description                                                                                                                                                                   |
| ------------ | -------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| codeBody     | string   | false    | The Clarity source code for this contract                                                                                                                                     |
| contractName | string   | false    | The name for this contract                                                                                                                                                    |
| appDetails   | object   | false    | A dictionary that includes `name` and `icon`                                                                                                                                  |
| finished     | function | false    | A callback that is fired when the transaction is signed and broadcasted. Your callback will receive an object back with a `txId` and a `txRaw`, both of which are strings.    |
| authOrigin   | string   | true     | The location of the authenticator. This is only necessary when developing the authenticator locally, or when using beta features. Defaults to `"https://app.blockstack.org"`. |

## Usage in React Apps

Make sure you follow the [setup instructions](/authentication/connect#in-react-apps) first. When you're using
`useConnect`, you don't have to specify `appDetails` - we'll pick that up from your existing configuration.

Each transaction signing method is exposed through the `useConnect` hook, but they're prefixed with `do` instead of
`open`, to remain consistent with our React action naming standards.

```tsx
import { useConnect } from '@blockstack/connect';

const MyComponent = () => {
  const { doContractCall } = useConnect();

  const onClick = async () => {
    const opts = {
      /** See examples above */
    };
    await doContractCall(opts);
  };

  return <span onClick={onClick}>Call my contract</span>;
};
```