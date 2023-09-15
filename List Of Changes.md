Fixes just for Desktop
======================


Fixes small white corners of tooltip windows on some hosts
-----------------------------------------------------------

In file juce_TooltipWindow.cpp

TooltipWindow::TooltipWindow

Change:

```
    setOpaque (true);
```

to: 
```
    setOpaque (false); // set to false avoids white triangles on some hosts
```



Resizable corner area is too small and fiddly to find with mouse
-----------------------------------------------------------------

In file juce_AudioProcessorEditor.cpp

AudioProcessorEditor::editorResized

At line 192

Change 
```
            const int resizerSize = 18;
```
to 
```
            const int resizerSize = 35;
```



Fixes just for iOS
==================


Fixes Ok / Cancel dialog boxes not showing on iOS
--------------------------------------------------

In file juce_UIViewComponentPeer_ios.mm

UIViewComponentPeer::UIViewComponentPeer add this line at line 1791:


```
    // currentlyFocusedPeer needs be set so that message boxes display in AUv3s
    iOSGlobals::currentlyFocusedPeer = this;
```


Like this:
```
    setTitle (component.getName());
    setVisible (component.isVisible());
    
    // currentlyFocusedPeer needs be set so that message boxes display in AUv3s
    iOSGlobals::currentlyFocusedPeer = this;
}
```




Always show "Save" rather than "Move" in save dialog box on iOS
----------------------------------------------------------------

In file juce_FileChooser_ios.mm

getViewController

Change to this at line 189
```
            UIDocumentPickerMode pickerMode = UIDocumentPickerModeExportToService;
```
Like this:

``````
//            UIDocumentPickerMode pickerMode = currentFileOrDirectory.existsAsFile()
//                                                ? UIDocumentPickerModeExportToService
//                                                : UIDocumentPickerModeMoveToService;

            // <-- fix: always show "Save" -->
            
            UIDocumentPickerMode pickerMode = UIDocumentPickerModeExportToService;
            
``````



Fixes fast tapping on popup menus not registered on iOS, and also sub-menus not working
--------------------------------------------------------------------------------------

In file juce_PopupMenu.cpp

handleMousePosition

Add this:

```
#if JUCE_IOS
    
    void handleMousePosition (Point<int> globalMousePos)
    {
        auto localMousePos = window.getLocalPoint (nullptr, globalMousePos);
        auto timeNow = Time::getMillisecondCounter();
        
        if (timeNow > window.timeEnteredCurrentChildComp + 10
             && window.reallyContains (localMousePos, true)
             && window.currentChild != nullptr
             && ! (window.disableMouseMoves || window.isSubMenuVisible()))
        {
            window.showSubMenuFor (window.currentChild);
        }
        
        // highlight only if pressed
        
        if (ModifierKeys::currentModifiers.isAnyMouseButtonDown()
            || ComponentPeer::getCurrentModifiersRealtime().isAnyMouseButtonDown()){
            highlightItemUnderMouse (globalMousePos, localMousePos, timeNow);
        }
        
        const bool overScrollArea = scrollIfNecessary (localMousePos, timeNow);
        const bool isOverAny = window.isOverAnyMenu();
        
        if (window.hideOnExit && window.hasBeenOver && ! isOverAny)
            window.hide (nullptr, true);
        else
            checkButtonState (localMousePos, timeNow, isDown, overScrollArea, isOverAny);
    }

#else
    
    void handleMousePosition (Point<int> globalMousePos)
    {
        auto localMousePos = window.getLocalPoint (nullptr, globalMousePos);
        auto timeNow = Time::getMillisecondCounter();

        if (timeNow > window.timeEnteredCurrentChildComp + 100
             && window.reallyContains (localMousePos, true)
             && window.currentChild != nullptr
             && ! (window.disableMouseMoves || window.isSubMenuVisible()))
        {
            window.showSubMenuFor (window.currentChild);
        }

        highlightItemUnderMouse (globalMousePos, localMousePos, timeNow);

        const bool overScrollArea = scrollIfNecessary (localMousePos, timeNow);
        const bool isOverAny = window.isOverAnyMenu();

        if (window.hideOnExit && window.hasBeenOver && ! isOverAny)
            window.hide (nullptr, true);
        else
            checkButtonState (localMousePos, timeNow, isDown, overScrollArea, isOverAny);
    }
    
#endif

```



Easier to read text size for MIDI keyboard component
----------------------------------------------------

In juce_MidiKeyboardComponent.cpp

in MidiKeyboardComponent::drawWhiteNote

change 
```
        auto fontHeight = jmin (12.0f, getKeyWidth() * 0.9f);
```

to 
```
        auto fontHeight = jmin (18.0f, getKeyWidth() * 0.9f); // easier to read text size
```
