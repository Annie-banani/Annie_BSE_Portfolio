# Alarm Clock Mat
Are you like me, who slaps the alarm clock and goes back to sleep in the morning? I built the alarm clock door mat to get rid of that bad habit. 

```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | Grade 6 |
|:--:|:--:|:--:|:--:|
| Annie W | Harker | Electrical Engineering | Incoming 6th grader

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
For my final milestone, I switched the Arduino out for an Arduino Nano ESP32, and using that, I added a website in which I could edit what time I wanted the alarm to sound. 
My biggest challenges 
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- I made a weight sensor, which is more of a giant button by cutting out some cardboard, and putting some foil on and taping it together. When you plug it in, the buzzer starts buzzing, and when you put force onto the giant button/weight sensor, the buzzer stops buzzing. I couldn't figure out why my buzzer wasn't working initially, but when I carefully checked the wiring it was because I forgot to add a wire that goes from my giant button to pin 2.
# First Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/GXYiP8zMwdg?si=pSS0th8kuIUSS5T5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your first milestone, describe what your project is and how you plan to build it. You can include:
My project is the alarm clock mat. For milestone one, I designed the circuit on tinkercad and wrote the code. I had and issue when I didn't know what to put in the seconds, so I searched it up, and apparently if you don't put any number in then it'll go on forever. But then I didn't want it to go on forever, so I put in "noTone" into the if loop, meaning if the button is pressed, then make it go quiet.  Then I made the circuit in real life, and put the code in. I plan on making the physical weight sensor next, using cardboard, tape, and aluminum foil. Then I'll make a website to control what time the buzzer sounds

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 

```c++

#include <WiFi.h>
#include <time.h>

const char* ssid = "Bluestamps-J9";
const char* password = "j9bestroom";

WiFiServer server(80);

int alarmHour = 0;
int alarmMinute = 0;
bool isPM = false;
bool alarmTriggered = false;

int buzzerPin = A2;
int buttonPin = A1;

//button logic
int wasPressed = 0; //prev value for buzzer
int alarmHour24 = 0;
int currentHour24;
int currentMinute;
int dispH;
bool pm;

int buttonState = 0;
int start;
int end;

time_t now;
struct tm *timeinfo;

bool currentlyPM;

void setup() {
  Serial.begin(115200);
  pinMode(buzzerPin, OUTPUT);
  pinMode(buttonPin, INPUT_PULLDOWN);
  digitalWrite(buzzerPin, LOW);

  delay(10);

  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  configTime(-28800, 3600, "pool.ntp.org");

  server.begin();
}

