# Flic Hub integration

The Flic hub is configured to send HTTP requests to the Home Assistant [webhook](https://www.home-assistant.io/docs/automation/trigger/#webhook-trigger) endpoint. This is accomplished by installing a custom JavaScript module
onto the Flic hub using the [Flic Hub SDK](http://myhub.flic.io/static/tutorial/).

Currently (as of December 2020), the Flic Hub SDK is in beta, and you'll need to send an email to
[support@flic.io](mailto://support@flic.io) to request the SDK-enabled firmware. Once the request is
approved, press the reset button for exactly 3 seconds after which the LED will blink for 20-30 seconds.
The Flic Hub will then reboot with the new firmware.

Next, use the IDE at [myhub.flic.io](http://myhub.flic.io) to add a new module containing the custom
code to run on the hub.

The following code, in the `main.js` file in this module, sends an HTTP POST request to Home Assistant
using a webhook ID of `flic_<button-name><event-name>` after each button click, double-click, or hold
event:

```javascript
var buttonManager = require("buttons");
var http = require("http");

// IP address or hostname of home assistant
// Note that HA must be on the same local subnet as the Flic Hub!
var ha_address = "192.168.10.13";

// Port number for the HA web interface
var ha_port = "8123";

// Prefix to use for webhook IDs
var webhook_prefix = "flic_";

var baseUrl = "http://" + ha_address + ":" + ha_port + "/api/webhook/" + webhook_prefix;

buttonManager.on("buttonSingleOrDoubleClickOrHold", function(evt) {
  var button = buttonManager.getButton(evt.bdaddr);
  var clickType = evt.isSingleClick ? "click" : evt.isDoubleClick ? "doubleclick" : "hold";
  var name = button.name.toLowerCase().replace(" ", "-");
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
          console.log("Error sending http request for " + button.name + ":"
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
