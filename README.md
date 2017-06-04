# simple_redis

[![crates.io](http://meritbadge.herokuapp.com/simple_redis)](https://crates.io/crates/simple_redis) [![Build Status](https://travis-ci.org/sagiegurari/simple_redis.svg)](http://travis-ci.org/sagiegurari/simple_redis) [![Build status](https://ci.appveyor.com/api/projects/status/knyrs33tyjqgt06u?svg=true)](https://ci.appveyor.com/project/sagiegurari/simple-redis) [![Crates.io](https://img.shields.io/crates/l/simple_redis.svg)](https://github.com/sagiegurari/simple_redis/blob/master/LICENSE) [![Documentation](https://docs.rs/simple_redis/badge.svg)](https://docs.rs/crate/simple_redis/) [![Crates.io](https://img.shields.io/crates/d/simple_redis.svg)](https://crates.io/crates/simple_redis)

> Simple [redis](https://redis.io/) client for [rust](https://www.rust-lang.org/).

* [Overview](#overview)
* [Usage](#usage)
* [Installation](#installation)
* [API Documentation](https://sagiegurari.github.io/simple_redis/)
* [Contributing](.github/CONTRIBUTING.md)
* [Release History](#history)
* [License](#license)

<a name="overview"></a>
## Overview
This library provides a very basic, simple API for the most common redis operations.<br>
While not as comprehensive or flexiable as [redis-rs](https://crates.io/crates/redis),
it does provide a simpler api for most common use cases and operations as well as automatic internal connection
and subscription (pubsub) handling.<br>
In addition, the entire API is accessible via redis client and there is no need to manage connection or pubsub instances in parallel.<br>
<br>
Connection resiliency is managed by verifying the connection before every operation against the redis server.<br>
In case of any connection issue, a new connection will be allocated to ensure the operation is invoked on a valid
connection only.<br>
However, this comes at a small performance cost of PING operation to the redis server.<br>
<br>
Subscription resiliency is ensured by recreating the internal pubsub and issuing new subscription requests
automatically in case of any error while fetching a message from the subscribed channels.
<br>
<br>
**This library is still in initial development stage and many more features will come soon.**

<a name="usage"></a>
## Usage
In order to use this library, you need to first include the crate as follows:

````rust
extern crate simple_redis;
````

Afterwards create a redis client using a connection string:

````rust
match simple_redis::create("redis://127.0.0.1:6379/") {
    Ok(mut client) =>  println!("Created Redis Client"),
    Err(error) => println!("Unable to create Redis client: {}", error)
}
````

Once you have a redis client, you can invoke any of the available commands directly or use the run_command function to invoke operations that were not implemented by the library.

````rust
match client.set("my_key", "my_value") {
    Err(error) => println!("Unable to set value in Redis: {}", error),
    _ => println!("Value set in Redis")
};

match client.get("my_key") {
    Ok(value) => println!("Read value from Redis: {}", value),
    Err(error) => println!("Unable to get value from Redis: {}", error)
};

/// run some command that is not built in the library
match client.run_command::<String>("ECHO", vec!["testing"]) {
    Ok(value) => assert_eq!(value, "testing"),
    _ => panic!("test error"),
};

/// publish messages
let mut result = client.publish("news_channel", "test message");
assert!(result.is_ok());

/// subscribe to channels
result = client.subscribe("important_notifications");
assert!(result.is_ok());
result = client.psubscribe("*_notifications");
assert!(result.is_ok());

loop {
    /// fetch next message
    match client.get_message() {
        Ok(message) => {
            let payload: String = message.get_payload().unwrap();
            assert_eq!(payload, "my important message")
        }
        _ => panic!("test error"),
    }
}
````

<a name="installation"></a>
## Installation
In order to use this library, just add it as a dependency:

```ini
[dependencies]
simple_redis = "*"
```

## API Documentation
See full docs at: [API Docs](https://sagiegurari.github.io/simple_redis/)

## Contributing
See [contributing guide](.github/CONTRIBUTING.md)

<a name="history"></a>
## Release History

| Date        | Version | Description |
| ----------- | ------- | ----------- |
| 2017-06-04  | v0.1.13 | Support more data types |
| 2017-06-03  | v0.1.10 | More commands added |
| 2017-06-03  | v0.1.8  | Maintenance |
| 2017-06-03  | v0.1.7  | pubsub support added |
| 2017-06-02  | v0.1.6  | Initial release. |

<a name="license"></a>
## License
Developed by Sagie Gur-Ari and licensed under the Apache 2 open source license.