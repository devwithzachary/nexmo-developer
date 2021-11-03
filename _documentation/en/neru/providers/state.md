---
title: State
description: The State Provider
navigation_weight: 4
---

# State Provider

The State provider allows you to store data on NeRu for your project instance to use. The state provider has a key-value store at its root. As well as lists and maps with various operations you can perform on them.

## Functions

* `set<T>(key: string, value: T)`
* `get<T>(key: string)`
* `del(key: string)`
* `hset(htable: string, keyValuePairs: [string, string][])`
* `hmget<T>(htable: string, keys: string[])`
* `hgetall<T>(htable: string)`
* `hexists(htable: string, key: string)`
* `hvals<T>(htable: string)`
* `hdel(htable: string, key: string)`
* `rpush<T>(list: string, value: T)`
* `lpush<T>(list: string, value: T)`
* `llen(list: string)`
* `lrange<T>(list: string, startPos: number, endPos: number)`

## Initializing the State Provider

To can access the state provider from a NeRu session:

```javascript
const router = neru.Router();
const session = neru.createSession();
const state = session.getState();
```

On subsequent calls to your code, you want to make sure that you are using the same session to have access to your state. You can do so by initializing your instance from an incoming request: 

```javascript
router.post("/onMessage", async (req, res, next) => {
  try {
    const session = neru.getSessionFromRequest(req);
    const state = session.getState();

    // Handle incoming message
  } catch (error) {
    next(error);
  }
});
```

## Instance Level State

Instance level state is a singleton which you can use to share data across multiple instances. It comes handy when different flows are working together e.g. data is coming through 3rd party API that affect logic of voice/messages flow:


```javascript
router.post("/onMessage", async (req, res, next) => {
  try {
    const state = neru.getInstanceState();

    // Handle incoming message
  } catch (error) {
    next(error);
  }
});
```

## Usage

For example, you can store, retrieve, and delete objects on a session using the key-value operations:

```javascript
router.post("/key-value", async (req, res, next) => {
  try {
    const session = neru.createSession();
    const state = session.getState();

    // store object
    await state.set("test_obj", { "foo": bar });

    // retrieve object
    const obj2 = await state.get("test_obj");

    // delete object
    await state.del("test_obj");

  } catch (error) {
    next(error);
  }
})
```

Or you can use instance state, and the hash table operations to persist data between sessions:

```javascript
router.post("/add-customer", async (req, res, next) => {
	try {
	  const instanceState = neru.getInstanceState();
  
	  const customer = req.body;
	  await instanceState.hset("customers", [[customer.phone , customer ]] )
	} catch (error) {
	  next(error);
	}
})
  
router.get("/on-phone-call", async (req, res, next) => {
	try {
	  const instanceState = neru.getInstanceState();
	  const number = req.query.number;
	  const customer = await instanceState.hget("customers", number);
	  console.log("customer", customer);
	  res.send(customer.name);
	} catch (error) {
	  next(error);
	}
})
```

```sh
curl --location --request POST '{INSTANCE_ENDPOINT}/add-customer' \
--header 'Content-Type: application/json' \
--data-raw '{
    "id":"1",
    "name": "John",
    "phone": "101"
}'


curl --location --request GET '{INSTANCE_ENDPOINT}/on-phone-call?number=101'
```