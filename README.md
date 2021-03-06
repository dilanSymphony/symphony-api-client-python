# symphony-api-client-python

## Overview
This Symphony bot client is written in an event handler architecture. The client keeps polling a datafeed and responds to different types of [Real Time Events](https://rest-api.symphony.com/docs/real-time-events) it receives.

To build a functional bot which responds to different types of incoming messages from datafeed (Connection, IM, Chat Room, etc....), the respective
type of listener needs to be implemented by inheriting the interfaces in the **listeners** folder. Currently **ConnectionListener**, **imListener**, **RoomListener** interfaces are provided.

## Environment Setup
This client is compatible with **Python 3.6 or above**

Create a virtual environment by executing the following command **(optional)**:
``python -m venv ./venv``

Activate the virtual environment **(optional)**:
``source ./venv/bin/activate``

Install dependencies required for this client by executing the command below.
``pip install -r requirements.txt``

## Getting Started
### 1 - Prepare the service account
The Python client operates using a [Symphony Service Account](https://support.symphony.com/hc/en-us/articles/360000720863-Create-a-new-service-account), which is a type of account that applications use to work with Symphony APIs. Please contact with Symphony Admin in your company to get the account.

The client currently supports two types of Service Account authentications, they are
[Client Certificates Authentication](https://symphony-developers.symphony.com/symphony-developer/docs/bot-authentication-workflow#section-authentication-using-client-certificates)
and [RSA Public/Private Key Pair Authentication](https://symphony-developers.symphony.com/symphony-developer/docs/rsa-bot-authentication-workflow).

### 2 - Implement the event listeners
As an example, the **roomListenerTestImp** has been implemented to respond with "Hello World", to a chat room in which there is an incoming message. To respond to other types of events, respective Listeners need to be implemented.

### 3.1 - Run bot with config.json
**RSA Public/Private Key Pair** is the recommended authentication mechanism by Symphony, due to its robust security and simplicity.

To run the bot using the **RSA Public/Private Key Pair**, a **rsa_config.json** should be provided. In our example, the json file resides in the
**resources** folder but it can be anywhere.

An example **main_RSA.py** has been provided to illustrate how all components work together.

To run the bot using the **Client Certificates Authentication**, a **config.json** should be provided. In our example, the json file resides in the
**resources** folder but it can be anywhere.

An example **main_certificate.py** has been provided to illustrate how all components work together.

**Notes:**
Most of the time, the **port numbers** do not need to be changed.

An example of json has been provided below.  (The "botPrivateKeyPath" ends with a trailing "/")

    {
      "sessionAuthHost": "MY_ENVIRONMENT.symphony.com",
      "sessionAuthPort": 443,
      "keyAuthHost": "MY_ENVIRONMENT.symphony.com",
      "keyAuthPort": 443,
      "podHost": "MY_ENVIRONMENT.symphony.com",
      "podPort": 443,
      "agentHost": "MY_ENVIRONMENT.symphony.com",
      "agentPort": 443,

      // For bot RSA authentication
      "botPrivateKeyPath":"./sym_api_client_python/resources/",
      "botPrivateKeyName": "bot_private_key.pem",

      // For bot cert authentication
      "botCertPath": "/path/to/bot-cert/",
      "botCertName": "/bot-cert.p12",
      "botCertPassword": "bot-cert-password",

      "botUsername": "YOUR_BOT_USERNAME",
      "botEmailAddress": "YOUR_BOT_EMAIL_ADDRESS",

      "appCertPath": "",
      "appCertName": "",
      "appCertPassword": "",
      "authTokenRefreshPeriod": "30"

      // Optional: If all the traffic goes through a single proxy, set this parameter. If using multiple proxies or only using a proxy for some of the components, set them below and don't useproxyURL
      "proxyURL": "http://localhost:8888",
      "proxyUsername": "proxy-username",
      "proxyPassword": "proxy-password",

      // Optional: set this if traffic to pod goes through a specific, unique proxy
      "podProxyURL": "http://localhost:8888",
      "podProxyUsername": "proxy-username",
      "podProxyPassword": "proxy-password",

      // Optional: set this if traffic to agent goes through a specific, unique proxy
      "agentProxyURL": "http://localhost:8888",
      "agentProxyUsername": "proxy-username",
      "agentProxyPassword": "proxy-password",

      // Optional: set this if traffic to KeyManager goes through a specific, unique proxy
      "keyManagerProxyURL": "http://localhost:8888",
      "keyManagerProxyUsername": "proxy-username",
      "keyManagerProxyPassword": "proxy-password",


      // Required: If a truststore is required to access on-prem components, provide a path to the python truststore. Needs to be .pem file.  Instructions below for converting JKS to python pem truststore. If truststore is not needed, set value as empty string ("").
      "truststorePath": "/path/to/truststore.pem"
    }



### Example main class (using RSA)
Adjust the following paths in the sample to match your configuration
 - "sym_api_client_python/logs/example.log"
 - "sym_api_client_python/resources/config.json"

Example Main Class:

    import logging
    from sym_api_client_python.configure.configure import SymConfig
    from sym_api_client_python.auth.rsa_auth import SymBotRSAAuth
    from sym_api_client_python.clients.sym_bot_client import SymBotClient
    from sym_api_client_python.listeners.\
            im_listener_test_imp import IMListenerTestImp
    from sym_api_client_python.listeners.\
            room_listener_test_imp import RoomListenerTestImp


    def configure_logging():
        logging.basicConfig(
                filename='./logs/example.log',
                format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                filemode='w', level=logging.DEBUG
        )
        logging.getLogger("urllib3").setLevel(logging.WARNING)


    def main():
        print('Python Client runs using RSA authentication')

        # Configure log
        configure_logging()

        # RSA Auth flow: pass path to rsa config.json file
        configure = SymConfig('../resources/config.json')
        configure.load_config()
        auth = SymBotRSAAuth(configure)
        auth.authenticate()

        # Initialize SymBotClient with auth and configure objects
        bot_client = SymBotClient(auth, configure)

        # Initialize datafeed service
        datafeed_event_service = bot_client.get_datafeed_event_service()

        # Initialize listener objects and append them to datafeed_event_service
        # Datafeed_event_service polls the datafeed and the event listeners
        # respond to the respective types of events
        im_listener_test = IMListenerTestImp(bot_client)
        datafeed_event_service.add_im_listener(im_listener_test)
        room_listener_test = RoomListenerTestImp(bot_client)
        datafeed_event_service.add_room_listener(room_listener_test)

        # Create and read the datafeed
        print('Starting datafeed')
        datafeed_event_service.start_datafeed()


    if __name__ == "__main__":
        main()



### Run with Example main class (using certificates)
Once the certificates are provided and example listeners are implemented, let's run the bot by executing following command:

``python3 main_certificate.py``.

Change the paths to both **log**, and **config.json**
 - "sym_api_client_python/logs/example.log"
 - "sym_api_client_python/resources/config.json"

Example Main Class:

    import logging
    from sym_api_client_python.configure.configure import SymConfig
    from sym_api_client_python.auth.auth import Auth
    from sym_api_client_python.clients.sym_bot_client import SymBotClient
    from sym_api_client_python.listeners.im_listener_test_imp import \
            IMListenerTestImp
    from sym_api_client_python.listeners.room_listener_test_imp import \
            RoomListenerTestImp


    def configure_logging():
        logging.basicConfig(
                filename='./logs/example.log',
                format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                filemode='w', level=logging.DEBUG
        )
        logging.getLogger("urllib3").setLevel(logging.WARNING)


    def main():
        print('Python Client runs using Cert authentication')

        # Configure log
        configure_logging()

        # Cert Auth flow: pass path to certificate config.json file
        configure = SymConfig('../resources/config.json')
        configure.load_config()
        auth = Auth(configure)
        auth.authenticate()

        # Initialize SymBotClient with auth and configure objects
        bot_client = SymBotClient(auth, configure)

        # Initialize datafeed service
        datafeed_event_service = bot_client.get_datafeed_event_service()

        # Initialize listener objects and append them to datafeed_event_service
        # Datafeed_event_service polls the datafeed and the event listeners
        # respond to the respective types of events
        im_listener_test = IMListenerTestImp(bot_client)
        datafeed_event_service.add_im_listener(im_listener_test)
        room_listener_test = RoomListenerTestImp(bot_client)
        datafeed_event_service.add_room_listener(room_listener_test)

        # Create and read the datafeed
        print('Starting datafeed')
        datafeed_event_service.start_datafeed()


    if __name__ == "__main__":
        main()



### 4 - Converting JKS (Java Key Store) to Python truststore
The Python SDK truststore requires that your certificates be in a .pem file that is a collection of your certificates. You can convert JKSto a .p12 and then convert to .pem:

    keytool -importkeystore -srckeystore myapp.jks -destkeystore myapp.p12 -srcalias myapp-dev -srcstoretype jks -deststoretype pkcs12

    openssl pkcs12 -in myapp.p12 -out myapp.pem

### 5 - Interacting with the joke bot!
The joke bot shows how a Symphony chat bot application works. In general, a chat bot application keeps polling the [datafeed](https://rest-api.symphony.com/reference#read-messagesevents-stream-v4) API for new messages, then it sends the messages to listeners to handle,
depending on the message type.

To interact with the joke bot, try ``/bot joke``

### 6 - Using Elements:
Symphony Elements allow developers to include native, interactive UI components as part of an inbound message.  When a form Element is submitted, an event of type "SYMPHONYELEMENTSACTION" is sent across the datafeed.  

In order to handle Elements events, implement an ElementsActionListener and parse the JSON payload using the [SymElementsParser](https://github.com/SymphonyPlatformSolutions/symphony-api-client-python/blob/staging_elements/sym_api_client_python/processors/sym_elements_parser.py)

    from sym_api_client_python.listeners.elements_listener import ElementsActionListener
    from sym_api_client_python.processors.sym_elements_parser import SymElementsParser
    from .processors.action_processor import ActionProcessor

    class ElementsListenerTestImp(ElementsActionListener):

        def __init__(self, sym_bot_client):
            self.bot_client = sym_bot_client
            self.action_processor = ActionProcessor(self.bot_client)

        def on_elements_action(self, action):
            stream_type = self.bot_client.get_stream_client().stream_info_v2(SymElementsParser().get_stream_id(action))
            if stream_type['streamType']['type'] == 'ROOM':
                self.action_processor.process_room_action(action)
            elif stream_type['streamType']['type'] == 'IM':
                self.action_processor.process_im_action(action)

Use SymElementsParser class inside ActionProcessor:

    from sym_api_client_python.processors.message_formatter import MessageFormatter
    from sym_api_client_python.processors.sym_elements_parser import SymElementsParser

    class ActionProcessor:

        def __init__(self, bot_client):
            self.bot_client = bot_client

        def process_room_action(self, action):
            try:
                action_clicked = SymElementsParser().get_action(action)
                if action_clicked == 'submit':
                    #do something
            except:
                raise

Note: to send Elements forms as messages, please refer to [this documentation](https://developers.symphony.com/symphony-developer/docs/available-components) for valid messageML samples.  

Symphony REST API offer a range of capabilities for application to integrate, visit the [official documentation](https://rest-api.symphony.com/reference) for more information.

# Release Notes

## 0.1.25
- Handle JSONDecodeError
- fixed bug in search_users()

## 0.1.24
- Created Wall Post Listener
- Created Message Suppression Listener
- Fixed typos and comments

## 0.1.23
- Updated Error handling logic for Server Errors (500's) in datafeed_event_service.py

## 0.1.22
- Updated authentication retry logic for RSA based authentication
- User has a maximum of 5 times to reauthenticate.  If limit is reached, MaxRetryException is raised.
- SDK Recovers and creates new datafeed if reauthenticate succeeds.  

## 0.1.21
- Added Truststore support for RSA based authentication

## 0.1.20
- Updated examples

## 0.1.19
- Updated examples

## 0.1.18
- Added support for Elements actions coming across datafeed
- Added ElementsActionListener
- Added Processors, containing MessageFormatter, SymElementsParser, SymMessageParser. Provides helpers for accessing message and action payload information
- Added Templates folder, containing example templates for sending in Elements
- Added FormBuilder to programaticaly generate Elements forms
- Added examples for ExpenseBot using Elements
- Cleaned up typos and formatting


## 0.1.17
- Added AdminClient, SignalsClient, and ConnectionsClient
- Added docstrings to new methods
- Added clarification to README

## 0.1.16
- Re-released 0.1.15 due to merge issues

## 0.1.15
- Added podProxyURL, agentProxyURL, and keyManagerProxyURL as supported parameters in the config.json and config loader. If proxyURL is set, all of these proxies will be set to that URL. Otherwise, it will use the proxy address provided.
- merge to using the same method in configure.py to load RSA and Cert


## 0.1.13
- Rewrite clients to use python sessions.
- Moved over requests to sessions for consistent headers, proxies and truststore
- Fixed auth using sessionAuth for both sessionAuth and keyAuth
- Fixed functions that pre-populated payload to accept params
- Datafeed now utilizes session user to determine userId
- Update to ProxyURL. No longer prepend http://
- Changed config.json for Proxy to mirror Java SDK


## 0.1.12
- The updates in this release may break your existing bot implementation.  Please ensure you review and test against this client prior to deployment in Production.
- Extensively renamed client libraries to follow PEP8 Python standard.  Naming now follows "snake_case" convention.
- Bug fixes.
