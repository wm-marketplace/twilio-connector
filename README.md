
Connector is a Java based backend extension for WaveMaker applications. Connectors are built as Java modules & exposes java based SDK to interact with the connector implementation.
Each connector is built for a specific purpose and can be integrated with one of the external services. Connectors are imported & used in the WaveMaker application. Each connector runs on its own container thereby providing the ability to have it’s own version of the third party dependencies.


1. Connector is a java based extension which can be integrated with external services and reused in many Wavemaker applications.
1. Each connector can work as an SDK for an external system.
1. Connectors can be imported once in a WaveMaker application and used many times in the applications by creating multiple instances.
1. Connectors are executed in its own container in the WaveMaker application, as a result there are no dependency version conflict issues between connectors.

## About Twilio Connector

### Introduction
Twilio is a cloud based communication platform which performs communication functions using its API's. Twilio lets you receive SMS, MMS, WhatsApp, Voice messages or even respond to the messages and many more services.

This connector exposes api to send messages to and receive messages from Twilio using WaveMaker application.

### Basic Twilio Actions
Basic actions that you can do with this connector
1. Send SMS to device
2. Send MMS message to the device
3. Send WhatsApp message
4. Receive and respond to message
5. Implementing OTP

### Prerequisite
1. Twilio Account with SID, Token, Twilio Number and API Key.
2. Java 1.8 or above
3. Maven 3.1.0 or above
4. Any java editor such as Eclipse, Intelij..etc
5. Internet connection

### Build
You can build this connector using following command
```
mvn clean install -DskipTests
```

###Deploy
You can import connector dist/twilio-connector.zip artifact in WaveMaker studio application under **File Explorer** section. On after deploying twilio
-connector in the WaveMaker studio application, make sure to update connector properties in the profile properties such as SID, token, Twilio Number and API
 Key.
 
# Integrate Twilio Connector into WaveMaker App

## Twilio Service

Twilio is a cloud-based communication platform that performs communication functions using its APIs. Twilio lets you receive SMS, MMS, WhatsApp, and Voice messages, respond to messages, and access many more services.

This connector exposes APIs to send messages to and receive messages from Twilio using a WaveMaker application.

## Step 1: Importing the Twilio Connector to Project

1. Download the latest Twilio connector zip. [here](https://github.com/wm-marketplace/twilio-connector/releases).
2. Import the downloaded Twilio connector zip into your app using the **Import Resource** option to the **Connector** folder.
![image](https://github.com/user-attachments/assets/165a7bfb-e0d7-4e52-a222-b08f48ea7d4f)
![image](https://github.com/user-attachments/assets/91e62974-3938-4d9e-a9c8-f216007f73d7)

## Step 2: Configure Twilio Configurable Properties in Profiles

By default, externalized connector properties are added in the project profiles.
Connector externalized properties are prefixed with `connector.${connectorName}`:

```properties
connector.twilio-connector.default.twilio.account.SID=
connector.twilio-connector.default.twilio.account.authtoken=
connector.twilio-connector.default.twilio.account.phoneNumber=
connector.twilio-connector.default.twilio.verify.services.serviceId=
```

## Step 3: Integrating Twilio into Application

Autowire the Connector Service into the added JavaService.

### Import Statement:

```java
import com.wavemaker.connector.twilio.TwilioConnector;

@Autowired
private TwilioConnector twilioConnector;
```

### Send SMS to Device

Send a transactional message to the given number.

```java
public void sendSMSToDevice(String toPhoneNumber, String messageBody){
    twilioConnector.sendSMS(toPhoneNumber, messageBody);
}
```

### Send MMS Messages to Device

Send an MMS message to the phone number.

#### Import Statements:

```java
import java.net.URI;
import java.net.URISyntaxException;
import java.util.List;
import java.util.ArrayList;
```

#### Method:

```java
public void sendMMS(String toPhoneNumber, String messageBody){
    List<URI> mediaUris = new ArrayList<>();
    URI url = null;
    try {
        url = new URI("" ); // Add URL string here
    } catch (URISyntaxException e) {
        throw new RuntimeException(e);
    }
    mediaUris.add(url);
    twilioConnector.sendMMS(toPhoneNumber, messageBody, mediaUris);
}
```

### Send WhatsApp Messages

Send a message from Twilio to WhatsApp.

#### Import Statements:

```java
import java.net.URI;
import java.util.List;
```

#### Method:

```java
public void sendWhatsAppMessage(String toPhoneNumber, String messageBody){
    List<URI> mediaUris = new ArrayList<>();
    URI url = null;
    try {
        url = new URI("" ); // Add URL string here
    } catch (URISyntaxException e) {
        throw new RuntimeException(e);
    }
    mediaUris.add(url);
    twilioConnector.sendWhatsAppMessage(toPhoneNumber, messageBody, mediaUris);
}
```

## Twilio Responding to SMS/MMS/WhatsApp Message

Receive and respond back to the device from Twilio. In the example below, when Twilio receives a message from the user, it registers a complaint in the database and returns the complaint details to the user.

#### Import Statements:

```java
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import com.wavemaker.connector.twilio.TwilioMessageListener;
import com.wavemaker.connector.twilio.model.TwilioMessage;
```

#### Method:

```java
public void respondToMessage(HttpServletRequest request, HttpServletResponse response){
    twilioConnector.receiveAndRespondTwilioMessage(request, response, new TwilioMessageListener() {
        @Override
        public String onMessage(TwilioMessage twilioMessage) {
            String body = twilioMessage.getText();
            logger.info("TwilioRequest: " + twilioMessage.toString());
            
            // Business logic to register complaint
            TicketDetails details = new TicketDetails();
            details.setSubject(body);
            details.setFromNumber(twilioMessage.getFromNumber());
            details.setStatus("OPEN");
            TicketDetails createdDetails = ticketService.create(details);
            
            // Response message
            return "Hi, Your Complaint has been registered with TicketNo.: " + createdDetails.getTicketNo() +
                   " with Subject " + createdDetails.getSubject();
        }
    });
}
```

> **Tip:** In the case of SMS/MMS, this method API should be configured in the Twilio Account under **PhoneNumber** as a Webhook. For WhatsApp, configure this API in **WhatsApp Sandbox Settings** under **Programming PhoneNumber -> Settings** as a Webhook.

## Implementing OTP

### Send OTP to the Phone Number

This API simplifies adding user verification to your application. It supports codes sent via **VOICE, SMS, and EMAIL**. The return value helps verify if the given phone number is valid.

#### Import Statements:

```java
import com.wavemaker.connector.twilio.constant.Channel;
import com.wavemaker.connector.twilio.model.VerificationResult;
```

#### Method:

```java
public VerificationResult sendOTPCode(String phoneNumber){
    return twilioConnector.sendOTP(phoneNumber, Channel.SMS);
}
```

> **Note:** `Channel` refers to an Enum:
>
> ```java
> SMS("sms"), CALL("call"), EMAIL("email");
> ```

### Validate OTP Sent to Given Channel

This API validates the entered OTP.

```java
public Boolean validateOTP(String phoneNumber, String otpCode){
    VerificationResult result = twilioConnector.verifyOTP(phoneNumber, otpCode);
    return result.isValid();
}
```
