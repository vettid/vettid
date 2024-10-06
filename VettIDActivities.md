**VettID Activities**

1. **EventSpace Manager Install Activity**
   1. install script hosted on GitHub
   1. check for updates
      1. apt update
      1. apt upgrade
   1. check firewall
      1. ufw deny incoming
      1. allow http/https/NATS
      1. warn about allowing ssh
   1. install TLS certificate – one of the following
      1. generate self-signed certificate
      1. install certmagic
         1. run certmagic to generate certificate
      1. install 3<sup>rd</sup> party certificate
   1. install NATS
      1. install NATS server
      1. install NATS client
      1. enable JetStream
      1. enable TLS
      1. enable persist to disk
      1. run NATS server under systemd
   1. install Concierge Service
      1. install Event Handlers
      1. install Actions
      1. Create NATS Operator for the Concierge Service
      1. connect Concierge Service to NATS
      1. initialize NATS namespaces
         1. Concierge
            1. includes kv store
         1. Queue
         1. Registration
         1. Subscriber
            1. create subscriber.concierge.queue for messaging with subscribers
   1. install EventSpace Manager Administrator (esma) CLI for administration
   1. Concierge Service watches Queue
      1. launches Subscriber Event Watcher to watch subscriber.concierge.queue
1. **Create EventSpace Subscriber Activity**
   1. use esma to create Subscriber Alice (e.g. esma -s subscriberID -o one\_time\_password -c)
      1. create unique Registration namespace (e.g. registration.Alice)
         1. based on subscriberID
   1. generate Concierge Service Connection key pair and unique keyID
      1. store Concierge Service Connection key pair and keyID in Concierge kv store
         1. add Concierge Service Connection public key and keyID  to unique Registration namespace
      1. create unique Subscriber namespace (e.g. subscriber.guid)
         1. assign a unique GUID
         1. add unique Subscriber URI to unique Registration namespace
      1. create NATS Token for Registration
         1. scoped to registration.Alice (read) + registration.Alice.app (write) + subscriber.concierge.queue (write only)
         1. set to OTP value
      1. create NATS Account and NKey for Subscriber
         1. scoped to unique Subscriber namespace (full control) + subscriber.concierge.queue (write only)
         1. place it in the unique Registration namespace
         1. store subscriber details in Concierge kv store
      1. Output is the URI for the unique Registration namespace and the OTP
1. **Data Vault Install Activity**
   1. install script hosted on GitHub
   1. check for encrypted disk and secure boot
   1. check for updates
      1. apt update
      1. apt upgrade
   1. check firewall
      1. ufw deny incoming
      1. warn about allowing ssh
   1. install NATS
      1. install NATS server
      1. install NATS client
      1. enable JetStream
      1. enable persist to disk
      1. run NATS server under systemd
   1. install Manager Service
      1. install Event Handlers
      1. install Actions
      1. create NATS Operator for Manager Service
      1. connect Manager Service to NATS
      1. initialize NATS namespaces
         1. Manager namespace
            1. includes kv store
            1. includes object store
         1. Vault namespace
            1. includes kv store
            1. includes object store
         1. Connections namespace
            1. includes kv store
            1. includes object store
         1. Profiles namespace
            1. includes kv store
            1. includes object store
         1. Event Handlers namespace
            1. includes kv store
            1. includes object store
         1. DataVault.Queue namespace
      1. Manager Service watches DataVault.Queue
1. **Data Vault Registration Activity**
   1. Alice runs the dvreg action (command) with URI and OTP (e.g. dvreg -u URI -o OTP)
      1. connects to Registration URI with provided OTP
      1. get Concierge Service Connection public key and keyID
         1. store in Manager kv store
      1. get NATS NKey for Subscriber
         1. store in Manager kv store
      1. get unique Subscriber namespace
         1. store in Manager kv store
      1. connect to the OwnerSpace namespace
         1. initialize the OwnerSpace namespace
            1. includes object store
      1. connect to the EventSpace namespace
         1. initialize the EventSpace namespace
            1. includes object store
      1. generate Data Vault key pair and keyID
         1. store in Manager kv store
      1. generate VettID App key pair and KeyID
         1. store in Manager kv store
      1. create NATS user & NKey for VettID Auth
         1. Store in Manager kv store
         1. scoped to unique subscriber.guid.OwnerSpace.Auth (write) subscriber.guid.OwnerSpace.AuthApp (read) + subscriber.concierge.queue (write)
         1. write to registration.Alice.app
      1. Manager Service launches an EventSpace Event Watcher to watch subscriber.guid.eventspace
      1. post a message to subscriber.concierge.queue indicating Data Vault registration is complete and providing the Data Vault public key and keyID encrypted using Concierge Service Connection public key
      1. Subscriber Event Watcher gets the message, decrypts it and posts it to Queue
         1. Concierge Service gets the message and launches a Registration Event Handler
            1. Registration Event Handler posts a confirmation message to the subscriber.guid.eventspace encrypted using the Data Vault public key
         1. EventSpace Event Watcher gets the message, decrypts it and paces it in DataVault.Queue
      1. Manager Service gets the message, evaluates the event type and launches an OwnerSpaceAuth Event Watcher to watch subscriber.guid.OwnerSpace.Auth
