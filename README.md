# IoT_telegram_nodeMCU
By Luka Spelberg 

## Introduction
In this guide you will learn how to control your nodeMCU (ESP8266) via telegram. In this step by step guide, we will cover all the potential errors, and make sure this will be a smooth process. so lets dive right in. 

## Parts
For this guide you will need the following parts: 
1. an ESP8266 board.
2. A LEDstrip
3. [Arduino IDE](https://www.arduino.cc/en/software)

## Step 1: Telegram setup. 
Before we start with the coding, we will first make sure everything in telegram is set-up

First you'll need to download telegram and make an account. Its available in the app store & google play store. 

<img width="250" alt="image" src="https://github.com/user-attachments/assets/3a1dcaa1-8d0a-4051-a5d5-885d19f91a2e" /> 

Once you are in the app you should navigate to the searchbar. Type "botfather" and there should pop-up a bot with over 3 million users.

<img width="250" alt="image" src="https://github.com/user-attachments/assets/4aa73143-e21d-4453-a6d0-2978d8780ad2" />

Open the bot and start a chat. type the command /newbot to create your very own bot. it will ask you to name it, the surname has to be unique so dont panic if it does not accept your name on the first try.

<img width="250" alt="image" src="https://github.com/user-attachments/assets/1793a719-ce04-4a7f-aaee-750d1d14b9b7" />

Make sure to copy the token that it sends with the confirmation message. this will be important later on.

Next, we have to gather your telegram chatID. You can get this by once again navigating to the searchbar. type "myidbot" and click on the first result. 

<img width="250" alt="image" src="https://github.com/user-attachments/assets/e9435950-aee1-42a9-addc-e825fafe3493" />

Start the conversation, and type "/getid" copy the number you get and save it for later. 

<img width="250" alt="image" src="https://github.com/user-attachments/assets/c9b1e6f9-8782-49ca-8c5a-523f3a4b64f3" />

Nice! now we you got all the telegram set up done.

## Step 2: arduino setup. 
Now that we got telegram done, we can move on to the technical stuff. 

Lets first make sure to have your pins in the correct place. your LEDstrip has 3 pins: a yellow one, a red one, and a black one. Place the yellow one on D1, the red one on 3V, and the black one on G.

<img width="500" alt="image" src="https://github.com/user-attachments/assets/a0731280-52d7-42e3-a33f-9091c6f67985" />

Now for arduino IDE we also need a few libraries. 
First, open arduinoIDE and look at the sidebar. Click on the third icon to open your library manager.

<img width="150" alt="image" src="https://github.com/user-attachments/assets/6befedc4-78ae-4d35-a538-f183124707ef" />
 
First install the adafruit neopixel library. It will probably be the third option you see. Make sure to check if you are downloading the correct one. If you are getting errors later on make sure you have not installed the DMA version by accident. We need the normal one. 

<img width="250" alt="image" src="https://github.com/user-attachments/assets/df52ae1f-5894-41ae-8af6-4fd7fd65a40a" />

Next, install the Universal telegram bot. This is still in the same library manager that we used for the adafruit neopixel library.

<img width="250" alt="image" src="https://github.com/user-attachments/assets/0624557a-c35b-49c5-85ce-dc79c30d3ad5" />
 
When you install it, it will ask you if you want to install their dependencies as well. Click on "install all" as we will need the dependency as well.

<img width="500" alt="image" src="https://github.com/user-attachments/assets/6c143db0-8d01-4ce1-86cc-0aad35abbead" />

Now that we have done that its time to make ArduinoIDE recognize your board.

You can do this easily by going to tools > board > ESP8266 > NodeMCU 1.0 (ESP-12E Module). You may need to scroll down in the last dropdown menu to find it.

<img width="500" alt="image" src="https://github.com/user-attachments/assets/c53eaea4-6a36-4e2e-be96-14cd533cfa55" />

Also make sure to select the correct port. Again, Tools > port > select your port. 

Congratulations! you are ready to start with the code now!

## Step 3: the code. 

To start click file > new sketch.  
Paste this code:  

```cpp
#ifdef ESP32
  #include <WiFi.h>
#else
  #include <ESP8266WiFi.h>
#endif
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>
#include <ArduinoJson.h>
#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
 #include <avr/power.h>
#endif

// Replace with your network credentials
const char* ssid = "your wifi username";
const char* password = "your wifi password";

// Initialize Telegram BOT
#define BOTtoken "your bot token that you received from the botfather"
#define CHAT_ID "your chattoken that you received from my id bot"

#define PIN D1
#define NUMPIXELS 12

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
#define DELAYVAL 500

#ifdef ESP8266
  X509List cert(TELEGRAM_CERTIFICATE_ROOT);
#endif

WiFiClientSecure client;
UniversalTelegramBot bot(BOTtoken, client);

int botRequestDelay = 1000;
unsigned long lastTimeBotRan;

const int ledPin = 2;
bool ledState = HIGH;

// Handle what happens when you receive new messages
void handleNewMessages(int numNewMessages) {
  for (int i=0; i<numNewMessages; i++) {
    String chat_id = String(bot.messages[i].chat_id);
    if (chat_id != CHAT_ID){
      bot.sendMessage(chat_id, "Unauthorized user", "");
      continue;
    }
    String text = bot.messages[i].text;
    String from_name = bot.messages[i].from_name;

    if (text == "/start") {
      String welcome = "Welcome, " + from_name + ".\n";
      welcome += "/led_on to turn GPIO ON \n";
      welcome += "/led_off to turn GPIO OFF \n";
      welcome += "/state to request current GPIO state \n";
      bot.sendMessage(chat_id, welcome, "");
    }

    if (text == "/led_on") {
      bot.sendMessage(chat_id, "LED state set to ON", "");
      pixels.clear(); 
      for(int i=0; i<NUMPIXELS; i++) {
        pixels.setPixelColor(i, pixels.Color(0, 150, 0));
        pixels.show();
        delay(DELAYVAL);
        ledState = LOW;
      }
    }
    
    if (text == "/led_off") {
      bot.sendMessage(chat_id, "LED state set to OFF", "");
      pixels.clear(); 
      pixels.show(); 
      ledState = HIGH;
    }
    
    if (text == "/state") {
      if (digitalRead(ledPin)){
        bot.sendMessage(chat_id, "LED is ON", "");
      }
      else{
        bot.sendMessage(chat_id, "LED is OFF", "");
      }
    }
  }
}

void setup() {
  Serial.begin(115200);

  #if defined(__AVR_ATtiny85__) && (F_CPU == 16000000)
  clock_prescale_set(clock_div_1);
  #endif

  pixels.begin();

  #ifdef ESP8266
    configTime(0, 0, "pool.ntp.org");
    client.setTrustAnchors(&cert);
  #endif

  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, ledState);
  
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  #ifdef ESP32
    client.setCACert(TELEGRAM_CERTIFICATE_ROOT);
  #endif
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi..");
  }
  Serial.println(WiFi.localIP());
}

void loop() {
  if (millis() > lastTimeBotRan + botRequestDelay)  {
    int numNewMessages = bot.getUpdates(bot.last_message_received + 1);

    while(numNewMessages) {
      handleNewMessages(numNewMessages);
      numNewMessages = bot.getUpdates(bot.last_message_received + 1);
    }
    lastTimeBotRan = millis();
  }
}
 ```

There are a few important things to understand here:

```cpp

// Replace with your network credentials
const char* ssid = "your wifi username;
const char* password = "your wifi password";

// Initialize Telegram BOT
#define BOTtoken "your bot token that you received from the botfather"  // your Bot Token (Get from Botfather)

// Use @myidbot to find out the chat ID of an individual or a group
// Also note that you need to click "start" on a bot before it can
// message you
#define CHAT_ID "your chattoken that you received from my id bot"

```
Input your wifi/hotspot password and username.
⚠️ Heads up: the NodeMCU only supports 2.4GHz Wi-Fi. If you’re using an iPhone hotspot, enable compatibility mode.

Once you have edited the code with your tokens, upload it to your NodeMCU.

<img width="350" alt="image" src="https://github.com/user-attachments/assets/00c42570-41f6-4640-be6d-cef342ad11ff" />

Open your bot on telegram using the link from Botfather.

<img width="250" alt="image" src="https://github.com/user-attachments/assets/ed354972-0bf5-4f7c-ba12-f779c738d4fd" />

Now you should be able to toggle the ledstrip on and off! 🎉

I hope this guide was clear, and you can remotely toggle your nodeMCU with telegram without any problems!

<img width="700" alt="image" src="https://github.com/user-attachments/assets/d4119663-1259-4a45-a5ee-4f293d28f865" />
