# ToDo

* Allow the ability to create a timeout with no expiration. This means I need a way to bind a section to the timeout.
* Change `await` to accept multiple sections
Example:

```vb
await {_timeout} and run ({-section::1},{-section::2})
# Or
await {_timeout} and run {-section::1}
await {_timeout} and run {-section::2}
```

## Bugs

* If `complete` is called on a timeout before `await`, nothing will run because there is no section to call back to.
  Not necessarily a bug, but should probably have some logging.

## Ideas

* Allow data to be set to a timeout via property.
  This means I need to make `complete timeout {_timeout}` not override the timeout's data and also call the section with data.
  However I'm not sure if I should make the first obj in data be the section's input, or first obj provided to complete. Probably complete.

  Maybe ^^^ should be additional data instead of section args... meta data?

* Multiple owners for timeouts...?

* Cancel reasons for timeouts.
    `cancel %object% [because| due to] %strings%`

* Timeout states...? Instead of just isCompleted, [pending, completed, expired, cancelled]
    `%object%'s state`. Would love me some enums. Could maybe do strings and but map {-enum::timeout::PENDING}? Seems kinda bad ngl. Maybe integers? No idea ngl. I could do custom syntax for it but ehhh

* Remaining time Expression?

* Reverse lookup for timeout owners.
    `[active] timeouts of %object%`

* Auto cleanup for timeouts? Maybe like an option, `cleanupDelay: 5 seconds` After `complete` is called?
    Potentially also have `new [[temp]orary|[perm]anent] timeout`

* Debug effect for a timeout
    Ex:

```v
Timeout:
    data:
    owner:
    section:
    result:
    isCompleted:

    taskID: 
    duration:
```
