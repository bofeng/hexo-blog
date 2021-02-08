---
title: 'Shortcut to create new file under current folder in macOS'
date: 2021-02-06 14:02:34
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
The year is 2021 and there is still no "New Text File" in folder's context menu in macOS.

To add a quick button for it, we could use automator.
1, Open the "Automator" app, choose "New Document > Application", choose "Run Apple Script", and drag to the workflow window
![](/post-images/1612732417980.png)
2, In the Apple Script content, paste the code below:
```applescript
set file_name to "untitled"
-- here we define the extension, I use .md, change to .txt if you want
set file_ext to ".md"
set is_desktop to false

-- get folder path and if we're in desktop (no folder opened)
try
    tell application "Finder"
        set this_folder to (folder of the front Finder window) as alias
    end tell
on error
    -- no open folder windows
    set this_folder to path to desktop folder as alias
    set is_desktop to true
end try

-- get the new file name (do not override an already existing file)
tell application "System Events"
    set file_list to get the name of every disk item of this_folder
end tell
set new_file to file_name & file_ext
set x to 1
repeat
    if new_file is in file_list then
        set new_file to file_name & " " & x & file_ext
        set x to x + 1
    else
        exit repeat
    end if
end repeat

-- create and select the new file
tell application "Finder"
    
    activate
    set the_file to make new file at folder this_folder with properties {name:new_file}
    if is_desktop is false then
        reveal the_file
    else
        select window of desktop
        set selection to the_file
        delay 0.1
    end if
end tell
```
3, Click "File > Save", name it "New .md File", and save it to "/Applications" folder
4, Open your applications folder, find the "New .md File" app, hold the `cmd` key and drag the app to "Finder"'s toolbar:
![](/post-images/1612732092973.png)

Now we can go to any folder then click that icon, then a new `untitled.md` file will be created inside that folder.

### References
* [https://www.maketecheasier.com/create-blank-text-file-mac/](https://www.maketecheasier.com/create-blank-text-file-mac/)
* [https://gist.github.com/rarylson/5d20fc96335851365a02](https://gist.github.com/rarylson/5d20fc96335851365a02)
