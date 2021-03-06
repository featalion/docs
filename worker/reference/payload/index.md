---
title: Sending and Accessing a Payload to IronWorker
layout: default
section: worker
breadcrumbs:
  - ['Reference', '/reference']
  - ['Payload', 'payload']
---

Workers are just sections of discrete, modular code that you queue tasks against.
It's rare, though, that you want to run the same code repeatedly with no changes or variables.
To address this, IronWorker has the concept of a "payload".
The payload is the same thing, conceptually, as an argument or variable that is passed to a
function, method, or command —
just a piece of information you want to supply at runtime and make available in the worker itself.

Client is able to specify payload for queued or scheduled workers.
Payloads are strings. But usually we suggest to use JSON format.
Also, when using `ruby`, `php`, and `node` runtimes JSON payload will be automatically parsed.

<section id="toc">
  <h3>Table of Contents</h3>
  <ul>
    <li><a href="#get_payload_in_a_worker">Get Payload in a Worker</a></li>
    <li><a href="#other_information">Other Information</a></li>
    <li><a href="#payload_filtering_in_the_hud">Payload Filtering in the HUD</a></li>
  </ul>
</section>

<h2 id="get_payload_in_a_worker">Get Payload in a Worker</h2>

When payload is posted to a worker when using the *queue* or *schedule* API endpoints, then it is stored in the database.
Before your worker is launched in the IronWorker [environment](/worker/reference/environment),
this payload is then stored in your worker's runtime directory. The location of that file is passed to the worker using the `-payload` command line flag.

To get the contents of your payload, you need to:

1. Read the `-payload` flag using `ARGV` (or whatever your language uses to read command line flags)
2. Open and read the file specified by the `-payload` flag
3. Parse the contents of the file (for example, if you encoded the payload when queuing the task)

Workers that use either `ruby` or `php` runtimes have more possibilities to access the payload.

### Access to a Payload in Ruby Runtime

Ruby workers have access to special methods to obtain the payload.

```ruby
payload # string representation of payload
params  # json parsed payload

# you can also access the following

config # your configuration variables see the configuration variables page for more info
iron_task_id # a worker's own task id, useful for checking status via api.
```

If specified payload is in a JSON format it will be parsed automatically into params.


### Access to a Payload in PHP Runtime

The payload in PHP runtime is accessible by calling method `getPayload()`.
If payload is a parsable JSON string it will be converted automatically.


```php
<?php
payload = getPayload(); // parsed JSON or string
?>
<!-- you can also access the following  -->
<?php
config = getConfig(); // parsed JSON or string
?>
```

### Access to a Payload in Node.js Runtime

```js
var worker = require('node_helper');
console.log("params:", worker.params);

// you can also access the following
console.log("config:", worker.config);
console.log("task_id:", worker.task_id);
```

<h2 id="other_information">Other Information</h2>

Your worker will also be passed `-id` and `-d` command line arguments.
The value of `-id` will be the ID of the task that is currently being executed,
and the value of `-d` will be the user-writable directory
that can be used for temporary storage for the duration of the task's execution.

<h2 id="payload_filtering_in_the_hud">Payload Filtering in the HUD</h2>

You can see your tasks payload in the [HUD](https://hud.iron.io).
Go to Worker project's section, click on "Tasks" tab and then on one of your workers where you use payload.
Click on "Details" link on task.

HUD filters payload by the next rule.
It looks for all keys on any level of nesting which contains substrings:

* `token`
* `security`
* `password`
* `secret`
* `pass`
* `connectionstring`
* `api_key`
* `license`

and change their values to `[FILTERED]`.

### Example: original payload


```js
{
  "database": {
    "connectionstring": "postgres://usr:pass@host:port/db"
  },
  "iron": {
    "project_id": "1234567890",
    "token": "TOKEN1234"
  },
  "3rdparty_service": {
    "user": "username",
    "service_pass": "userp4ss"
  }
}
```

### Example: payload visible through the HUD


```js
{
  "database": {
    "connectionstring": "[FILTERED]"
  },
  "iron": {
    "project_id": "1234567890",
    "token": "[FILTERED]"
  },
  "3rdparty_service": {
    "user": "username",
    "service_pass": "[FILTERED]"
  }
}
```