void loop() {
  WiFiClient client = server.available();

  if (client) {
    Serial.println("New Client.");
    String currentLine = "";
    String requestLine = "";

    while (client.connected()) {
      if (client.available()) {
        char c = client.read();
        Serial.write(c);
        if (c == '\n') {
          if (currentLine.length() == 0) {
            if (requestLine.indexOf("hour=") >= 0) {
              start = requestLine.indexOf("hour=") + 5;
              end = requestLine.indexOf("&", start);
              if (end == -1) end = requestLine.indexOf(" ", start);
              alarmHour = requestLine.substring(start, end).toInt();
              alarmTriggered = false;
            }
            if (requestLine.indexOf("minute=") >= 0) {
              start = requestLine.indexOf("minute=") + 7;
              end = requestLine.indexOf("&", start);
              if (end == -1) end = requestLine.indexOf(" ", start);
              alarmMinute = requestLine.substring(start, end).toInt();
              alarmTriggered = false;
            }
            if (requestLine.indexOf("period=") >= 0) {
              start = requestLine.indexOf("period=") + 7;
              end = requestLine.indexOf("&", start);
              if (end == -1) end = requestLine.indexOf(" ", start);
              String val = requestLine.substring(start, end);
              if (val == "PM") isPM = true;
              else isPM = false;
              alarmTriggered = false;
            }

            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println();

            client.println("<!DOCTYPE html><html><head><title>Alarm Clock</title></head><body>");

            client.print("<h2>Alarm set for: ");
            client.print(alarmHour);
            client.print(":");
            if (alarmMinute < 10) client.print("0");
            client.print(alarmMinute);
            client.print(" ");
            if (isPM) client.println("PM</h2>"); else client.println("AM</h2>");

            time(&now);
            timeinfo = localtime(&now);
            client.print("<h2>Current time: ");
            dispH = timeinfo->tm_hour;
            currentlyPM = dispH >= 12;
            if (dispH == 0) dispH = 12;
            else if (dispH > 12) dispH = dispH - 12;
            client.print(dispH);
            client.print(":");
            if (timeinfo->tm_min < 10) client.print("0");
            client.print(timeinfo->tm_min);
            client.print(" ");
            if (currentlyPM) client.println("PM</h2>"); else client.println("AM</h2>");

            client.println("<h1>Hour</h1>");
            client.println("<form action=\"/\" method=\"GET\">");
            client.println("<select name=\"hour\">");
            for (int i = 1; i <= 12; i++) {
              client.print("<option value=\"");
              client.print(i);
              client.print("\"");
              if (i == alarmHour) client.print(" selected");
              client.print(">");
              client.print(i);
              client.println("</option>");
            }
            client.println("</select>");

            client.println("<select name=\"minute\">");
            for (int i = 0; i < 60; i += 1) {
              client.print("<option value=\"");
              client.print(i);
              client.print("\"");
              if (i == alarmMinute) client.print(" selected");
              client.print(">");
              if (i < 10) client.print("0");
              client.print(i);
              client.println("</option>");
            }
            client.println("</select>");

            client.println("<select name=\"period\">");
            client.print("<option value=\"AM\"");
            if (!isPM) client.print(" selected");
            client.println(">AM</option>");
            client.print("<option value=\"PM\"");
            if (isPM) client.print(" selected");
            client.println(">PM</option>");
            client.println("</select>");

            client.println("<input type=\"submit\" value=\"Set Alarm\">");
            client.println("</form>");

            if (alarmTriggered) {
              client.println("<h1 style=\"color:red;\">ALARM IS GOING OFF!</h1>");
            }

            client.println("</body></html>");
            client.println();
            break;
          } else {
            if (requestLine == "") requestLine = currentLine;
            currentLine = "";
          }
        } else if (c != '\r') {
          currentLine += c;
        }
      Serial.println("Alarm set..");
      }
    }

    client.stop();
    Serial.println("Client Disconnected.");
  }



  time(&now);
  timeinfo = localtime(&now);

  currentHour24 = timeinfo->tm_hour;
  currentMinute = timeinfo->tm_min;



  Serial.print("Current time: ");
  dispH = currentHour24;
  pm = dispH >= 12;
  if (dispH == 0) dispH = 12;
  else if (dispH > 12) dispH = dispH - 12;
  Serial.print(dispH);
  Serial.print(":");
  if (currentMinute < 10) Serial.print("0");
  Serial.print(currentMinute);
  Serial.print(":");
  if (timeinfo->tm_sec < 10) Serial.print("0");
  Serial.print(timeinfo->tm_sec);
  Serial.println(pm ? " PM" : " AM");

  alarmHour24 = alarmHour;
  if (isPM && alarmHour != 12) alarmHour24 = alarmHour + 12;
  if (!isPM && alarmHour == 12) alarmHour24 = 0;
  // if (alarmHour == -1) alarmHour24 = -1;

  // Serial.print("Alarm: ");
  // Serial.print(alarmHour24);
  // Serial.print(":");
  // Serial.print(alarmMinute);
  // Serial.print(" | Current: ");
  // Serial.print(currentHour24);
  // Serial.print(":");
  // Serial.println(currentMinute);

  buttonState = digitalRead(buttonPin);
  if(buttonState) {wasPressed = 1;} //change the value of wasPressed
  Serial.print("Button: ");
  Serial.println(buttonState);


  if (alarmHour24 == currentHour24 && alarmMinute == currentMinute && !alarmTriggered) {
    tone(buzzerPin, 523);
    alarmTriggered = true;
    Serial.println("ALARM GOING OFF!");
  } else if (alarmTriggered && wasPressed){
    noTone(buzzerPin);
    alarmTriggered = false;
    Serial.println("turning alarm off!");
    alarmHour24 = -1;
    alarmHour = -1;
  }

  if (alarmTriggered && currentMinute != alarmMinute) {
    noTone(buzzerPin);
    alarmTriggered = false;
    wasPressed = 0;
  }

  if (alarmTriggered && buttonState) {
    noTone(buzzerPin);
    alarmTriggered = false;
    Serial.println("Alarm dismissed!");
  }

  if(currentMinute != alarmMinute) wasPressed = 0;

  delay(400);
}
```

# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
