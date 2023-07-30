# whisper-websocket-server

This is a small demonstration Python websockets program to run on your own server that will accept audio input from a client Android phone and transcribe it to text using whisper.cpp voice recognition, and return the text string results to the phone for insertion into text message or email or use as command or a web search.

An app on the client Android phones called Kõnele acts as the input service which sends the audio to this server. Kõnele acts as an Android input device, like a keyboard.

Basically, together the system acts like Google's voice input service or voice search, but you self host it so your data stays under your control. It works great for de-Googled open Android phones like LineageOS or Graphene.

Though written as a simple test, this works so well for me that it is part of my regular phone usage!

This program uses these other software systems:
- [whisper.cpp](https://github.com/ggerganov/whisper.cpp/) for audio to voice transcription, install to server with git clone
- [Kõnele](https://f-droid.org/en/packages/ee.ioc.phon.android.speak/) client Android app; install to phone via f-droid repo is recommended
- [wsocket](https://github.com/ksenginew/WSocket) Python websocket library, install to server with pip
- sox audio processing command line tools, install to server with OS tools (apt etc.)

The setup is pretty simple, with a few things on the server side and a few on the client side (Android phone).

---
## Demo video

Both recogntions here were done using the "tiny" model.


https://user-images.githubusercontent.com/10035429/219758618-c6c05a27-059d-427a-9e13-550fedbc281a.mp4


https://user-images.githubusercontent.com/10035429/219812393-9a4adbc6-09b3-41ec-aa3a-dfb84b60730e.mp4


---
## Server side setup (Python websocket server with sox+whisper.cpp)

On your server machine where the voice recognition will be performed, install ggerganov whisper.cpp from github repo as explained there. Link above. Download "base" model. Test it to the point that you can run the local transcription tests in the README with base model. UPDATE: the "tiny" model works very well too and is much faster. I now recommend you use the "tiny" model and I have modified the code in minimal.py to reflect that.

Your server should have a well-known IP address. You could probably also use dynamic DNS of some sort, instead of a fixed IP address.

Install wsocket library. Use a venv if you like:
````
    pip install wsocket
````

Install sox audio processing library (e.g. apt install sox, for Ubuntu/Debian).

Install "minimal.py" from this repo into your working directory and edit the host and port in the run(). The "host" should be the IP address of the interface on the local machine that the program should listen for service requests. This might be "localhost", or possibly "0.0.0.0" to listen on all interfaces on the server. The "port" can be most anything but must agree with whatever you set up on the client side (below). I used port 9001. Also edit the "transcmd" in minimal.py to reference the location of your whisper installation. The "transcmd" is simply a shell command that will be invoked to convert (using sox) the audio file sent over from the phone to a format that whisper.cpp can use, and then invoke whisper.cpp "main" on it with the base recognition model. The result is printed on stdout, captured in the Python program minimal.py, and sent back over the websocket to Kõnele on the phone for insertion into the text message, email, or command.

Start the server running from the command line and be ready to watch output:
````
    python3 minimal.py
````

Eventually you will set this up to start automatically, like in /etc/rc.local or using systemd or whatever. I like to set it up to start in a "screen" session.

---
## Client side setup (Android phone using Kõnele app for input)

Install Kõnele app (also known as K6nele) from f-droid repo onto your Android phone. I use Lineageos as my Android platform but most any Android should work. Google Play services is not required.

Configure Kõnele as follows. Under "Kõnele settings"->"Recognition services"->"Kõnele (fast recognition)" where it says "Service based on the Kaldi GStreamer server..." set the URL to point to your server address and port (set up above), for example:
````
    ws://YOURSERVERHOSTNAMEORIPADDRESS:9001/client/ws/speech
````

The port number in that ws:// URL must agree with the port that you set the server to run, above.

Make sure you set the "Audio format" to "raw" (the default).

I also use the "Anysoft Keyboard" app from f-droid as my basic input mechanism on my Android phone. I think things will work with the regular Android input keyboard system, but if you have problems, install the Anysoft Keyboard and try again.

---
## Usage

In order to get the microphone on the virtual keyboard to invoke Kõnele (and send audio to the server for transcription) you will probably have to invoke the Kõnele app first on its own once, after you boot your phone. After that it should happen automatically when you invoke your normal input system, until you reboot your phone.

When you click on the microphone input on your Android phone's virtual keyboard, you should see the server side "minimal.py" print a message acknowledging the connection. As you speak, the audio data should be captured into a .raw audio file in the working directory on the server side. When you click to stop input on the phone, you should see a message on the server side indicating that the audio is complete, and whisper.cpp should be invoked on the saved audio file after conversion to an audio format that whisper.cpp wants. Then a message should be sent back to the phone with the text of the transcription. The resulting text can be inserted into a text message or email as you wish.

If you click on the Kõnele app itself (instead of the microphone on the Android keyboard) it will do a web search with the recognized text instead.

---
## Limitations and cautions

There is absolutely no security in this implementation. I run this over a tinc VPN, but if you expose the server IP to the world there are security implications. That is your problem.

There can only be one client connection to the voice recognition server at a time. The code does not attempt to use a different filename to capture the audio if multiple requests come in at once. I'm not even sure if the websockets implementation could handle multiple requests at once. This could be fixed pretty easily.

The audio format in the transmission to the server is Kõnele's default raw and uncompressed audio. Kõnele supports also FLAC, but if you change the format on the client side in Kõnele, you will also have to change the sox command on the server side. The encoding format could be determined dynamically by the url on the server, but this is not currently done.

---
## Alternatives

The whisper.cpp code is more optimized for Apple silicon, and does not run nearly as fast on Android phones as it does on Apple phones (a quarter the speed?) But it still runs fairly well! If you want to try to run a local (on-phone, offline with no network connection) then I recommend that you look at this:

- [WhisperInput](https://github.com/alex-vt/WhisperInput)

I compiled it and tried it out and it works great. For my setup (Pixel 5a with fast network and fast server) it is on average significantly slower returning recognitions since it is running on-phone compared to running on a fast server, but it still runs very well! Give it a try. For short recognitions, it could be faster to run on-phone, especially if your server is not fast.

Another server based approach uses the Kaldi voice recognizer instead of Whisper:

- [Kaldi-Gstreamer-server](https://github.com/alumae/kaldi-gstreamer-server)

I ran a Kaldi server for years and accessed it from my open Android phone using Kõnele as the frontend (just as I describe above). It worked pretty well. I find Whisper to be a much superior voice recognition system though. I think maybe Kaldi works better in other languages than it does in English, but I doubt it compares to Whisper these days.

---
## Feedback

These instructions are being improved quickly now. If you encounter problems, send a question and these instructions will be improved.