1. **VettID App Registration Activity**
   1. Alice launches VettID App
   1. VettID App asks to enable OS authentication to launch app (e.g. device PIN or biometrics)
      1. do not proceed if not enabled
   1. Alice prompted to register by providing a URI and OTP.
      1. May be provided via QR code or an embedded link in an email or SMS
   1. VettID App connects to Registration URI with provided OTP
   1. get NATS NKey for VettID Auth
      1. securely store on device
   1. get unique Subscriber namespace
      1. securely store on device
   1. generate session key pair
      1. store temporarily
   1. connect to subscriber.guid.OwnerSpace.Auth
      1. post registration message including session public key
         1. not encrypted (keyID 0)
   1. OwnerSpaceAuth Event Watcher gets the message, decrypts it and writes it to DataVault.Queue
      1. Manager Service gets the message, evaluates it and launches a Data Vault Registration Event Handler
      1. Data Vault  Registration Event Handler acknowledges registration by posting response in OwnerSpace.AuthApp
         1. response is encrypted using the session key and includes a public key and keyID for the Data Vault and a private key and keyID for VettID App
      1. VettID app gets and decrypts the response
         1. securely stores the Data Vault public key and keyID on device
         1. securely stores the VettID App private key and keyID on the device
      1. posts an acknowledge message in OwnerSpace.Auth
         1. encrypted using Data Vault public key
      1. OwnerSpaceAuth Event Watcher gets the message, decrypts it and puts in in the DataVault.Queue
         1. Manager service gets the message and launches an OwnerSpace Event Handler for OwnerSpace.Queue
         1. post a message to subscriber.concierge.queue indicating VettID app registration is complete
      1. Subscriber Event Watcher gets the message, decrypts it and writes it to Queue
      1. Concierge Service gets the message and launches a Registration Event Handler
         1. Registration Event Handler posts a confirmation message to subscriber.guid.eventspace encrypted using Data Vault public key
      1. Registration Event Handler removes unique Registration namespace
      1. Registration Event Handler disables Token (OTP)
         1. posts a message to Queue for NKey rotation
      1. Concierge Service gets the message and launches a Concierge Nkey Rotation Event Handler for the Subscriber
         1. New Subscriber NKey created , encrypted with Data Vault public key and written to subscriber.guid.eventspace
         1. EventSpace Event Watcher gets the message, decrypts it and adds it to DataVault.Queue
         1. Manager service gets the message from DataVault.Queue and launches an NKey Rotation Event Handler to process it
            1. NKey  Rotation Event Handler writes updated Subscriber NKey to Manager kv store
            1. NKey Rotation Event Handler reconnects using new Subscriber NKey
            1. NKey Rotation Event Handler uses new Subscriber NKey to generate a new VettID Auth Nkey
               1. posts a message to DataVault.Queue to trigger a reconnection for EventSpace, OwnerSpace, and OwnerSpace.Auth Event Watchers using the new NKey
            1. Manager Service gets the message, evaluates the event type and restarts the EventSpace, DataVault.Queue and OwnerSpace.Auth Event Watchers
            1. NKey Rotation Event Handler posts updated VettID Auth NKey to OwnerSpace.AuthApp  encrypted using VettID App public key
            1. VettID app retrieves the message, decrypts it and securely stores the key on the device
               1. ` `reconnects using the new VettID Auth NKey
               1. posts an encrypted message to OwnerSpace.Auth acknowledging rotation
            1. OwnerSpace.Auth Event Watcher gets the message decrypts it and writes it to DataVault.Queue
            1. Manager Service gets the message and passes it to the NKey Rotation Event Handler
            1. NKey Rotation Event Handler posts a rotation complete message to subscriber.concierge.queue encrypted using Concierge Service public key
         1. Subscriber Event Watcher gets the message, decrypts it and writes it to Queue
         1. Concierge Service gets the message and passes it to the Concierge NKey Rotation Event Handler
         1. Concierge NKey Rotation Event Handler removes the old Subscriber Account/NKey with NATS
         1. Concierge NKey Rotation Event Handler posts a message to subscriber.guid.eventspace confirming key rotation complete.
         1. EventSpace Event Watcher gets the message, decrypts it and writes it to DataVault.Queue
         1. Manager Service gets the message and passes it to the NKey Rotation Event Handler
         1. NKey Rotation Event Handler removes old VettID App user/NKey
   1. NKey Rotation Event Handler posts a message to OwnerSpace.AuthApp indicating registration complete encrypted using the VettID App public key
   1. VettID app gets the message, decrypts it and displays it to the user
