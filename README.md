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
   
3. Grant ADB permissions
   
   https://tasker.joaoapps.com/userguide/en/help/ah_secure_setting_grant.html
   
5. Install AutoTools and AutoInput (AutoTools only required for keyboard switching)
   
   AutoTools: https://play.google.com/store/apps/details?id=com.joaomgcd.autotools
   
   AutoInput: https://play.google.com/store/apps/details?id=com.joaomgcd.autoinput

## Keyboard Switching
This is relatively simple, I will attach code below but this link can explain it much better than I can.

https://www.xda-developers.com/how-to-automatically-change-your-keyboard-on-a-per-app-basis/

This guide was written around the use case of switching when a specific app was opened for changing between voice typing and keyboard typing, the general process is the same except changing the profile to a state instead.

### Code
1. Setup profile with State: Keyboard Out
2. Create task to switch to the keyboard, this only activates while the profile is true. I also like to have the screen flash up that it has switched just so I know it's working although this is not necessary.
3. Create an exit task to switch the keyboard back to the original one once the physical keyboard is removed.
   
```

    Profile: Keyboard switcher
    	State: Keyboard Out
    
    
    
    Enter Task: Keyboard connected
    
    A1: Flash [
         Text: Keyboard connected switching to Samsung Keyboard
         Continue Task Immediately: On
         Dismiss On Click: On ]
    
    A2: AutoTools Secure Settings [
         Configuration: Input Method: com.samsung.android.honeyboard/.service.HoneyBoardService
         Timeout (Seconds): 60
         Structure Output (JSON, etc): On ]
    
    
    
    Exit Task: keyboard disconnected
    
    A1: Flash [
         Text: Keyboard disconnected switching to Gboard
         Continue Task Immediately: On
         Dismiss On Click: On ]
    
    A2: AutoTools Secure Settings [
         Configuration: Input Method: com.google.android.inputmethod.latin/com.android.inputmethod.latin.LatinIME
         Timeout (Seconds): 60
         Structure Output (JSON, etc): On ]
```

![](https://github.com/harrisondrew/Samsung-Keyboard-Import-Text-Shortcuts/blob/main/KeyboardSwitchProfile.jpg)

## Automating text shortcuts
The first task is to transform the .txt file provided by [DenverCoder1](https://github.com/DenverCoder1) into a CSV file, this enables it to be read as an array and extract each line in turn. There are several duplicates in there so keep an eye out if you use that one yourself or just view the one stored here. 
However I doubt most people need LaTeX dictionary so just ensure whatever you are importing is in the CSV, others might work too but I have only played around enough to get this working.

This utilises Actions v2 in AutoTools but using Actions v1 is possible and can solve some obscure use cases where Actions v2 is unable to read data.
If your shortcuts involve brackets or are having issues have a look at the V1 program I made. [TextShortcutsV1](https://github.com/harrisondrew/Samsung-Keyboard-Import-Text-Shortcuts/blob/main/TextShortcuts_ActionsV1)

### Code
This one is a bit more advanced however nothing in comparison to most!

```
    Task: TextShortcutsV2

    <Clears all the variables initially, had a few issues with data being saved in memory and messing things up further down the line>
    A1: Variable Clear [
         Clear All Variables: On ]

    <Set the initial starting point, not required however if you have to pause and return this is a useful feature to have>
    A2: Variable Set [
         Name: %CurrentPos
         To: 0
         Structure Output (JSON, etc): On ]

    <Read the whole array, this is to work out how long the whole file is to be used later in the loop. 
      The array is stored in %latex_length and the length of the array can be read through %latex_length(#). 
      Ensure that the file is being read in vertical mode and it is reading the CSV correctly, 
      use flashes to output singular lines until you are sure the programs work, then add a loop into it>
    A3: AutoTools Arrays [
         Configuration: Input Arrays: /storage/emulated/0/Download/AutoTools/dictionary.csv
         Vertical Mode: true
         Separator: 
         
         Item Separator: ,
         Input Is File: true
         Names: latex_length,symbols_length
         Output Variables Separator: ,
         Merged Array Name: atmergedarray
         Timeout (Seconds): 60
         Structure Output (JSON, etc): On ]

    <Loop. Reads current position and compares it to the last index in the array, 
    if it is less it will continue. +1 so that it can read the last item. 
    <= would be best here but is not an option>
    A4: If [ %CurrentPos < %latex_length(#)+1 ]

        <Read the line at the current index position and output into a variable>
        A5: Read Line [
             File: Download/AutoTools/dictionary.csv
             Line: %CurrentPos
             To Var: %CSV_Line
             Structure Output (JSON, etc): On ]

        <Use AutoTools to parse the array into separate variables>
        A6: AutoTools Arrays [
             Configuration: Input Arrays: %CSV_Line
             Vertical Mode: true
             Separator: 
             
             Item Separator: ,
             Names: latex,symbol
             Output Variables Separator: ,
             Merged Array Name: atmergedarray
             Timeout (Seconds): 60
             Structure Output (JSON, etc): On ]

        <A7 are screen press actions performed by AutoInput Actions v2.
        This operates by pressing the + icon on the settings screen, 
        then entering the appropiate items into both text fields then finally pressing add>
        A7: AutoInput Actions v2 [
             Configuration: Actions To Perform: click(id,com.samsung.android.honeyboard:id/text_shortcuts_add_menu)
             
             setText(id,com.samsung.android.honeyboard:id/text_shortcut_add_popup_shortcut_edittext,%latex\(\))
             
             setText(id,com.samsung.android.honeyboard:id/text_shortcut_add_popup_phrase_edittext,%symbol\(\))
             
             click(id,android:id/button1)
             Not In AutoInput: true
             Not In Tasker: true
             Separator: ,
             Check Millis: 1000
             Timeout (Seconds): 600
             Structure Output (JSON, etc): On ]

        <Iterate CurrentPos index position to go to the next item>
        A8: Variable Add [
             Name: %CurrentPos
             Value: 1
             Wrap Around: 0 ]

        <Goto A4 and repeat everything while the IF statement is TRUE>
        A9: Goto [
             Type: Action Number
             Number: 4 ]
    
    
```
    
![](https://github.com/harrisondrew/Samsung-Keyboard-Import-Text-Shortcuts/blob/main/TextShortcutsV2.jpg)

#Helpful links
1. Reading items from CSV
   https://www.reddit.com/r/tasker/comments/jaifd8/comment/g8qk4qv/?utm_source=share
2. Autoinput write variable (ActionV1)
   https://www.reddit.com/r/tasker/comments/7j7bn3/comment/dr47n6n/?utm_source=share
