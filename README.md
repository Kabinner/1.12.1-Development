# 1.12.1-Development
Documenting my experience.
```
This is a work in progress.
Please submit an issue if you want to add, or something does not function properly. 
```
- [File isolation](#file-isolation)
- [Moveable Frames](#moveable-frames)
- [Strings](#strings)
- [AI Chatbot prompt](#ai-chatbot-prompt)
- [Debugging](#debugging)
- - [Enable Lua error messages](#enable-lua-error-messages)  
- - [Debugging Hooks](#debugging-hooks)  
- - - [Tracing Function Execution](#tracing-function-execution)  
- - - [Manually Hooking a Function](#manually-hooking-a-function)  
- - - [Hooking Multiple Functions Dynamically](#hooking-multiple-functions-dynamically)  
- - - [Debugging Hooked Functions](#debugging-hooked-functions)  
- - [Inspecting Addon Variables](#inspecting-addon-variables)  
- - [Logging and Persistent Debugging](#logging-and-persistent-debugging)  
- [Debug.lua](#debuglua)

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
### To review logs, reload the UI (/console reloadui) and inspect DebugLog:
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
Debug:trace("Mapping ", self.name .. ":" .. function_name, "to: ", self.object[function_name])
```
```
-> Ledger [TRACE]: Mapping Ledger:enable to: 1F8E5E08
```
```lua
Debug:log("Addon:dispatch: ", e, " -> ", self.name .. ":" .. self.object_map_lookup[id(func)], func)
```
```
-> Ledger [INFO]: Addon:dispatch: PLAYER_LOGIN -> Ledger:enable 1F8E5E08
```

```lua
local function id(_)
    return string.sub(tostring(_), -8)
end

local DEBUG = true
local DEBUG_NAME = 'Ledger'
local Debug = {
    LEVEL="TRACE",
    INFO="INFO",
    TRACE="TRACE"
}
function Debug:print(level, color, ...)
    if not DEBUG_ADDON then
        return
    end
    local msg = ""
    for idx, value in ipairs(arg) do
        if type(value) == "table" or type(value) == "function" then
            msg = msg .. id(value) .. " "
        elseif value == nil then
            msg = msg .. "nil" .. " "
        else
            msg = msg .. value .. " "
        end
    end
    print(color .. DEBUG_NAME .. " [".. level .."]: " .. msg)

end
function Debug:log(...)
    self:print(Debug.INFO, "|cffffd700", unpack(arg))
end
function Debug:trace(...)
    if self.LEVEL ~= self.TRACE then
        return
    end
    self:print(Debug.TRACE, "|cffffd700", unpack(arg))
end

```
## AI Chatbot Prompt
```
1. You are a senior developer in Lua version 5.0.
1a. You will only suggest code in Lua version 5.0 and not above. 
2. All the questions will be about World of Warcraft addon development, 
2a. you will only reference version 1.12.1 of World of Warcraft
<important>DO NOT use modern versions of World of Warcraft. Only the very first version of World of Warcraft.</important>
```
