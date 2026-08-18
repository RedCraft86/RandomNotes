## This document outlines how to manually update the Expressive Text plugin to recent Unreal Engine 5 versions

# Prerequisites
## Locating the Expressive Text Plugin
- If you **have Unreal Engine 5.5 installed**, you can download the Expresive Text plugin by installing it to the engine and locating the file at `UE_INSTALL_DIR/Engine/Plugins/Marketplace/(Plugin here with encrypted name)`
- If you **do not have Unreal Engine 5.5 installed**, you can either download it (deselect everything in install options for a smaller size) and obtain the plugin via the above step, or use *Asset Manager Studio* to download it directly.

## Creating a C++ Project
1) In your target Unreal Engine version (for example, 5.7) create a **C++ based project**.
2) Once done, go to the folder of said project and create a new folder named `Plugins`
3) Put the Expressive Text plugin into that Plugins folder

# Porting the Plugin
## Initial Steps
- Open up `ExpressiveText.uplugin` and remove the `EngineVersion` entry entirely.
- In the same file, remove the `Installed` entry as well.

## Porting to 5.6
- In `ExpressiveTextLayout.h` - `Line 406`
  - Change `OffsetX` to `Offset`
    
- In `ExpressiveTextCompiler.h` - `Line 107`
  - Remove the `auto` from within the parenthesis
    
- In `UnlogImplementation.h` - `Line 438` (the `ProcessFormatString` function)
  - Remove the `FString Result...` and `return...` lines only and copy the following code to replace them
  ```cpp
  TCHAR Buffer[2048];
  int32 Length = FCString::Snprintf(Buffer, UE_ARRAY_COUNT(Buffer), Format, Args...);
  if (Length >= 0 && Length < UE_ARRAY_COUNT(Buffer))
  {
      return FString(Buffer);
  }
  
  // Uh-oh
  if (Length < 0)
  {
      return FString();
  }
  
  TArray<TCHAR> LargeBuffer;
  LargeBuffer.SetNumUninitialized(Length + 1);
  FCString::Snprintf(LargeBuffer.GetData(), LargeBuffer.Num(), Format, Args...);
  return FString(LargeBuffer.GetData());
  ```
  
## Porting to 5.7
- **Apply all the changes for 5.6 before starting**
  
- `ExpressiveTextCompiler.h` - `Line 257`  
  and `ExpressiveTextRun.cpp` - `Line 133`  
  and `ExTextGoogleFontsImporterSubsystem.h` - `Line 80`  
  - Change `CompositeFont` to `GetInternalCompositeFont()`
    
- `ExTextGoogleFontsImporterSubsystem.h` - `Line 515`  
  - Change `CompositeFont` to `GetMutableInternalCompositeFont()`
    
- `ExTextGoogleFontsImporterSubsystem.h` - `Line 78`  
  - Change `CompositeFont` **on the right** to `GetInternalCompositeFont()`
  - Change `CompositeFont` **on the left** to `GetMutableInternalCompositeFont()`

## Porting to 5.8
- **Apply all changes for 5.6 and 5.7 before starting**

- `UnlogImplementation.h` - `Line 288`  
  and `ExpressiveTextEditorSubsystem.cpp` - `Line 83`
  - Change `OnPostEngineInit` to `GetOnPostEngineInit()`

- `ExTextComboBox.h` - `Line 9`
  - Change `<SSearchableComboBox.h>` to `<Widgets/Input/SSearchableComboBox.h>`

- `ExTextMultiLineEditableText.h` - `Line 23, 37, 41, 42`
  - Change `EditableTextLayout->` to `SlateEditableTextLayout.` (**Important: Arrow becomes period!**)

- `HTTP.h` and `RemoteLogSink.h` (Oh boy, this one is annoying)
  - Use the regex `\bSet\w+Field\b` to search for all `Set[...]Field` function calls
  - If the first parameter is a raw `"..."` make sure to wrap it in `TEXT()` to become `TEXT("...")`  
    Example 1: `"appId"` -> `TEXT("appId")` | Example 2: `"events"` -> `TEXT("events")`  
    (If it is already wrapped, no change is needed)

**The following is to fix the ExpressiveTextActor flickering** (thanks @\_stumpchunkman\_ on discord)
- `ExpressiveTextComponent.h` - `Line 177`  
  - Add a third parameter to the NewObject function and provide `RF_Transient | RF_DuplicateTransient`  
    `...(this, *ObjectName);` -> `...(this, *ObjectName, RF_Transient | RF_DuplicateTransient);`

- `ExpressiveTextComponent.h` - `Line 212`  
  - Add a second parameter to the UPROPERTY macro and provide `DuplicateTransient`  
    `UPROPERTY( Transient )` -> `UPROPERTY( Transient, DuplicateTransient )`

**@\_stumpchunkman\_'s guide on how to get the Expressive Text Actor to cast shadows again**
  - [AndrewCFG/expressive-customizations](https://github.com/AndrewCFG/expressive-customizations)

# Final Steps
## If you are using it in a C++ project
Technically, you are done. You can move the plugins folder into your project and your project will compile the plugin approprately.

## If you are using it in Multiple or BP-Only projects
- Run the C++ project that you are modifying the plugin in.
- Go to the Plugins tab and find the Expressive Text plugin.
- Click on "Package" under the plugin description. (If you do not see this, make sure `Installed` is either removed or set to false in the `.uplugin` file)
- Once packaged, you can move the packaged result into your target Unreal's Plugins/Marketplace folder.
