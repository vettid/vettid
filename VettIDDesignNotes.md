1. **Design Notes**
   1. VettID should always fail safe and never fail open.
   1. VettID is designed to support any asymmetric key pair algorithm (e.g. RSA, ElGamal, post-quantum etc.). 4096 bit RSA key pairs should be the default.
   1. Every key pair generated must be assigned a keyID.
   1. Key pairs should be automatically rotated on a regular basis.
   1. VettID uses NATS (with JetStream enabled) for data storage, management and messaging. VettID leverages NATS for queuing messages, as a key value store and an object store.
   1. NATS should be configured to persist data to disk and encrypt it.
   1. NATS should be configured to support TLS to encrypt data in transit.
   1. NATS naming convention should use capitalization for internal namespaces (not Internet exposed) and all lowercase for externally exposed namespaces
   1. VettID Data Vaults are assigned a NATS Account and Nkey for control of the owner’s Subscriber namespace
   1. NATS Users/Nkeys are created for each connection that allow the connection to access the owner’s EventSpace
   1. VettID integration/communication is based on External Events.
   1. Events are are written to specific queues based on purpose(e.g. DataVault.Queue for internal communication in the Data Vault or subscriber.guid.eventspace for external communications with connections).
   1. Internal Events are decrypted JSON objects that must have a minimum of a timestamp, event type, eventID and connectionID
   1. External Events have 2 elements, a keyID and the encrypted JSON object. This encryption is in addition to the TLS connection.
   1. Services (e.g. AppManager, Manager or Concierge) watch their respective Queues for new Internal Events, evaluate the event type/eventID for the new event received, evaluate the rules associated with the event type/connectionID and launches Event Handlers to process the events (or passes events to an existing Event Handler based on the eventID)
   1. Services launch Event Watchers to watch remote queues
   1. Event Watchers monitor defined queues, retrieve events, decrypt events based on the keyID and write the decrypted events to the Service Queues
   1. Every user must have an EventSpace for integration/communication with other users, apps and services.
   1. Every EventSpace must publish a profile identifying the owner, sharing any desired data, listing the supported event types and defining any trusted resources or links
   1. Identity is defined by the owner but requires a public key, first name, age and generalized location (e.g. city, state, province) at a minimum.
   1. Shared Data are properties from the owner’s Data Vault they are willing to share with their connections (e.g. contact information, BTC address)
   1. Trusted Resources identify packages that may be used to install new Event Handlers or mobile applications
   1. Trusted Resources listed must include a version number and a hash to validate the package
   1. Trusted Links allow owners to share their meaningful links like a link tree
   1. Every connection (user, app or service) is based on a unique per connection key exchange.
   1. Unique connection key pairs must be regularly rotated.
   1. Users can define rules that determine how events from specific connections, or specific event types, are handled for automation purposes.