1. **Create VettID Credential Activity**
   1. Alice clicks “Create Credential” in VettID app
      1. post an enrollment message to OwnerSpace.Auth encrypted using Data Vault public key
   1. OwnerSpaceAuth Event Watcher gets the message, decrypts it and places it in the DataVault.Queue
   1. Manager Service gets the message, checks the Event Type and launches an Enrollment Event Handler
      1. Enrollment Event Handler creates skeleton VettID Credential JSON (keys, no values)
         1. creates a VettID Owner user and NKey scoped to  subscriber.guid.OwnerSpace.Feed\* (read) + subscriber.guid.OwnerSpace.Queue (write) + subscriber.concierge.queue (write)
         1. moves the VettID Owner NKey into the skeleton
      1. Enrollment Event Handler posts a message to OwnerSpace.AuthApp prompting for value to add to the first key (challenge) encrypted using the VettID App public key
      1. VettID app retrieves the message, decrypts it and presents it to Alice
         1. Alice provides/selects a challenge (e.g. password, PIN, custom secret question)
            1. VettID app encrypts the challenge with the Data Vault public key and writes it to OwnerSpace.Auth
         1. OwnerSpaceAuth Event Watcher retrieves the message, decrypts it and adds the message to DataVault.Queue
         1. Manager Service gets the message and gives it to the Enrollment Event Handler
         1. Enrollment Event Handler adds the challenge to the skeleton JSON
      1. Enrollment Event Handler posts a message to OwnerSpace.AuthApp prompting for value to add to the next key (response) encrypted using the VettID App public key
      1. VettID app retrieves the message, decrypts it and presents it to Alice
         1. Alice provides a response
            1. VettID app encrypts the response with the Data Vault public key and writes it to OwnerSpace.Auth
         1. OwnerSpaceAuth Event Watcher retrieves the message, decrypts it and adds the message to DataVault.Queue
         1. Manager Service gets the message and gives it to the Enrollment Event Handler
         1. Enrollment Event Handler adds the response to the skeleton JSON
      1. Enrollment Event Handler posts a message to OwnerSpace.AuthApp prompting for any secrets to be added to the credential (e.g. BTC private key, seed phrase) encrypted using the VettID App public key
      1. VettID app retrieves the message, decrypts it and presents it to Alice
         1. Alice provides a secret
            1. VettID app encrypts the message with the Data Vault public key and writes it to OwnerSpace.Auth
         1. OwnerSpaceAuth Event Watcher retrieves the message, decrypts it and adds the message to DataVault.Queue
         1. Manager Service gets the message and gives it to the Enrollment Event Handler
         1. Enrollment Event Handler adds the secret to the skeleton JSON
      1. Process repeated for any other secrets Alice wishes to add.
   1. Alice completes adding requested data to her credential and selects “Finish Credential” in the app
      1. message written to OwnerSpace.Auth to issue credential encrypted using Data Vault public key
         1. OwnerSpaceAuth Event Watcher retrieves the message, decrypts it and adds the message to DataVault.Queue
         1. Manager Service gets the message and gives it to the Enrollment Event Handler
         1. Enrollment Event Handler finalizes the credential
            1. gets a new Credential key pair
            1. stores Credential private key in Manager kv store
            1. encrypts the VettID Credential JSON with the Credential public key
            1. writes the encrypted VettID Credential to OwnerSpace.AuthApp.Objectstore
            1. posts a message to OwnerSpace.AuthApp that credential is ready encrypted using VettID App public key
         1. VettID app retrieves the message, decrypts it and collects the credential
            1. pulls encrypted credential from object store
            1. securely stored on device
            1. posts a confirmation message to OwnerSpace.Auth encrypted using Data Vault public key
         1. OwnerSpaceAuth Event Watcher retrieves the message, decrypts it and adds the message to DataVault.Queue
         1. Manager Service gets the message and gives it to the Enrollment Event Handler
         1. Enrollment Event Handler removes the credential from OwnerSpace.AuthApp.Objectstore
