1.  **VettID Lexicon**

    1.  **Action**-- Actions are atomic serverless functions, scripts
        or executables that perform a single task.
    2.  **App Manager Service** -- The App Manager Service is the
        Manager Service for an AppSpace. It monitors AppSpace.Queue for
        new Internal Events, applies applicable Rules and then calls the
        appropriate Event Handler.
    3.  **AppSpace** -- AppSpace is the EventSpace for cloud based
        applications. Users of a cloud based application use the
        AppSpace to deliver External Events to the application.
    4.  **AppSpace Cluster**-- A Kubernetes based application that
        provides a combination of the EventSpace and Data Vault
        functionality for cloud based applications. It provides an API
        for integration in lieu of the VettID Mobile App.
    5.  **Concierge Service** -- The Concierge Service is the Manager
        Service for the EventSpace Manager. It monitors the
        Concierge.Queue, applies applicable Rules and then calls the
        appropriate Event Handler.
    6.  **Data Vault** -- An Ubuntu server or appliance (arm64 or amd64)
        that has outbound Internet egress. It should have secure boot
        enabled and the hard drive should be encrypted. The firewall
        should be enabled to block all incoming connections. SSH may be
        exempted. It runs NATS server with JetStream enabled and the
        Manager Service.
    7.  **Event Handler**-- An Event Handler defines and executes the
        workflow for processing a specific event type. Event Handlers
        invoke Actions to perform steps in the workflow and can use the
        Event Handler namespace to share data between Actions or
        maintain state.
    8.  **EventSpace** -- A sub-topic of the NATS Subscriber namespace
        that is accessible to the owner's connections. Every connection
        receives a NATS NKey which allows write access. Every EventSpace
        must have a Profile and a topic where connections can deposit
        External Events for the owner.
    9.  **EventSpace Server**-- Software deployed onto an Ubuntu server
        that is Internet accessible. An admin account is used to
        create/remove unique NATS Subscriber namespaces. Any number of
        Subscriber namespaces may be created as long as resources allow.
    10. **Event Type** -- Defines the Event Handler required to process
        the event.
    11. **Event Watcher**-- An Event Watcher is launched by a Manager,
        App Manager or Concierge Service to monitor a NATS namespace for
        new External Events. When new External Events are received the
        Event Watcher pulls the appropriate private key based on the
        keyID, decrypts the External Event and writes an Internal Event
        to the Queue.
    12. **External Event**-- External Events are comprised of a keyID
        and an encrypted JSON object. The JSON object is encrypted using
        a connection specific public key identified by the keyID. The
        JSON object must include a timestamp, connection ID Event Type,
        Event ID and Data. External Events are used anytime the Event
        transits the Internet.
    13. **Feed** -- A time series list of all of the owner's Events. It
        may be filtered and searched. Events in the feed may be tagged,
        archived or deleted.
    14. **Internal Event** -- Internal Events are created by Event
        Watchers when they collect and decrypt External Events. Internal
        Events are not encrypted and are only used within a single NATS
        server. Internal Events are used to communicate between
        Services, Event Handlers and Actions.
    15. **Manager Service** -- The Manager Service runs on the Data
        Vault and watches the DataVault.Queue for Internal Events. When
        the Manager Service receives a new Internal Event, it applies
        any Rules related to the ConnectionID and then call the
        appropriate Event Handler to process the Event based on the
        Event Type/Event ID.
    16. **Message** -- Any text, personal data, private data, secrets or
        files that the owner wants to share with a connection. Files
        include images, audio files, video files and packages. Every
        file must include a hash for verification.
    17. **OwnerSpace** -- A sub-topic of the Subscriber namespace that
        is used by the Data Vault and VettID Mobile App to exchange
        External Events.
    18. **Personal Data** -- Data that an owner is willing to openly
        share with connected users and apps. Examples include name,
        email address, age, location, phone numbers, accounts
        (X/Twitter, Facebook, LinkedIn, Instagram, etc.), public keys,
        web sites, etc.
    19. **Private Data** -- Sensitive data that an owner may share with
        a connected user or app based on owner authorization or a rule
        created by the owner. This includes passport numbers, Social
        Security Numbers, date of birth, place of birth, account
        numbers, encryption keys, etc.
    20. **Profile** -- A collection of Personal Data that the owner
        wishes to share in an EventSpace. It will be shared with every
        connection to that EventSpace. Owners can create multiple
        profiles but only one may be applied to an EventSpace. Every
        Profile must include a public key, a list of supported Event
        Types, first name, age, state and country at a minimum.
    21. **Queue** -- NATS topics that exist on the Data Vault,
        EventSpace Server and AppSpace Cluster that are used to trigger
        Event Handlers or pass Internal Events to a running Event
        Handler. Manager Services monitor their respective Queues to
        identify when an Event Handler needs to be called.
    22. **Request** -- A proposal to the owner sent by a connection that
        allows the terms of the proposal to be modified by the owner.
        Modified requests must be approved by the initiating connection.
        A request approved by both parties becomes a Transaction.
    23. **Rules** -- Rules define the criteria for the automation of a
        Request or a Transaction
    24. **Secret** -- Any private keys, numbers or strings that the
        owner wants to store in their VettID Credential. Secrets may be
        displayed in the VettID Mobile App upon owner authentication but
        otherwise are meant to be used by an Event Handler as part of a
        transaction or request. The design goal is to limit usage of
        secrets to the Data Vault so the Secret itself never leaves
        the DV. Secrets can include data like seed phrases or private
        keys for crypto wallets.
    25. **Transaction** -- A proposal to the owner sent by a connection
        with fixed terms. Owners may reject or authorize a transaction.
        Approved modified Requests and Requests authorized by an owner
        without modification are treated as transactions.
    26. **VettID Credential** -- An encrypted JSON object containing the
        owner's factor of authentication and Secrets. Used to
        authenticate the owner and authentication is required to
        add/remove/use any secrets it contains. Only the Data Vault
        holds the decryption key. Key pair used to encrypt and decrypt
        the credential is rotated on every use.
    27. **VettID Mobile App** -- iOS/Android app that is used to connect
        to and manage the owner's Data Vault. It communicates with the
        Data Vault via the OwnerSpace namespace.
