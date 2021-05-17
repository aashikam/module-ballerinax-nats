## Overview

This module provides the capability to send and receive messages by connecting to the NATS server.

NATS messaging enables the communication of data that is segmented into messages among computer applications and services. Data is encoded and framed as a message and sent by a publisher. The message is received, decoded, and processed by one or more subscribers. NATS makes it easy for programs to communicate across different environments, languages, cloud providers and on-premise systems. Clients connect to the NATS system, usually via a single URL, and then subscribe or publish messages to a subject.

### Basic Usage

#### Setting up the connection

First step is setting up the connection with the NATS Basic server. The following ways can be used to connect to a
NATS Basic server.

1. Connect to a server using the default URL
```ballerina
nats:Client natsClient = check new(nats:DEFAULT_URL);
```

2. Connect to a server using the URL
```ballerina
nats:Client natsClient = check new("nats://serverone:4222");
```

3. Connect to one or more servers with custom configurations
```ballerina
nats:ConnectionConfiguration config = {
    connectionName: "my-nats",
    noEcho: true
};
nats:Client natsClient = check new(["nats://serverone:4222",  "nats://servertwo:4222"],  config);
```

#### Publishing messages

##### Publishing messages to the NATS basic server

Once connected, publishing is accomplished via one of the below three methods.

1. Publish with the subject, and the message content.
```ballerina
string message = "hello world";
nats:Error? result = 
    natsClient->publishMessage({ content: message.toBytes(), subject: "demo.nats.basic"});
```

2. Publish as a request that expects a reply.
```ballerina
string message = "hello world";
nats:Message|nats:Error reqReply = 
    natsClient->requestMessage({ content: message.toBytes(), subject: "demo.nats.basic"}, 5);
```

3. Publish messages with a replyTo subject
```ballerina
string message = "hello world";
nats:Error? result = natsClient->publish({ content: message.toBytes(), subject: "demo.nats.basic",
                                                    replyTo: "demo.reply" });
```

#### Listening to incoming messages

##### Listening to messages from a NATS server

1. Listen to incoming messages with `onMessage` remote method
```ballerina
// Binds the consumer to listen to the messages published to the 'demo.example.*' subject
@nats:ServiceConfig {
    subject: "demo.example.*"
}
service nats:Service on new nats:Listener(nats:DEFAULT_URL) {

    remote function onMessage(nats:Message message) {
    }
}
```

2. Listen to incoming messages and reply directly with `onRequest` remote method
```ballerina
// Binds the consumer to listen to the messages published to the 'demo.example.*' subject
@nats:ServiceConfig {
    subject: "demo.example.*"
}
service nats:Service on new nats:Listener(nats:DEFAULT_URL) {

    // The returned message will be published to the replyTo subject of the consumed message
    remote function onRequest(nats:Message message) returns string? {
        return "Reply Message";
    }
}
```

### Advanced Usage

#### Setting up TLS

The Ballerina NATS module allows the use TLS in communication. This setting expects a secure socket to be
set in the connection configuration as shown below.

##### Configuring TLS in `nats:Listener`
```ballerina
nats:SecureSocket secured = {
    cert: {
        path: "<path>/truststore.p12",
        password: "password"
    },
    key: {
        path: "<path>/keystore.p12",
        password: "password"
    }
};
nats:Listener natsListener = check new("nats://serverone:4222", secureSocket = secured);
```

##### Configuring TLS in `nats:Client`
```ballerina
nats:SecureSocket secured = {
    cert: {
        path: "<path>/truststore.p12",
        password: "password"
    },
    key: {
        path: "<path>/keystore.p12",
        password: "password"
    }
};
nats:Client natsClient = check new("nats://serverone:4222", secureSocket = secured);
```