1. **Connect to Data Vault Activity**
   1. Alice clicks “Connect to Data Vault” in the mobile app
      1. VettID app posts a connection request to OwnerSpace.Auth encrypted using Data Vault public key
         1. VettID app puts VettID Credential into OwnerSpace.Auth.Objectstore
      1. OwnerSpaceAuth Event Watcher grabs the message, decrypts it and writes it to the DataVault.Queue
      1. Manager picks up the message and launches a Vault Connection Event Handler
         1. Vault Connection Event Handler retrieves the VettID Credential from OwnerSpace.Auth.Objectstore
            1. Vault Connection Event Handler removes the VettID Credential from the object store
            1. Vault Connection Event Handler grabs the Credential private key and decrypts the VettID Credential
            1. Vault Connection Event Handler creates authentication request based on the challenge contained in the JSON
               1. encrypted with VettID App public key
               1. written to OwnerSpace.AuthApp
            1. VettID app retrieves the message, decrypts it and presents the challenge to Alice
               1. Alice provides her response and the VettID app encrypts it with the Data Vault public key
                  1. written to OwnerSpace.Auth
            1. OwnerSpaceAuth Event Watcher retrieves the message, decrypts it and then writes it to DataVault.Queue
               1. Manager Services gets the message and passes it to the Vault Connection Event Handler
                  1. Vault Connection Event Handler compares the response with the response contained in the JSON
                  1. if it matches, connection Event Handler gets a new VettID Owner key pair
                  1. Stores the new VettID Owner public key in the Manager kv store
                  1. collects the VettID Owner NKey from the credential
                  1. Encrypts the new VettID Owner private key and VettID Owner NKey with the current VettID App public key and posts it to OwnerSpace.AuthApp
               1. VettID app retrieves the message, decrypts it, stores the new VettID Owner private key on the device and temporarily stores the NKey
               1. VettID app uses the NKey to connect to watch OwnerSpace.Feed and write to OwnerSpace.Queue
                  1. VettID app posts a confirmation message that the private key was received to OwnerSpace.Queue encrypted using the Data Vault public key
                  1. OwnerSpace Event Watcher retrieves the message, decrypts it and writes it to DataVault.Queue
                     1. Manager Service gets the message and passes it to the Authentication Event Handler
                        1. Credential is encrypted using a new Credential key pair
1. new Credential private key stored in Manager kv store
1. new VettID Credential is written to OwnerSpace.Feed.Objectstore
   1. posts a connected message to OwnerSpace.Feed along with location of new the new VettID Credential encrypted with the new VettID Owner public key
   1. stops OwnerSpace.Auth Event Watcher
   1. VettID app retrieves the message, decrypts it and shows the user as connected in the app
      1. connects to ObjectSpace.Feed.Objectstore and collects new VettID Credential
      1. securely stores ne VettID Credential on the device.
   1. VettID app sends completed message to OwnerSpace.Queue encrypted using data Vault public key
   1. OwnerSpace Event Watcher gets the External Event, decrypts it and writes it to DataVault.Queue
   1. Manager Service gets the Internal Event, evaluates it and passes it off to the Authentication Event Handler
      1. Authentication Event Handler gets the message and removes the new VettID Credential from OwnerSpace.Feed.Objectstore
