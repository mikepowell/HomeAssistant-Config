# Flic Hub integration

## Update 2/24/2021

I'm no longer using the Flic integration described here, the range was just not long enough to support
my three buttons with a single hub and I couldn't justify another $100+ for a second hub. I'm using
Aqara Zigbee buttons instead, they look and feel cheaper than the Flic buttons but don't require a hub
and the range is good.

---

The Flic hub is configured to send HTTP requests to the Home Assistant [webhook][ha-webhook] endpoint.
This is accomplished by installing a custom JavaScript module onto the Flic hub using the
[Flic Hub SDK][flic-sdk].

Currently (as of December 2020), the Flic Hub SDK is in beta, and you'll need to send an email to
[support@flic.io][flic-support] to request the SDK-enabled firmware. Once the request is
approved, press the reset button for exactly 3 seconds after which the LED will blink for 20-30 seconds.
The Flic Hub will then reboot with the new firmware.

Next, use the IDE at [myhub.flic.io][flic-ide] to add a new module containing the custom
code to run on the hub.

The following code, in the `main.js` file in this module, sends an HTTP POST request to Home Assistant
using a webhook ID of `flic_<button-name><event-name>` after each button click, double-click, or hold
event:

```javascript
var buttonManager = require("buttons");
var http = require("http");

// IP address or hostname of home assistant
var ha_address = "192.168.10.13";

// Port number for the HA web interface
var ha_port = "8123";

// Prefix to use for webhook IDs
var webhook_prefix = "flic_";

var baseUrl = "http://" + ha_address + ":" + ha_port + "/api/webhook/" + webhook_prefix;

buttonManager.on("buttonSingleOrDoubleClickOrHold", function(evt) {
  var button = buttonManager.getButton(evt.bdaddr);
  var buttonName = button.name ? button.name : "unnamed-" + button.serialNumber
  var clickType = evt.isSingleClick ? "click" : evt.isDoubleClick ? "doubleclick" : "hold";

  var name = buttonName.toLowerCase().replace(" ", "-");
  var url = baseUrl + name + "_" + clickType;

  http.makeRequest(
    {
      url: url,
      method: "POST",
      headers: {"Content-Type": "application/json"},
      content: JSON.stringify({"serial-number": button.serialNumber, "click-type": clickType})
    },
    function(err, res) {
      if (!err) {
          console.log("HTTP request: " + url);
      }
      else {
          console.log("Error sending http request for " + buttonName + ":"
          + button.serialNumber + " - status " + res.statusCode );
      }
    }
  );
});

console.log("Module running");
console.log("Webhook endpoint: " + baseUrl + "<button-name><event-name>");

```

## More information

- [Flic Hub SDK tutorial](http://myhub.flic.io/static/tutorial/)
- [Full SDK documentation](https://docs.google.com/document/d/e/2PACX-1vTAyIl82edoPl_mTE28uWraTy5nBOdwnAEmGvvS7DsEnc_0ZvpLQhzsf38olE9nsDUgaa8fE-u0foWe/pub)
- [Super helpful blog post](https://ecozonnewoning.nl/2020/10/flic2-buttons-koppelen-aan-homeassistant/)
  (in Dutch)
- [Home Assistant webhook triggers](https://www.home-assistant.io/docs/automation/trigger/#webhook-trigger)

[ha-webhook]: https://www.home-assistant.io/docs/automation/trigger/#webhook-trigger
[flic-sdk]: http://myhub.flic.io/static/tutorial/
[flic-support]: mailto://support@flic.io
[flic-ide]: http://myhub.flic.io
