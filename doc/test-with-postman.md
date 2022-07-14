# Test with Postman

This instruction introduce how to test with the [Post REST client tool](https://www.postman.com/product/rest-client/).

## Test ONVIF API

Before using `device-onvif-camera`, the user can verify the camera's functionality via ONVIF APIs, we provide the following collections for testing:
- Capabilities
- Auto Discovery
- Network Configuration
- System Function
- User Handling
- Metadata Configuration
- Video Streaming
- Video Encoder Configuration
- PTZ
- Event Handling
- Analytics

### Import the Postman collections

Download and import the following JSON files into Postman REST client tool:
- [onvif.postman_collection.json](./postman/onvif.postman_collection.json)
- [onvif.postman_environment.json](./postman/onvif.postman_environment.json)

### Set Up the Authentication for ONVIF security

Replace the following onvif `environment variable` on the Postman REST client.
- WS_USERNAME - The username for a certified user
- WS_NONCE -  A random, unique number generated by a client
- WS_UTC_TIME - The UtcTime when the request is made.
- WS_PASSWORD_DIGEST - a digest that is calculated according to an algorithm defined in the specification for WS-UsernameToken:
  Digest = B64ENCODE( SHA1( B64DECODE( Nonce ) + Date + Password ) )

#### How to generate the PasswordDigest?
According to the ONVIF spec and [programmer guide](https://www.onvif.org/wp-content/uploads/2016/12/ONVIF_WG-APG-Application_Programmers_Guide-1.pdf), the client needs to provide the password digest for WS-UsernameToken.
For example, we can generate the password digest in golang:
```go
package main

import (
	"crypto/sha1"
	"encoding/base64"
	"fmt"
)

func main() {
	nonce := "abcd"
	password := "Password1!"
	created := "2022-06-06T12:26:37.769698Z"
	passwordDigest := generatePasswordDigest(nonce, created, password)

	fmt.Println("Nonce:", nonce)
	fmt.Println("Created:", created)
	fmt.Println("PasswordDigest:", passwordDigest)
}

//Digest = B64ENCODE( SHA1( B64DECODE( Nonce ) + Date + Password ) )
func generatePasswordDigest(Nonce string, Created string, Password string) string {
	sDec, _ := base64.StdEncoding.DecodeString(Nonce)
	hasher := sha1.New()
	hasher.Write([]byte(string(sDec) + Created + Password))
	return base64.StdEncoding.EncodeToString(hasher.Sum(nil))
}
```
The runnable code: https://go.dev/play/p/ZnE2nZYorg9

### Set Up the API Endpoint
 
Generally, the device web service endpoint is http:/${address}:${port}/onvif/device_service, then we can use `GetCapabilities` ONVIF function to query other web service's endpoint:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope ...>
    <env:Body>
        <tds:GetCapabilitiesResponse>
            <tds:Capabilities>
                <tt:Device>
                    <tt:XAddr>http://192.168.12.123/onvif/device_service</tt:XAddr>
                    ...
                </tt:Device>
                <tt:Events>
                    <tt:XAddr>http://192.168.12.123/onvif/Events</tt:XAddr>
                    ...
                </tt:Events>
                ...
        </tds:GetCapabilitiesResponse>
    </env:Body>
</env:Envelope>
```

And we should replace the following onvif `environment variable` on the Postman REST client.
- DEVICE_ENDPOINT - device web service endpoint
- MEDIA_ENDPOINT - media web service endpoint
- EVENT_ENDPOINT - event web service endpoint
- PTZ_ENDPOINT - ptz web service endpoint

Then we can execute other ONVIF function via Postman REST client tool.

## Test device-onvif-camera API

After adding the device according to the [Getting Started Guide](./getting-started-guide.md), then we can import the following Postman collections for testing the APIs:
- [onvif.postman_collection.json](./postman/device-onvif-camera.postman_collection.json)
- [onvif.postman_environment.json](./postman/device-onvif-camera.postman_environment.json)