---
title: "Implementing an opaque type in typescript"
date: "2020-02-18 18:30:00 UTC"
tags:
  - typescript
  - types
location: Adelaide St W, Toronto, ON, Canada
geo: [43.647767, -79.389963]
---


Say, you're in a situation where you have a user type, that looks a bit as
follows:

```typescript
export type User = {
  firtName: string;
  lastName: string;
  email: string;
}

function save(user: User) {
   // ...
}

const user = {
  firstName: 'Evert',
  lastName: 'Pot',
  email: 'foo@example.org',
}

save(user);
```

But, instead of accepting _any_ string for an email address, you want to
ensure that it only accepts email addresses that are valid.

You might want to structure your user type as follows:

```typescript
type Email = string;

export type User = {
  firtName: string;
  lastName: string;
  email: Email
}
```

This doesn't really do anything, we aliased the Email to be exactly like a
string, so any string is now also an `Email`.

We can however extend the email type _slighty_ to contain a property that
nobody can ever add.

```typescript
const validEmail = Symbol('valid-email');

type Email = string & {
  [validEmail]: true
}

export type User = {
  firstName: string;
  lastName: string;
  email: Email
}
```

In the above example, we're taking advantage of Javascript's [Symbol][1]. A
symbol is a type that is always unique. 

We're adding a property with this key to our Email string. A user can _only_
add this property, if they have an exact reference to the original symbol.

Given that we don't export this symbol, it's not possible anymore for a user
to construct an `Email` type manually.

Now when we compile this:

```typescript
const user = {
  firstName: 'Evert',
  lastName: 'Pot',
  email: 'foo@example.org',
}

save(user);
```

We get the following error:


```
src/post/user.ts:31:6 - error TS2345: Argument of type '{ firstName: string; lastName: string; email: string; }' is not assignable to parameter of type 'User'.
  Types of property 'email' are incompatible.
    Type 'string' is not assignable to type 'Email'.
      Type 'string' is not assignable to type '{ [validEmail]: true; }'.
```

So how do turn our strings into a valid `Email` type? With an assertion
function:

```typescript
function assertValidEmail(input: string): asserts input is Email {
  
  // Yes this is very basic, but it's here for illustration purposes.
  if (!input.includes('@')) {
    throw new Error(`The string: ${input} is not a valid email address`);
  }

}
```

Now to construct our valid user object:

```typescript
const email = 'foo@example.org';
assertValidEmail(email);
const user:User = {
  firstName: 'Evert',
  lastName: 'Pot',
  email,
}

save(user);
```

This is helpful, because it allows you to construct types, such as string
types, and enforce their contents to be validated.

This means that the information that a string is a valid email, is almost like
a tag or label on your string that ensures that whoever constructed the
original user object, already made sure that it's valid, and that the user
object can therefore never be created in an invalid state.


An interesting side-note is that even though we used a symbol as a market, we
never actually had to add it to string. The marker only exists in the
type-system as a means to ensure that nobody can easily create the `Email` type,
circumventing the validation system.

After compilation, from a javascript perspective, email is still always just a
string.

Instead of an assertion function, you can also use a type guarding function:

```typescript
function isValidEmail(input: string): input is Email {
  
  return (input.includes('@')); 


}
```

The main difference is that assertions should throw an exception, and type
guards just return true or false.


Here's the full script:

```typescript
const validEmail = Symbol('valid-email');

type Email = string & {
  [validEmail]: true
}

export type User = {
  firstName: string;
  lastName: string;
  email: Email
}

function save(user:User) {

}

function assertValidEmail(input: string): asserts input is Email {
  
  if (!input.includes('@')) {
    throw new Error(`The string: ${input} is not a valid email address`);
  }

}

const email = 'foo@example.org';
assertValidEmail(email);
const user:User = {
  firstName: 'Evert',
  lastName: 'Pot',
  email,
}

save(user);
```

This compiles to:

```javascript
const validEmail = Symbol('valid-email');

function save(user) {

}

function assertValidEmail(input) {
    if (!input.includes('@')) {
        throw new Error(`The string: ${input} is not a valid email address`);
    }
}

const email = 'foo@example.org';
assertValidEmail(email);
const user = {
    firstName: 'Evert',
    lastName: 'Pot',
    email,
};
save(user);
```

Effectively, having a variable that has the type `Email` is proof that at some point
`assertValidEmail` was called, and it didn't throw the exception.

[1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol