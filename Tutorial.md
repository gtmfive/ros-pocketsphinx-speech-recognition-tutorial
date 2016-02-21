# Introduction #

ROS/Pocketsphinx Speech Recognition Tutorial

Part 1) Install sfml audio from http://www.sfml-dev.org/index.php
> SFML (simple and fast multimedia library) is a C++ API that provides you low and high level access to graphics, input, audio, etc. We will use this as our wav file player.

This is the same mechanism utilized by Garratt Gallagher from his kinect piano play demo.
I have modified the wav playing section of Garratt’s code so that it can be invoked on the command line and takes the path to any wav file as an easy mechanism for playing wav files.
For those that have not had the best success with the ros sound\_play node configuration, you will find this to be a welcome alternative as it can be used in any program as a nice reliable way to play back sounds.

For this speech recognition tutorial, I use it to play back wav files in response to recognized sentences.

Part 2) Setup and Configuration of  the audio player program

  * Create a directory to house your wav “response” files that will be called when you want your robot to reply to a section of recognized speech. Mine is located at ~/audio\_files
  * Create new custom ros package for the cpp code that will play the wav files (exaudio.cpp)
> > o Create directory to house your custom ros package (mine is ~/scott\_code)
> > o Create custom ros package (roscreate-pkg audio\_player std\_msgs rospy roscpp)
> > o cd audio\_player and type rosmake (enter)
> > o cd to src dir and download wav\_player.cpp to that dir
> > o Go back up one directory and edit the CMakeList.txt
> > o At bottom of file add the following lines


rosbuild\_add\_executable(wav\_player src/wav\_player.cpp)

target\_link\_libraries(wav\_player sfml-audio)


> o save and exit the file then type rosmake (enter)
> o put a wav file into your audio\_files directory
> o cd to the bin directory of the audio\_player package (mine is at /home/wilson2/scott\_code/audio\_player/bin)
> o Type the following (or your equivalent of) to test the audio player, changing out the name of the wav file I used for one in your audio\_files directory


./wav\_player /home/wilson2/audio\_files/hello\_scott.wav

You should now hear your wav file play. I will show you in a moment how to call your new wav\_player application from your speech recognition progam

Part 3) Pocketsphinx install and voice recognition python script

  * Go to http://cmusphinx.sourceforge.net/wiki/download/ to download the pocketsphinx speech recognition software. (basically you need pocketsphinx and sphinxbase).
  * Copy the voice recognition python script (called custom\_voice\_cmd.py) attached to this entry to a directory on your system. Note: if it is not executable, then run


sudo chmod +x custom\_voice\_cmd.py

Let’s examine the python code before we run it.
Before we start, this is where I want to thank Eric Perko for pointing me to the cwru-ros-pkg that the python script I have attached here was derived from.

….import statements.....
At the top of the python script you will see the import statement for the components that the script depends on. You must install these on your system for the python script to work. The ones you may not have installed yet are...
pygtk
gobject
pygst
gst
You should be able to google search for them or just use Ubuntu’s package manager to get them loaded onto your system. Additional information on installing these files can be found at http://cmusphinx.sourceforge.net/wiki/gstreamer

….path to language model and dictionary files....
If you scroll further down the script you will see the following entries:
> asr.set\_property('lm', '/home/wilson2/scott\_code/language\_model/1140.lm')
> asr.set\_property('dict', '/home/wilson2/scott\_code/language\_model/1140.dic')
I will explain to you in a moment how you will create your own custom version of these that are created from your own sentences. Once you have them, you will edit these two entries to point that the location that you download your versions of these files to after creating them.

If you scroll down further, you will get to a block of code preceding several if-elseif statements like this...
> print "Final: " + hyp
> hyp = hyp.lower()

> if "go to" and "refrigerator" in hyp:
> > str = "Going to the refrigerator. \n"
> > self.msg.data = "refrigerator"

> elif "turn left" in hyp:
> > str = "Turning left. \n"
> > self.msg.data = "left"

When the speech recognition program runs, it will do it’s best to find the best match in you list of sentences to what is spoken. It will then print that to the screen with the print "Final: " + hyp line.

Post that, the block of if statements is used to decide what you want to do with the recognized sentence. Basically, you’ll have on if statement for every sentence that you train your system to recognize.
The last entry in the if block is an else statement that basically is the catch all that prints “I don’t understand” if no good match is found.

….inside an if-then block...

> elif "go forward" in hyp:
> > str = "Moving forward. \n"
> > self.msg.data = "forward"
> > os.system("/home/wilson2/scott\_code/audio\_player/bin/wav\_player /home/wilson2/audio\_files/off\_we\_go.wav")

In the example block above a couple of things happen.
if the sentence “go forward” is matched, then the str variable is set and can be used to print to the screen for informational output and a ros message is set, which is what will be posted to other ros nodes downstream which you will be able to test independently in a terminal window using the rostopic list and echo commands.

If you would like your robot to respond to your recognized sentence, you can have it “play” a response wav file in your audio\_files directory using the sfml c++ application (wav\_player) we created at the top of this tutorial, by calling you wav playing program with the os.system command, and passing the program name and the wav file to play.

Part 4) Creating your own custom sentences

  * Create a directory to hold and create your language model in. I called mine surprisingly enough language\_model
  * Create a file called corpus.txt and enter the sentences that you want your robot to understand. Mine include some basic entries like turn left, turn right, hello wilson (the name of my robot), and go to the refrigerator (something I hope Wilson we be able to do from a vslam map in the not too distant future)
  * Go to the following url and upload your corpus.txt file  http://www.speech.cs.cmu.edu/tools/lmtool.html then hit the Compile Knowledegebase button. This will create the custom language model for your sentences.
  * Extract zip file of created language model into your language\_model dir
  * Edit the entries in the python script we walked through above to point at your new .dic and .lm files.
  * Edit the else-if blocks to match the sentences in your corpus.txt file
  * Edit the sections to what you want the “to the screen” str messages to be, the “over a ros topic” self.data.msg to be and to execute any wav file you desire.



Part 5) Running your new custom speech recognition program

  * The python script needs roscore to be running. So open a new terminal window and type roscore
  * After you modify, save and exit the custom\_voice\_cmd.py file in a separate terminal window you are ready to run it. Just type ./custom\_voice\_cmd.py from the directory where it lives.
  * When the gtk gui interface appears, press the speak button and say one of your sentences. It will try to show you what it is interpretting. Keep and eye on the terminal window that you ran the python script from as well to see if it matched one of your sentences or if it returned with an “ I don’t understand” response.
  * To test that the python script is posting your recognized sentence on a rostopic do the following
> > o With your program running, open a new terminal window and type rostopic list
> > o You can then run rostopic echo chatter or rostopic echo speak to see it pick up the rostopic message corresponding to your entry in the else-if block of the python script. You can then create code to use these message to drive other robot behaviors.



That’s about it. I hope that others find this tututorial useful, if you did, please shoot me an e-mail at bellscotcv@gmail.com. Thanks to Garratt Gallagher for his sfml coding, to Eric Perko for the tips and to the cwru-ros-pkg developers for all of there work which the code in the tutorial was derived from.


# Details #

Add your content here.  Format your content with:
  * Text in **bold** or _italic_
  * Headings, paragraphs, and lists
  * Automatic links to other wiki pages