
                      Codettes Bootcamp 2022

              Jenielva shepherd




















10. Interface & Application Programming
10.1 Objectives
Wk 1
[x] Setup Dev Environment for ESP32 S2 (done)
[ ] Setup NodeJS Dev Environment on your PC (done)
[ ] Explain the HackOmation quadrant in relation to your final project.
[ ] Build UI mockups for your FInal Project and HTML Layout
Wk 2
[ ] Build HTML5 Chat app
* Draw mockup / layout (done)
* frame and add id’s to <div>’s(done)
* Style the page and (done)
* wire up the JS code and understand(done)
Wk 3
[ ] Build Chat app back-end NodeJS
* Build NodeJS server side to: 
* host your ChatApp (Express static HTML)
* Build / test API endpoints (for: users & messages)
Wk 4
[ ] Setup MongoDB datastore & connect via NodeJS
* Setup MongoDB datastore + mongoose ODM (Object-Document-Manager)
* Store and recall message data using an API (ex. request top 100 msg)
* Wire up MongoDB to API endpoints
* Update app-flow to use back-end for Users and “old” messages
Wk 5
[ ] Create data-bound widgets to display sensor data
 * On ESP32 add MQTT client + ArduinoJSON
 * Send Sensor data to MQTT server (as a JSON object)
 * Create a DataCard, a Gauge and a time Chart widget on Dashboard (use chat app)
 * Strategy on DataBinding and Widget updating (Last updated)
 * User Login/Pw (state persistence)
[ ] Add Screenshots and description of the process of creation. 
[ ] Describe the design & programming steps
[ ] Screenshots or video of your Prototype/app working
[ ] Describe any errors or problems with the process and how you fixed them. 
[ ] Include all the files you created for download. 



Week 1
Wk 1
[ ] Setup Dev Environment for ESP32 S2 (done)
[ ] Setup NodeJS Dev Environment on your PC(done)
[ ] Explain the HackOmation quadrant in relation to your final project.
[ ] Build UI mockups for your FInal Project and HTML Layout

Day 1.
10.2 Setup Dev Environment for ESP32 S2
Day 1 one was an online session. The teacher went over the slides and instructed the students to set up our  dev environment for the esp32 s2 using the instructions from slide 13.



After setting up the environment everyone had to test out the environment using the instruction in slide 17.


Note; i was unsuccessful in setting up my esp32 s2 environment ,so i could not do the test on day 1 of week 1.

