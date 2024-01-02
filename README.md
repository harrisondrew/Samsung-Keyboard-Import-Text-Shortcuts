# Method to automate importing text shortcuts into Samsung Keyboard
TLDR Cannot use math dictionary on samsung keyboard, so automated input from CSV list
For code jump to bold parts

Gboard on my Samsung Tab S7+ has an annoying issue where the toolbar, where autocorrect words and other features appear, does not show up when the physical hardware keyboard is connected. For me this is the magnetic book cover however bluetooth may be used as well.

I work a lot with LaTeX so imported the below personal dictionary into GBoard
https://github.com/DenverCoder1/latex-gboard-dictionary/

Because of the above issue with not being able to view the toolbar I couldn't make use of the math symbol library when writing notes so created a tasker plugin to automatically switch the keyboard from GBoard to Samsung Keyboard whenever the physical keyboard was connected. 

The next issue then became that Samsung keyboard does not allow you to import your own dictionary like google does therefore it cannot be made use of again, however you can create text shortcuts which are basically autocorrect and show up as an option in the toolbar when entered.
This works except each shortcut must be entered manually and you cannot import a text file or anything, no-one online had any solutions other than it can't be done or switch to a different keyboard which I did not want to do because of other available integrations. Also with over 1000 shortcuts I did not fancy doing this by hand.

The only option left to me, without rooting is to automate the input of every single entry.

## Setting up Tasker
1. Install Tasker from the google play store
   https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm&hl=en_GB&gl=US
2. Grant ADB permissions
   https://tasker.joaoapps.com/userguide/en/help/ah_secure_setting_grant.html
3. Install AutoTools and AutoInput (AutoTools only required for keyboard switching)
   AutoTools: https://play.google.com/store/apps/details?id=com.joaomgcd.autotools
   AutoInput: https://play.google.com/store/apps/details?id=com.joaomgcd.autoinput

## Keyboard Switching
This is relatively simple, I will attach code below but this link can explain it much better than I can.
https://www.xda-developers.com/how-to-automatically-change-your-keyboard-on-a-per-app-basis/
This guide was written around the use case of switching when a specific app was opened for changing between voice typing and keyboard typing, the general process is the same except changing the profile to a state instead.

### Code


## Automating text shortcuts
The first task is to transform the .txt file provided in [DenverCoder1s github repo](https://github.com/DenverCoder1) into a CSV file, this enables it to be read as an array and extract each line in turn. There are several duplicates in there so keep an eye out if you use that one yourself or just view the one stored here. 
However I doubt most people need LaTeX dictionary so just ensure whatever you are importing is in the CSV, others might work too but I have only played around enough to get this working.

###Code