1. **Configure Data Vault Activity**
   1. Alice selects Configure Data Vault in the VettID mobile app
   1. VettID mobile app generates message to request all owner KV entries from Vault namespace
      1. Encrypts the message with the Data Vault public key and posts the message to OwnerSpace.Queue
      1. OwnerSpace Event watcher gets the message, decrypts it and writes it to DataVault.Queue
      1. Manager service gets the message, checks the event type and launches a Vault Event Handler
      1. Vault Event Handler gets all keys and values from the vault, places them into a JSON object, encrypts the message with the VettID Owner public key and posts the message to OwnerSpace.Feed
   1. VettID mobile app grabs the message, decrypts it using the VettID Owner private key and presents the data to Alice
      1. Alice reviews the vault data (kv data) and selects a key to update it’s value
         1. if Alice updates a value, she is prompted to save her change
         1. if save is selected, VettID mobile app creates a message with the updated value, encrypts it using the Data Vault public key and posts it to OwnerSpace.Queue
         1. OwnerSpace Event Watcher gets the message, decrypts it using the Data Vault private key and writes the message to the Data Vault queue
         1. Manager service gets the message from the queue, checks the event type and passes it to the Vault Event Handler
         1. Vault Event Handler updates the value identified and posts a completed message to DataVault.Queue
         1. Manager service gets the message from DataVault.Queue and passes it to the Vault Event Handler
         1. Vault Event Handler encrypts the completed message with the VettID Owner public key and writes the message to OwnerSpace.Queue
         1. VettID mobile app grabs the message, decrypts it and presents the completed message to Alice.
         1. May be repeated for any number of updates.
      1. Alice may also create new keys and add values to those keys.
1. **Add a Secret Activity**
   1. Alice selects “Create a New Secret” in the VettID mobile app
      1. VettID mobile prompts Alice for the secret name and value
      1. Alice provides the secret name and value
      1. new secret details are encrypted using the Data Vault public key and written to OwnerSpace.Queue
      1. OwnerSpace Event Watcher gets the message, decrypts it and drops it in DataVault.Queue
      1. Manager Service gets the message, evaluates the event type and launches a Authentication Event Handler
         1. Authentication Event Handler generates an authentication request, encrypts it using the VettID Owner public key and writes it to OwnerSpace.Feed
         1. VettID mobile app gets the message, decrypts it and asks Alice to authenticate
         1. Alice agrees
         1. VettID app puts VettID Credential into OwnerSpace.Queue.Objectstore
      1. VettID mobile app posts a confirmation message encrypted using the Data Vault public key to OwnerSpace.Queue pointing to the credential
      1. OwnerSpace Event Watcher gets the message, decrypts it and places it in the DataVault.Queue
      1. Manager gets the message and provides it to the Authentication Event Handler
         1. Authentication Event Handler gets the VettID Credential from OwnerSpace.Queue.Objectstore
            1. Authentication Event Handler removes the VettID Credential from the object store
            1. Authentication Event Handler gets the Credential private key and decrypts the VettID Credential
            1. Authentication Event Handler creates authentication request based on the challenge contained in the JSON
               1. encrypted with VettID Owner public key
               1. written to OwnerSpace.Feed
            1. VettID app gets the message, decrypts it and presents the challenge to Alice
               1. Alice provides her response and the VettID app encrypts it with the Data Vault public key
                  1. written to OwnerSpace.Queue
               1. OwnerSpace Event Watcher gets the message, decrypts it and places it in DataVault.Queue
               1. Manager Service gets the message and provides it to the Authentication Event Handler
               1. Authentication Event Handler compares the response with the response contained in the JSON
               1. if it matches, Authentication Event Handler adds the new secret into the credential JSON
                  1. Event Handler grabs a new Credential key pair
                  1. stores Credential private key in Manager kv store
                  1. encrypts the VettID Credential JSON with the new Credential public key
                  1. writes the encrypted VettID Credential to OwnerSpace.Feed.Objectstore
                  1. posts a message to OwnerSpace.Feed that credential is ready encrypted using VettID Owner public key
                  1. VettID app retrieves the message, decrypts it and collects the credential
                     1. pulls encrypted credential from object store
                     1. stored on device
                     1. posts a confirmation message to OwnerSpace.Queue encrypted using Data Vault public key
               1. OwnerSpace Event Watcher receives the message, decrypts it and writes it to DataVault.Queue
                  1. Manager Service gets the message and passes it to Authentication Event Handler
                     1. Authentication Event Handler removes the credential from OwnerSpace.Feed.Objectstore
                     1. posts a secrets updated message to OwnerSpace.Feed encrypted with the VettID Owner public key
                     1. VettID app retrieves the message, decrypts it and notifies Alice that the secret was saved
1. **Add a Connection Activity**
   1. Alice selects “New Connection” in the VettID App
   1. VettID App posts a new connection message to OwnerSpace.Queue encrypted using Data Vault public key
      1. OwnerSpace Event Watcher gets the message, decrypts it and writes it to DataVault.Queue
      1. Manager Service gets the message, evaluates the event type and launches a Connection Event Handler to process the message
         1. Connection Event Handler generates a new NATS user/NKey scoped to subscriber.guid.eventspace (write)
         1. adds connection data to Connection namespace
         1. encrypts the connection NKey and EventSpace URI with the VettID App public key and posts it to OwnerSpace.Feed
      1. VettID App gets the message and decrypts it
         1. Alice is asked how she would like to share the invitation