Day 2 In lab session
With the help of one of my classmates I was able to set up my environment and do the test.
The first test was to create a Web Server (with websockets) using esp32 s2 in arduino.
How:
Step 1 was to connect a led to pin 15 of the  esp32 s2. And create a folder to save the code for the webserver. In this folder create another folder named data (the html index will be saved here.
Step 2 Install spiffs(if you don’t have it already) and the needed libraries in the Arduino IDE.
Step 3 Open a new Arduino ide sketch and paste this code
    #include <WiFi.h>
#include <SPIFFS.h>
#include <ESPAsyncWebServer.h>
#include <WebSocketsServer.h>

// Constants
const char *ssid = "ESP32-AP";
const char *password =  "LetMeInPlz";
const char *msg_toggle_led = "toggleLED";
const char *msg_get_led = "getLEDState";
const int dns_port = 53;
const int http_port = 80;
const int ws_port = 1337;
const int led_pin = 15;

// Globals
AsyncWebServer server(80);
WebSocketsServer webSocket = WebSocketsServer(1337);
char msg_buf[10];
int led_state = 0;

/***********************************************************
 * Functions
 */

// Callback: receiving any WebSocket message
void onWebSocketEvent(uint8_t client_num,
                      WStype_t type,
                      uint8_t * payload,
                      size_t length) {

  // Figure out the type of WebSocket event
  switch(type) {

    // Client has disconnected
    case WStype_DISCONNECTED:
      Serial.printf("[%u] Disconnected!\n", client_num);
      break;

    // New client has connected
    case WStype_CONNECTED:
      {
        IPAddress ip = webSocket.remoteIP(client_num);
        Serial.printf("[%u] Connection from ", client_num);
        Serial.println(ip.toString());
      }
      break;

    // Handle text messages from client
    case WStype_TEXT:

      // Print out raw message
      Serial.printf("[%u] Received text: %s\n", client_num, payload);

      // Toggle LED
      if ( strcmp((char *)payload, "toggleLED") == 0 ) {
        led_state = led_state ? 0 : 1;
        Serial.printf("Toggling LED to %u\n", led_state);
        digitalWrite(led_pin, led_state);

      // Report the state of the LED
      } else if ( strcmp((char *)payload, "getLEDState") == 0 ) {
        sprintf(msg_buf, "%d", led_state);
        Serial.printf("Sending to [%u]: %s\n", client_num, msg_buf);
        webSocket.sendTXT(client_num, msg_buf);

      // Message not recognized
      } else {
        Serial.println("[%u] Message not recognized");
      }
      break;

    // For everything else: do nothing
    case WStype_BIN:
    case WStype_ERROR:
    case WStype_FRAGMENT_TEXT_START:
    case WStype_FRAGMENT_BIN_START:
    case WStype_FRAGMENT:
    case WStype_FRAGMENT_FIN:
    default:
      break;
  }
}

// Callback: send homepage
void onIndexRequest(AsyncWebServerRequest *request) {
  IPAddress remote_ip = request->client()->remoteIP();
  Serial.println("[" + remote_ip.toString() +
                  "] HTTP GET request of " + request->url());
  request->send(SPIFFS, "/index.html", "text/html");
}

// Callback: send style sheet
void onCSSRequest(AsyncWebServerRequest *request) {
  IPAddress remote_ip = request->client()->remoteIP();
  Serial.println("[" + remote_ip.toString() +
                  "] HTTP GET request of " + request->url());
  request->send(SPIFFS, "/style.css", "text/css");
}

// Callback: send 404 if requested file does not exist
void onPageNotFound(AsyncWebServerRequest *request) {
  IPAddress remote_ip = request->client()->remoteIP();
  Serial.println("[" + remote_ip.toString() +
                  "] HTTP GET request of " + request->url());
  request->send(404, "text/plain", "Not found");
}

/***********************************************************
 * Main
 */

void setup() {
  // Init LED and turn off
  pinMode(led_pin, OUTPUT);
  digitalWrite(led_pin, LOW);

  // Start Serial port
  Serial.begin(115200);

  // Make sure we can read the file system
  if( !SPIFFS.begin()){
    Serial.println("Error mounting SPIFFS");
    while(1);
  }

  // Start access point
  WiFi.softAP(ssid, password);

  // Print our IP address
  Serial.println();
  Serial.println("AP running");
  Serial.print("My IP address: ");
  Serial.println(WiFi.softAPIP());

  // On HTTP request for root, provide index.html file
  server.on("/", HTTP_GET, onIndexRequest);

  // On HTTP request for style sheet, provide style.css
  server.on("/style.css", HTTP_GET, onCSSRequest);

  // Handle requests for pages that do not exist
  server.onNotFound(onPageNotFound);

  // Start web server
  server.begin();

  // Start WebSocket server and assign callback
  webSocket.begin();
  webSocket.onEvent(on WebSocket Event);
  
}

void loop() {
  
  // Look for and handle WebSocket data
  webSocket.loop();
}
      

SteP 4 Create an Index.html file and save it in the data folder. 
   Code:
<!DOCTYPE html>
<meta charset="utf-8" />
<title>WebSocket Test</title>

<script language="javascript" type="text/javascript">

var url = "ws://192.168.4.1:1337/";
var output;
var button;
var canvas;
var context;

// This is called when the page finishes loading
function init() {

    // Assign page elements to variables
    button = document.getElementById("toggleButton");
    output = document.getElementById("output");
    canvas = document.getElementById("led");
    
    // Draw circle in canvas
    context = canvas.getContext("2d");
    context.arc(25, 25, 15, 0, Math.PI * 2, false);
    context.lineWidth = 3;
    context.strokeStyle = "black";
    context.stroke();
    context.fillStyle = "black";
    context.fill();
    
    // Connect to WebSocket server
    wsConnect(url);
}

// Call this to connect to the WebSocket server
function wsConnect(url) {
    
    // Connect to WebSocket server
    websocket = new WebSocket(url);
    
    // Assign callbacks
    websocket.onopen = function(evt) { onOpen(evt) };
    websocket.onclose = function(evt) { onClose(evt) };
    websocket.onmessage = function(evt) { onMessage(evt) };
    websocket.onerror = function(evt) { onError(evt) };
}

// Called when a WebSocket connection is established with the server
function onOpen(evt) {

    // Log connection state
    console.log("Connected");
    
    // Enable button
    button.disabled = false;
    
    // Get the current state of the LED
    doSend("getLEDState");
}

// Called when the WebSocket connection is closed
function onClose(evt) {

    // Log disconnection state
    console.log("Disconnected");
    
    // Disable button
    button.disabled = true;
    
    // Try to reconnect after a few seconds
    setTimeout(function() { wsConnect(url) }, 2000);
}

// Called when a message is received from the server
function onMessage(evt) {

    // Print out our received message
    console.log("Received: " + evt.data);
    
    // Update circle graphic with LED state
    switch(evt.data) {
        case "0":
            console.log("LED is off");
            context.fillStyle = "black";
            context.fill();
            break;
        case "1":
            console.log("LED is on");
            context.fillStyle = "red";
            context.fill();
            break;
        default:
            break;
    }
}

// Called when a WebSocket error occurs
function onError(evt) {
    console.log("ERROR: " + evt.data);
}

// Sends a message to the server (and prints it to the console)
function doSend(message) {
    console.log("Sending: " + message);
    websocket.send(message);
}

// Called whenever the HTML button is pressed
function onPress() {
    doSend("toggleLED");
    doSend("getLEDState");
}

// Call the init function as soon as the page loads
window.addEventListener("load", init, false);

</script>

<h2>LED Control</h2>

<table>
    <tr>
        <td><button id="toggleButton" onclick="onPress()" disabled>Toggle LED</button></td>
        <td><canvas id="led" width="50" height="50"></canvas></td>
    </tr>
</table>


Step 5 opload the code 

The results:









The second test was to create a web server with multiple sliders to control LED brightness. 
How:
Step 1 was to connect the leds to the  esp32 s2. 
Step 2 Install spiffs(if you don’t have it already) and the needed libraries in the Arduino IDE.
Step 3 create a folder to organize your files
          
Arduino sketch that handles the web server;
index.html: to define the content of the web page;
sytle.css: to style the web page;
script.js: to program the behavior of the web page—handle what happens when you move the slider, send, receive and interpret the messages received via WebSocket protocol.
Step 4 copy the codes from https://randomnerdtutorials.com/esp32-web-server-websocket-sliders/
Step 5 connect the esp32 s2 , upload the code and open the html file.
 Results
































Week 2
Wk 2
[ ] Build HTML5 Chat app
* Draw mockup / layout (done)
* frame and add id’s to <div>’s(done)
* Style the page and (done)
* wire up the JS code and understand(done)
Day 1 online session
In this session we were introduced to mqtt lens ( A chrome plugin). The class was instructed to download the Mqtt extension and then connect to the mqttwebserver test.mosquitto.org.
After the lesson i used the link provided in the slide and went on w3school.com to learn more about HTML, Css and javascript.(i haven't learned everything yet)
We were also instructed to create a mockup of the chataplication we wanted to create and to create an html file with div and wire up the js code.
Addpicture of mockup








Day 2 inlab session
In the lab session we were instructed to continue working on our web chat application. I managed to create a web interface but I could not connect to the mqtt server. 
After some time everyone changed because test.mosquitto.org was giving problems.
I still was able to connect though and decided to skip that and work on the styling for my webchat.
This what the chat looked like:


This was styled using the css tutorial on w3schools

Week 3
Wk 3
[ ] Build Chat app back-end NodeJS
* Build NodeJS server side to: 
* host your ChatApp (Express static HTML)
* Build / test API endpoints (for: users & messages)

Day 1 online session
In this session we went over how far we were with the webchat application and had to share our scenes. I still was able to connect to the mqtt server now but was not able to send message. I kept getting an error in line 1 of my html.

I tried putting the js code and my styling (css)in a different file but I still kept getting the same error.
 
Day 2 in lab session
I finally discovered why I could not send messages. I made a few mistakes in the div class for the button. 
The code is supposed to look like this:
<div class="text" class="chatbox" id="chatlog">
      <input class="input"type="text" id="chatInput" id="sendMesageButton>
        <button class="sendbutton">
             <button onclick="sendMessageButton(getElementById('chatInput').value);getElementById('chatInput').value='';">Send</button>
			 </div> 
But i had this instead:
<div class="text" class="chatbox id=chatlog>
      <input class="inpt"type="text id="chatInput" id="sendMesageButton>
        <button class=sendbutton">
             <button onclick="sendMessageButton(getElementById('chatInput').value);getElementById('chatInput').value='';">Send</button>
			 </div> 

So after I finally got the code to work , I added more. I created another app.js file and added the new javascript there. With this javascript I was able to see who entered the chat and see the messages sent in the chat.



I also set up my Node Js environment. I had to do this twice because I didn't wait for everything to download the first time and shut down my terminal window.
So not everything was installed.








Week 4
Wk 4
[ ] Setup MongoDB datastore & connect via NodeJS
* Setup MongoDB datastore + mongoose ODM (Object-Document-Manager)
* Store and recall message data using an API (ex. request top 100 msg)
* Wire up MongoDB to API endpoints
* Update app-flow to use back-end for Users and “old” messages

I  installed mongodb and restyled my webchat to make it look a bit like whatsapp. (but I messed up the code and can't see or send  messages at the moment.)

I could not take part in class because my wifi was bad and I wasn't feeling well.
Same for day 2.

