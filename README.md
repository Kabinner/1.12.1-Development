# 1.12.1-Development
Documenting my experience.
```
This is a work in progress.
Please submit an issue if you want to add, or something does not function properly. 
```
- [Addon template](#template)
- [File isolation](#file-isolation)
- [Enable Lua error messages](#enable-lua-error-messages)  
- [Strings](#strings)
- [Function calls](#function-calls)
- [UIDropDownMenu](#uidropdownmenu)
- [Moveable Frames](#moveable-frames)
- [Movable Minimap Button](#minimap-button)
- [Making a basic window](#frame-window)
- [Debugging](#debugging)
- - [Debugging Hooks](#debugging-hooks)  
- - - [Tracing Function Execution](#tracing-function-execution)  
- - - [Manually Hooking a Function](#manually-hooking-a-function)  
- - - [Hooking Multiple Functions Dynamically](#hooking-multiple-functions-dynamically)  
- - - [Debugging Hooked Functions](#debugging-hooked-functions)  
- - [Inspecting Addon Variables](#inspecting-addon-variables)  
- - [Logging and Persistent Debugging](#logging-and-persistent-debugging)  
- [Debug.lua](#debuglua)
- [AI Chatbot prompt](#ai-chatbot-prompt)
- [Resources](#resources) API, Locate Interface files(icons, etc). etc
  
## Enable Lua error messages:
```
/console scriptErrors 1
    Displays errors in-game instead of silently failing.
```

## File isolation
```lua
-- Isolate this file.
local _G = getfenv(0)
_G = setmetatable({
    _G = _G
}, {
    __index = _G
})
setfenv(1, _G)
```

## Strings
```lua
[[Interface\Spellbook\UI-SpellbookPanel-BotLeft]]
instead of
"Interface\\Spellbook\\UI-SpellbookPanel-BotLeft"
e.g:

BottomLeft:SetTexture([[Interface\Spellbook\UI-SpellbookPanel-BotLeft]])
```
## Function calls
### unpack, Variadic 
```lua
function MyFunc(foo, bar, ...)
   -- foo = "Foo"
   -- bar = "Bar"
   -- arg[1] == "foobar"
   -- MyOtherFunc(unpack(arg)) # calling another function:
   -- table.getn(arg) == 2     # length
end

MyTable = {
  "foobar","barfoo"
}
MyFunc("Foo", "Bar", unpack(MyTable))
```

### UIDropDownMenu
| Function | Description | Example |
|----------|------------|---------|
| **`UIDropDownMenu_Initialize(frame, initFunction)`** | Initializes the dropdown menu and calls `initFunction` to populate it with options. | `UIDropDownMenu_Initialize(MyDropDown, Initialize)` |
| **`UIDropDownMenu_AddButton(info, level)`** | Adds a button (option) to the dropdown menu. The `info` table should define properties like `text`, `func`, and `value`. | `UIDropDownMenu_AddButton({ text = "Option 1", func = MyFunction })` |
| **`UIDropDownMenu_SetWidth(width, frame)`** | Sets the width of the dropdown menu. | `UIDropDownMenu_SetWidth(150, MyDropDown)` |
| **`UIDropDownMenu_SetText(text, frame)`** | Sets the label (text) displayed on the dropdown button. | `UIDropDownMenu_SetText("Choose an option", MyDropDown)` |
| **`UIDropDownMenu_SetSelectedID(frame, id)`** | Sets the selected option by index (starting from 1). | `UIDropDownMenu_SetSelectedID(MyDropDown, 2)` |
| **`UIDropDownMenu_SetSelectedValue(frame, value)`** | Sets the selected option by its assigned `value`. | `UIDropDownMenu_SetSelectedValue(MyDropDown, "option2")` |
| **`UIDropDownMenu_SetSelectedName(frame, name)`** | Sets the selected option by its displayed `text`. | `UIDropDownMenu_SetSelectedName(MyDropDown, "Option 2")` |
| **`UIDropDownMenu_GetSelectedID(frame)`** | Returns the index of the currently selected option. | `local id = UIDropDownMenu_GetSelectedID(MyDropDown)` |
| **`UIDropDownMenu_GetSelectedValue(frame)`** | Returns the `value` of the currently selected option. | `local value = UIDropDownMenu_GetSelectedValue(MyDropDown)` |
| **`UIDropDownMenu_GetSelectedName(frame)`** | Returns the `text` of the currently selected option. | `local name = UIDropDownMenu_GetSelectedName(MyDropDown)` |


## Moveable Frames
```
Saves the position of the frame to layout_cache.txt on logout.
Important! Frames should be created in the OnLoad event. or the position saving won't work.
```
```lua
LedgerFrame:EnableMouse(true)
LedgerFrame:SetMovable(true)
LedgerFrame:SetUserPlaced(true)

LedgerFrame:RegisterForDrag("LeftButton")

-- Purely draggable.
LedgerFrame:SetScript("OnDragStart", function() LedgerFrame:StartMoving() end)
LedgerFrame:SetScript("OnDragStop", function() LedgerFrame:StopMovingOrSizing() end)

-- While holding shift.
LedgerFrame:SetScript("OnDragStart", function() if IsShiftKeyDown() then LedgerFrame:StartMoving() end end)
LedgerFrame:SetScript("OnDragStop", function() LedgerFrame:StopMovingOrSizing() end)
```

# Debugging
## Debugging Hooks
```lua
-- Save the original function
local oldSendChatMessage = SendChatMessage  

-- Create a hook to track chat messages
SendChatMessage = function(msg, chatType, language, channel)
    DEFAULT_CHAT_FRAME:AddMessage("[Debug] Sending Message: " .. msg)
    return oldSendChatMessage(msg, chatType, language, channel)
end
```

### Tracing Function Execution
If you need to know when and how often a function is being called:
```lua
local function debugTrace(funcName)
    local oldFunc = _G[funcName]
    _G[funcName] = function(...)
        DEFAULT_CHAT_FRAME:AddMessage("[TRACE] " .. funcName .. " called!")
        return oldFunc(...)
    end
end

debugTrace("CastSpellByName")
debugTrace("TargetUnit")
```
## Inspecting Addon Variables
```lua
function printTable(t, indent)
    indent = indent or ""
    for k, v in pairs(t) do
        if type(v) == "table" then
            DEFAULT_CHAT_FRAME:AddMessage(indent .. k .. " = {")
            printTable(v, indent .. "  ")
            DEFAULT_CHAT_FRAME:AddMessage(indent .. "}")
        else
            DEFAULT_CHAT_FRAME:AddMessage(indent .. k .. " = " .. tostring(v))
        end
    end
end

-- Example usage:
printTable(MyAddonData)
```

## Logging and Persistent Debugging
Add this to your .toc file:
```
## SavedVariables: DebugLog
```
### Use this script to store logs:
```lua
DebugLog = DebugLog or {}

function logDebugMessage(msg)
    table.insert(DebugLog, msg)
end

logDebugMessage("Addon Loaded!")
```
### To review logs, reload the UI (/reload) and inspect DebugLog:
```lua
    printTable(DebugLog)
```

## Manually Hooking a Function
```lua
-- Save the original function
local old_UseAction = UseAction

-- Create our hooked function
function UseAction(slot, checkCursor, onSelf)
    DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Used action slot: " .. slot)
    
    -- Call the original function
    old_UseAction(slot, checkCursor, onSelf)
end
```
### Hooking a UI Function: Detecting When the Character Frame Opens
```lua
-- Save the original function
local old_ToggleCharacter = ToggleCharacter

-- Override the function
function ToggleCharacter(frame)
    DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Character Frame Opened: " .. frame)
    
    -- Call the original function
    old_ToggleCharacter(frame)
end
```
### Hooking Multiple Functions Dynamically
```lua
-- List of functions we want to hook
local functionsToHook = {
    "UseAction",
    "TargetUnit",
    "RepairAllItems"
}

-- Hook each function dynamically
for _, funcName in ipairs(functionsToHook) do
    if _G[funcName] then  -- Ensure the function exists
        local oldFunc = _G[funcName]
        _G[funcName] = function(...)
            DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Function called: " .. funcName)
            return oldFunc(...)
        end
    end
end
```
### Hook Protected Functions in Combat
```lua
local frame = CreateFrame("Frame")
frame:RegisterEvent("UNIT_SPELLCAST_START")

frame:SetScript("OnEvent", function(self, event, unit, spell)
    if unit == "player" then
        DEFAULT_CHAT_FRAME:AddMessage("[EVENT] Player is casting: " .. spell)
    end
end)
```
## Debugging Hooked Functions
```lua
local function debugTrace(funcName)
    local oldFunc = _G[funcName]
    _G[funcName] = function(...)
        DEFAULT_CHAT_FRAME:AddMessage("[DEBUG] " .. funcName .. " called!")
        return oldFunc(...)
    end
end

debugTrace("CastSpellByName")
debugTrace("TargetUnit")
debugTrace("UseAction")
```

## Debug.lua
```lua
Debug:trace(self, "map: ", self.object.name, "[", id(self.object), "] ", self.object.name, ".", function_name, " = ", callback)
```
```
-> [TRACE] Loader[20A4E4C8]:map: Ledger[20A4E668]: Ledger.enable = 1F8E5E08
```
```lua
Debug:trace(self, "dispatch ", e, " -> ", self.name, ":", self.object_map_lookup[id(func)], "[", func, "]")
```
```
-> [TRACE] Loader[20A4E4C8]:dispatch: PLAYER_LOGIN -> Ledger:enable[1F8E5E08]
```

```lua
local function id(_)
    return string.sub(tostring(_), -8)
end
local function len(_)
    if type(_) == "table" and _["n"] then
        return table.getn(_)
    end
end

local string = setmetatable(string, {})
function string.unpack(_)
    if not table.getn(_) then
        return
    end
    args = {}
    for idx, value in ipairs(_) do
        args[idx] = value .. " "
    end  
    return unpack(args)
end    

local function print(_, ...)
    local msg = ""

    if type(_) == "table" then
        if _.name then
            msg = _.name .. ": "
        else
            msg = id(_)
        end
    elseif type(_) == "string" then
        msg = _
    end

    for idx, value in ipairs(arg) do
        if type(value) == "table" or type(value) == "function" then
            msg = msg .. id(value)
        elseif type(value) == "boolean" or type(value) == "number" then
            msg = msg .. tostring(value)
        elseif value == nil then
            msg = msg .. "nil"
        else
            msg = msg .. value
        end
    end

    DEFAULT_CHAT_FRAME:AddMessage(msg);
end


-- Debug
local Debug = {
    LEVEL="TRACE",
    INFO="INFO",
    TRACE="TRACE",
}
function Debug:print(_, level, color, ...)
    if type(_) == "table" and not _.debug then
        return
    end

    local msg = ""
    if type(_) == "string" then
        msg = _
    end
    
    if type(_) == "table" and _.name then 
        msg =  color .. "[".. level .."] Loader[" .. id(_) .."]:"
    end
    print(msg, unpack(arg))

end
function Debug:log(caller, ...)
    self:print(caller, Debug.INFO, "|cffffd700", unpack(arg))
end
function Debug:trace(caller, ...)
    if not caller or caller.LEVEL ~= self.TRACE then
        return
    end
    self:print(caller, Debug.TRACE, "|cffffd700", unpack(arg))
end


```
### Minimap button
```lua
MinimapButtonFrame = CreateFrame("Button", "BLINKER_UI_FRAME_MINIMAP_BUTTON", Minimap)
MinimapButtonFrame:EnableMouse(true)
MinimapButtonFrame:SetMovable(true)
MinimapButtonFrame:SetUserPlaced(true)
MinimapButtonFrame:SetPoint("TOPLEFT", Minimap)

MinimapButtonFrame:SetWidth(24)
MinimapButtonFrame:SetHeight(24)
MinimapButtonFrame:SetFrameStrata("MEDIUM")
MinimapButtonFrame:RegisterForClicks("LeftButtonDown", "RightButtonDown");
MinimapButtonFrame:SetNormalTexture([[Interface\Icons\Spell_Arcane_Blink]])
MinimapButtonFrame:SetHighlightTexture([[Interface\Buttons\ButtonHilight-Square]])

MinimapButtonFrame:RegisterForDrag("LeftButton")
MinimapButtonFrame:SetScript("OnDragStart", function() if IsShiftKeyDown() then MinimapButtonFrame:StartMoving() end end)
MinimapButtonFrame:SetScript("OnDragStop", function() MinimapButtonFrame:StopMovingOrSizing() end)
MinimapButtonFrame:SetScript("OnEnter", function(self) 
end)
MinimapButtonFrame:SetScript("OnClick", function(self)
  if IsShiftKeyDown() then
    return nil
  end
  if arg1 == "LeftButton" then
  elseif arg1 == "RightButton" then
  end
end)
```

### Frame window
A Frame window that looks like the basic Spellbook Window.
```lua
local LedgerFrame = CreateFrame("Frame", "FRAME_LEDGER_PANEL", UIParent)
LedgerFrame:SetWidth(384)
LedgerFrame:SetHeight(512)
LedgerFrame:SetPoint("CENTER", UIParent, "CENTER", 0, 0)

LedgerFrame:EnableMouse(true)
LedgerFrame:SetMovable(true)
LedgerFrame:SetUserPlaced(true)

LedgerFrame:RegisterForDrag("LeftButton")
LedgerFrame:SetScript("OnDragStart", function() LedgerFrame:StartMoving() end)
LedgerFrame:SetScript("OnDragStop", function() LedgerFrame:StopMovingOrSizing() end)

local CloseButton = CreateFrame("Button", "FRAME_LEDGER_PANEL_BUTTON_CLOSE", LedgerFrame, "UIPanelCloseButton")
CloseButton:SetPoint("CENTER", LedgerFrame, "TOPRIGHT", -44, -25)
CloseButton:SetScript("OnClick", function() LedgerFrame:Hide() end)

local Icon = LedgerFrame:CreateTexture(nil, "BACKGROUND")
Icon:SetTexture([[Interface\Spellbook\Spellbook-Icon]])
Icon:SetWidth(58)
Icon:SetHeight(58)
Icon:SetPoint("TOPLEFT", LedgerFrame, "TOPLEFT", 10, -8)

local TopLeft = LedgerFrame:CreateTexture(nil, "ARTWORK")
TopLeft:SetTexture([[Interface\Spellbook\UI-SpellbookPanel-TopLeft]])
TopLeft:SetWidth(256)
TopLeft:SetHeight(256)
TopLeft:SetPoint("TOPLEFT", LedgerFrame, "TOPLEFT", 0, 0)

local TopRight = LedgerFrame:CreateTexture(nil, "ARTWORK")
TopRight:SetTexture([[Interface\Spellbook\UI-SpellbookPanel-TopRight]])
TopRight:SetWidth(128)
TopRight:SetHeight(256)
TopRight:SetPoint("TOPRIGHT", LedgerFrame, "TOPRIGHT", 0, 0)

local BottomLeft = LedgerFrame:CreateTexture(nil, "ARTWORK")
BottomLeft:SetTexture([[Interface\Spellbook\UI-SpellbookPanel-BotLeft]])
BottomLeft:SetWidth(256)
BottomLeft:SetHeight(256)
BottomLeft:SetPoint("BOTTOMLEFT", LedgerFrame, "BOTTOMLEFT", 0, 0)

local BottomRight = LedgerFrame:CreateTexture(nil, "ARTWORK")
BottomRight:SetTexture([[Interface\Spellbook\UI-SpellbookPanel-BotRight]])
BottomRight:SetWidth(128)
BottomRight:SetHeight(256)
BottomRight:SetPoint("BOTTOMRIGHT", LedgerFrame, "BOTTOMRIGHT", 0, 0)

local TitleText = LedgerFrame:CreateFontString("FRAME_LEDGER_PANEL_TITLE_TEXT", "ARTWORK", "GameFontNormal")
TitleText:SetPoint("CENTER", LedgerFrame, "CENTER", 6, 230)
TitleText:SetText("Ledger")

local PageText = LedgerFrame:CreateFontString("FRAME_LEDGER_PANEL_PAGE_TEXT", "ARTWORK", "GameFontNormal")
PageText:SetWidth(102)
PageText:SetPoint("BOTTOM", LedgerFrame, "BOTTOM", -14, 96)
PageText:SetText("Page 1")
```
## AI Chatbot Prompt
```
1. You are a senior developer in Lua version 5.0.
1a. You will only suggest code in Lua version 5.0 and not above. 
2. All the questions will be about World of Warcraft addon development, 
2a. you will only reference version 1.12.1 of World of Warcraft
<important>DO NOT use modern versions of World of Warcraft. Only the very first version of World of Warcraft.</important>
```

## Template
### File Structure
```
MyAddon/
│-- MyAddon.toc    (Addon manifest file)
│-- MyAddon.lua    (Main script file)
│-- MyAddon.xml    (UI layout file, optional)
│-- Readme.md      (Documentation, optional)
```
### MyAddon.toc    (Addon manifest file)
```toc
## Interface: 11200
## Title: My Addon
## Notes: This is a test addon
## Author: YourName

MyAddon.lua
```

### MyAddon.lua    (Main script file)
```lua

local frame = CreateFrame("Frame")
frame:RegisterEvent("ADDON_LOADED")
frame:RegisterEvent("PLAYER_LOGIN")
frame:RegisterEvent("PLAYER_LOGOUT")
frame:SetScript("OnEvent", function()
    if event == "ADDON_LOADED" and arg1 == "MyAddon" then
        -- make your frames here.
        this:UnregisterEvent("ADDON_LOADED")
    elseif event == "PLAYER_LOGIN" then
        DEFAULT_CHAT_FRAME:AddMessage("Welcome to Azeroth!")
    elseif event == "PLAYER_LOGOUT" then
    end
end)
```

## Resources
- Lua 5.0 [https://www.lua.org/manual/5.0/manual.html]
- 1.12.1 API [https://wowpedia.fandom.com/wiki/World_of_Warcraft_API?oldid=352751]
- Timer-facility [https://github.com/allfoxwy/UnitXP_SP3/wiki/Timer-facility]
- MPQ Editor v 4.0.0.937 (English, 32+64-bit) [http://zezula.net/en/mpq/download.html]
- Addons
- - Improved Error Frame [https://legacy-wow.com/vanilla-addons/improved-error-frame/]
- - pfStudio [https://shagu.org/pfStudio/]
- - Prat [https://github.com/Qxcl/Prat-turtle]
- - SuperMacro [https://github.com/Monteo/SuperMacro]
- Turtle WoW [https://turtle-wow.org/]
- - wiki "For Addon Developers" [https://turtle-wow.fandom.com/wiki/Addons#For_Addon_Developers]
- VisualStudio Community [https://visualstudio.microsoft.com/downloads/]
- - Plugins: Lua, GitHub Repositories
