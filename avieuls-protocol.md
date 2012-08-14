Avieuls Protocol
================

Message Structure
-----------------
### Elements
* message type (1 byte)
* message body (max. 99 bytes, length is known to receiver)

### Remarks
The message is wrapped inside XBee-TX/RX Packets, so the length is known and a checksum is already checked. So we don't need to include that again. Every request message may be sent as a broadcast. The packets also always contain the sender.


Message Types
-------------
Format: `<message name> (<message type> <direction>)`

where

* direction 'in' denotes a message sent from the requestor to the device
* direction 'out' denotes a message sent from the device to the requestor.


### requestInfo (0x01, in)
Requests information about the device and its capabilities. The device responds with an announceServices message.

#### Body Structure
no data


### announceServices (0x02, out)
Sends information about the devices capabilities and services. See the chapter "device services".

#### Body Structure
* service 1 (6 bytes): Service provided by the device
    * service index (1 byte): index of the service in the device.
        * service type (4 bytes): type of the service as described in the chapter "device services"
        * service version (1 byte): version of the service
* service 2 (6 bytes): same structure
* ... more services, up to 16 total ...


### serviceUnknown (0x10, in)
The service requested is unknown to the device. Probably the service index was wrong.
#### Body Structure
* service (1 byte): the service-index that was requested, but does not exist


### serviceCall (0x20, in)
A service-dependent call. The call will not produce a result-message (in contrast to a serviceRequest).
#### Body Structure
* service (1 byte): index of the service in the announceServices message
* request type (2 bytes): type of the request (service dependent).
* payload (up to 96 bytes): dependent on the service request type


### serviceRequest (0x30, in)
A service-dependent request to the device. The request will be answered by the device with a serviceResponse with the same type. Overlapping service requests from the same requestor are not explicitly supported, the answers might be received in an arbitrary order.
#### Body Structure
* service (1 byte): index of the service in the announceServices message
* request type (2 bytes): type of the request (service dependent).
* payload (up to 96 bytes): dependent on the service request type


### serviceResponse (0x31, out)
Response to a serviceRequest. One call always results in exactly one response, either a serviceResponse or a serviceRequestUnknown.
#### Body Structure
* service (1 byte): index of the service in the announceServices message
* request type (2 bytes): type of the request this is the answer to.
* payload (up to 96 bytes): dependent on the service response type


### serviceRequestUnknown (0x3F, out)
The request type is unknown to or unsupported by the device/service.
#### Body Structure
* service (1 byte): index of the service in the announceService message
* request type (2 bytes): request type that is unknown or unsupported (see serviceRequest)


### serviceSubscribe (0x40, in)
Subscribe to messages a service (see servicePublish). Results in either a service subscriptionConfirm or a serviceSubscriptionUnknown message.
#### Body Structure
* service (1 byte): index of the service in the announceServices message
* subscription type (2 bytes): type of subscription (service dependent).
* payload (up to 96 bytes): dependent on the service


### serviceUnsubscribe (0x41, in)
Unsubscribe from a service (see serviceSubscribe).
#### Body Structure
* service (1 byte): index of the service in the announceServices message
* subscription type (2 bytes): type of subscription  (see serviceSubscribe).

        
### serviceSubscriptionConfirm (0x42, out)
Confirmation of a successful service subscription.
#### Body Structure
* service (1 byte): index of the service in the announceServices message
* subscription type (2 bytes): type of subscription  (see serviceSubscribe).


### serviceSubscriptionUnknown (0x4F, out)
The subscription could not be completed because the service does not support this subscription type.
#### Body Structure
* service (1 byte): index of the service in the announceServices message
* subscription type (2 bytes): type of subscription  (see serviceSubscribe).


### servicePublish (0x49, out)
Messages to subscribers (see serviceSubscribe).
#### Body Structure
* service (1 byte): index of the service in the announceServices message
* subscription type (2 bytes): type of the subscription (see serviceSubscribe).
* payload (up to 96 bytes): dependent on the service
