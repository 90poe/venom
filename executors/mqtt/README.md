# Venom - Executor MQTT

Step to use read / write on MQTT topics.

An example can be found in `tests/mqtt.yml`

## Input

ClientType can be one of:
* "consumer"
* "producer"
* "persistent_queue"

"persistent_queue" prepares a queue by setting the session_clean property so that later "consumer" steps can retrieve data.
Note the use of the name "persistent_queue" rather than MQTT's more usual clean_session. This is as attempt to reduce unexpected behaviour when one leaves the "persistent_queue" option out of the step config.

## Limitations and Future Improvements

### Limitations

Given the way that venom starts each task with no state held across steps and that mqtt would lose messages if it subscribed after a message is sent we need a mechanism to register persistent topics for a later mqtt step. This will reduce our ability to test all combinations MQTT session settings. 

MQTT could deliver messages after connection, but before the service gets a chance to bind a subscription handler. We attach a handler using AddRoute before connection and then subscribe after connection. This should ensure that no messages are lost and subscriptions behave as you need.

### Future work 

* Add the ability to obtain message content from files rather than from within the yaml config. This need not be specific to this executor
* Add support for codecs so we can support serialisation formats other than json. This should really be a capability that any executor can take advantage of rather than solved for each individually
* Add TLs support for MQTT interface
