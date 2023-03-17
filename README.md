# ESP32_Websocket_Audiostream
Using Websockets to stream Audio waveforms between two ESP32's to Serial output. 

ESP32 Server to ESP32 Client/modem to Serial

Hello and welcome to another excellent challenge in extending a tutorial! I’ll be working on this tutorial on getting two ESP32’s to talk to each other via websocket. After reviewing my past moves, I’ve remembered that I’m trying to get to an onboard baby boot sensor suite to stream data to an ML training/testing platform, in order to classify a system of inputs. So, today the goal is to get two ESP’s to talk to each other via websocket, and the receiver to post all data in a live stream to serial…as though the entire dataset were coming straight into Serial and not over Wifi. This is to simplify passing the data to RStudio, in the event that RStudio doesn’t want to talk over websocket so easily. I’ve already successfully gotten RStudio to send and receive data from Arduino Serial in a past exercise, so if this stage works, I’ll be able to combine two exercises into a datastream, including RStudio as an analysis engine on the data to monitor effectiveness of the threshold detection algorithm. 

Here’s the circuit schematic from the tutorial:

I’ll be editing this in a little bit to use a sensor with a faster sampling rate on the server side, just to test the speed limitations of the setup, but for now we’ll take it as is. 

First stage is to grab the codebase and compile it to make sure it works. And, after some checking the cut and paste (the server side didn’t have the whole codebase included in one spot) and reorganizing, and putting the webpage into PROGMEM (a trick I learned from a past exercise), both the server and client compile. Buuut…I don’t want to send to an OLED, and I don’t want to handle a web page, AAAND I don’t want to use the DHT sensor. I would rather use a sensor with a higher sample rate like a microphone, and have the client post directly to serial. So, time for some refactoring. Here’s the checklist:

Get rid of the DHT code and plug in the microphone
Get rid of the webpage, because we’re sending straight to an ESP32 client only
Get rid of the OLED and print straight to Serial

Let’s see how that goes. 

Ok, so the one thing I didn’t do was get rid of the website, because it doesn’t seem to interrupt the function of the ESP client, and it will be more handy to let that persist for later debugging via browser console if the ESP client doesn’t cooperate. Everything else sucessfully refactored, so now it’s just a matter of hooking up the sensors and seeing if the compiled code flops a real run. 

A couple things I learned from this bit of code is the use of the json serialize and deserialize. I was having trouble in past exercises chunking and de-chunking the code, especially with the async Highcharts plotter. I was struggling before getting the sample rate separated from the refresh rate, and this seems like the key to that…especially the deserialize and plot separately bit. I’m going to go back and comment that into the other codes and exercises, because I did get stuck there before. 

That code I’d stored as A0_Stream_AP_Grapher or ESP_Stream_Graph_Motion

I’ve also seen that the ESP32 socket server can handle multiple clients, some on a web page and some without…which means I can combine two prior goals; I can send copies of the data to RStudio and Highcharts simultaneously, so that one can do live plotting and the other can do analysis. That would be handy. 

In any case, here’s the code that compiled:
ESP_to_ESP_Socket_Server:

ESP_to_ESP_Socket_Client:

The modified circuits look like this, not including power sources and the Serial connection to the client:

Now it’s time to upload the code and see if these guys talk!

So first thing I noticed is that both the server and the client know how to connect to Wifi, but that they’re not sharing data. I should double back and check what I cut out…maybe I took too much off the top. 

The webpage from the server just shows this:

But that’s expected because I commented in the stored webpage that it doesn’t know how to access the correct datasource, and that it probably needs to be refactored, but at least the webpage is loading, so it does know that there’s data coming in. And the Highcharts script is running, so that should be the only error on the webpage version, which is good because that does simplify future projects. 

I also found out that I’d wired the server’s sensor wrong, and had to do some hunting to find where A0 actually was (it’s in SVN), so I’ll update the schematic image when I get to the end of the project and know for sure where I want that input pin. It is reading and printing to serial, though, so it *should* be broadcasting via websocket as well. 

So, now I need to read back through the tutorial and find out where I’m missing a connection between the two. 

And, another read through and I just noticed that there was an extra digit in the static IP address I’d fed the client to tell it where to find the server, so hopefully that was the key problem. 

Loading again and we get:

Which isn’t exactly what I wanted, but hey, it’s a signal that’s printing! 

At this point, I’ve figured out websocket data transfer to Serial…and even though it’s a flatline, it’s the same flatline that was being transmitted, so the fact that there’s a sensor flaw doesn’t impact the accomplishment of getting the websocket to work!

And I found out that the signal was flatlined for two reasons; I plugged it into the wrong pin again, and the battery was dying. So…here’s the real deal. 

And here’s me laying down a phat beat. 

So…that’s it for the ESP to ESP to Serial printer! I’m all set now to work on getting this connected to RStudio, but that’s another goal post for another day. 

Oh, and here’s the working codes:
