# Mockable

Based on: https://nalanj.dev/posts/mocking-without-loaders/

tl;dr: it's still too hard/annoying to mock ESM modules. Export mockable-wrapped functions, and your tests can override them as needed, with near-zero overhead in production.

🚨 🚨 **NOTE: Only works on functions.** 🚨 🚨

## Install

```sh
npm install @atcodes/mockable
```

## Usage

### 1A: Export mockable resources

When writing your own libraries and utilities, export them as mockable resources:

```js
import { mockable } from '@atcodes/mockable';

export const getDBConnection = mockable(getDBConnection);

async function getDBConnection(){ /* ... */ };
```

### 1B: Proxy 3rd party libraries as mockable

Need to mock a 3rd party library that you don't control? Proxy it as a mockable resource:

```js
import { somethingUseful } from 'something-useful';
import { mockable } from '@atcodes/mockable';

export const getSomethingUseful = mockable(somethingUseful);
```

### 2: Override in tests

```js
import { test } from "node:test";
import { assert } from "node:assert/strict";

//the function that we want to mock
import { getDBConnection } from "./get-db.js";

//the module that we're going to execute and which uses the mocked function
import { handleRequest } from "./handle-request.js";

test("success", async (t) => {
	// override findUser for this test
	getDBConnection.override(() => {
		return {
			query: (id) => {
				return {
					id,
					name: "Test Tester",
					email: "test@test.com",
					createdAt: new Date(),
					updatedAt: new Date(),
				};
			}
		};
	});

	// clear the override after the test runs
	t.after(() => getDBConnection.clear());

	const resp = await handleRequest("/users/12");
	assert.equal(resp.statusCode, 200);
});
```