NKey and URI are packaged as a QR Code or link that can be shared

1. Alice shares with Bob
1. Bob takes a photo of the QR code (or connection user clicks the link)
   1. QR Code (or link) launches VettID App and asks if Bob would like to accept the invitation
   1. Bob selects yes
      1. Bob’s VettID app generates an add connection message (includes Nkey and URI), encrypted with the connection’s Data Vault public key and posts it to the Bobs OwnerSpace.Queue
      1. Bob’s OwnerSpace Event Watcher gets the message, decrypts it and writes it to Bob’s DataVault.Queue
      1. Bob’s Manager Service gets the message, evaluates the event type and launches a Connection Event Handler
         1. Connection Event Handler stores the NKey and URI in the Connection namespace
         1. Connection Event handler uses the NKey to collect the Alice’s profile, encrypts it with the Bob’s VettID App public key and writes it to the Bob’s OwnerSpace.Feed
      1. Bob’s VettID App gets the event, decrypts it and presents it to Bob
         1. Bob evaluates the user’s profile and decides if he wants to accept the connection
         1. Bob accepts the connection and Bob’s VettID App writes a connection accepted message, encrypts it with the Bob’s Data Vault public key and writes it to the Bob’s OwnerSpace.Queue
      1. Bob’s OwnerSpace Event Watcher gets the message, decrypts it and writes it to the Bob’s DataVault.Queue
         1. Bob’s Manager Service gets the message, evaluates it and passes it to the Bob’s Connection Event Handler
         1. Bob’s Connection Event Handler creates a new NATS user/NKey scoped to Bob’s subscriber.guid.eventspace (write)
         1. Bob’s Connection Event Handler generates a new Connection key pair
            1. private key stored in Connection namespace
         1. Bob’s Connection Event Handler encrypts the new connection public key, NKey and EventSpace URI with the Alice’s public key (keyID 1) from her profile and writes it to the Alice’s subscriber.guid.eventspace
      1. Alice’s EventSpace Event Watcher gets the message, decrypts it and writes it to DataVault.Queue
      1. Alice’s Manager Service gets the message, evaluates it and passes it to a Connection Event Handler
      1. Connection Event Handler saves the public key, NKey and URI to the Connection namespace
      1. Connection Event Handler uses the NKey and URI to get Bob’s profile
         1. encrypts it with the VettID App public key and writes it to OwnerSpace.Feed
      1. Alice’s VettID App gets the message, decrypts it and presents the connection’s profile for Alice to evaluate
         1. Alice accepts the new connection and the VettID App generates an accepted message, encrypts it using the Data Vault public key and writes it to OwnerSpace.Queue
   1. Alice’s OwnerSpace Event Watcher gets the message, decrypts it and writes it to DataVault.Queue
      1. Alice’s Manager Service gets the message, evaluates it and passes it to the Connection Event Handler
         1. Connection Event Handler gets the accept message
            1. generates new Connection key pair
               1. saves Connection private key to Connection namespace
            1. generates new NATS user/Nkey
               1. saves NKey to Connection namespace
               1. removes old NATS NKey
            1. creates a message containing the new NKey and public key, encrypts it using the Bob’s Connection public key and writes it to the Bob’s subscriber.guid.eventspace
   1. Bob’s EventSpace Event Watcher gets the message, decrypts it and writes it to the Bob’s DataVault.Queue
   1. Bob’s Manager Service gets the message, evaluates it and passes it to the Connection Event Handler
      1. Bob’s Connection Event Handler gets the message, saves the new NKey and public key in the Connection namespace
      1. creates a new message for connection complete, encrypts it with Bob’s Connection public key and writes it to Alice’s subscriber.guid.eventspace
   1. Alice’s EventSpace Event Watcher gets the message, decrypts it and writes it to DataVault.Queue
   1. Alice’s Manager Service gets the message, evaluates it and passes it to Connection Event Handler
      1. Alice’s Connection Event Handler gets the message and posts a connection complete message encrypted using the Connection public key to the Bob’s subscriber.guid.eventspace
1. **Send a Message Activity**

1. **Create a Rule Activity**

1. **Make a Request Activity**

1. **Perform a Transaction Activity**

1. **Rotate Nkeys Activity**

1. **Rotate Connection Key Pair Activity**



