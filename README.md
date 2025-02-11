# 1.12.1-Development

Basic Development Tools

    Text Editor:
        Notepad++ – Syntax highlighting and easy navigation.
        Visual Studio Code – Lua extensions available for better development.
        SciTE – Lightweight and used in the past for WoW development.

    Lua Interpreter (Optional)
        Lua 5.0 – The WoW 1.12.1 client runs on Lua 5.0, so use this version for testing.

    WoW Addon Manager (Optional)
        Helps organize and enable/disable addons.
        Manual organization in Interface\AddOns\ is recommended for 1.12.1.

2. Reference Materials
API & Functions Documentation

    Vanilla WoW API Documentation (GitHub)
    WoWWiki Archive (API List)
    Events List in 1.12.1

Console Variables (CVars)

    Vanilla 1.12.1 CVars – Includes graphics, sound, debugging settings.

3. Debugging & Development Utilities
Debugging

    Enable Lua Errors:
        Use the following command to enable script errors in-game:

    /console scriptErrors 1

    This will show errors when your addon has issues.

Print Debug Messages:

    Use DEFAULT_CHAT_FRAME:AddMessage("Debug Message") in your Lua scripts.

Use SavedVariables:

    Store addon data between sessions in your addon's .toc file:

        ## SavedVariables: MyAddonData

Logging

    Capture errors by writing logs into a file using in-game debugging functions.

4. Essential Addon Development Concepts
Basic File Structure

MyAddon/
│-- MyAddon.toc    (Addon manifest file)
│-- MyAddon.lua    (Main script file)
│-- MyAddon.xml    (UI layout file, optional)
│-- Readme.txt     (Documentation, optional)

TOC File (Addon Manifest)

Example MyAddon.toc file:

## Interface: 11200
## Title: My Addon
## Notes: This is a test addon
## Author: YourName

MyAddon.lua

Basic Lua Script (Main File)

Example MyAddon.lua:

-- Print message when addon is loaded
DEFAULT_CHAT_FRAME:AddMessage("MyAddon Loaded!")

-- Create a simple frame to listen for events
local frame = CreateFrame("Frame")
frame:RegisterEvent("PLAYER_ENTERING_WORLD")
frame:SetScript("OnEvent", function()
    DEFAULT_CHAT_FRAME:AddMessage("Welcome to Azeroth!")
end)

5. Learning Resources

    WoW UI & Macros Forum (Archived): WoWWiki Archive
    Addon Examples & Open-Source Repos:
        Vanilla 1.12 Addon Repository
        1.12 Addon Libraries
    Lua 5.0 Documentation: Official Lua 5.0 Reference

This list provides a complete toolkit for getting started with WoW 1.12.1 addon development, covering everything from tools to debugging and Lua scripting. Let me know if you need a specific example! 🚀
You said:
could you give me arcane information about debugging addons. tips, hooks, etc.
ChatGPT said:

Debugging addons in World of Warcraft 1.12.1 (Vanilla) is a challenging process due to the lack of modern debugging tools. However, there are arcane techniques that can help you track issues, inspect game behavior, and improve your debugging workflow. Below is an in-depth look at hidden debugging techniques, hooks, and tricks that will help you fine-tune your addons.
1. Essential Debugging Console Commands

These console variables (CVars) will help reveal errors and debugging information:

    Enable Lua error messages:

/console scriptErrors 1

    Displays errors in-game instead of silently failing.

Reload UI without restarting the game:

/console reloadui

    Refreshes all active addons.

Print the value of a variable or function:

    /dump UnitName("player")

        This command isn’t native to 1.12.1, but you can create a /dump function to print variables.

2. Advanced Debugging Hooks

In 1.12.1, hooking functions allows you to intercept and modify behavior. You can track how the UI interacts with game functions.
Hooking a Function for Debugging

-- Save the original function
local oldSendChatMessage = SendChatMessage  

-- Create a hook to track chat messages
SendChatMessage = function(msg, chatType, language, channel)
    DEFAULT_CHAT_FRAME:AddMessage("[Debug] Sending Message: " .. msg)
    return oldSendChatMessage(msg, chatType, language, channel)
end

This hook lets you spy on chat messages before they are sent.
3. Deep Inspection of Events
Finding Events Fired by the Game

You can use an event-tracking frame to log all game events to the chat frame.

local debugFrame = CreateFrame("Frame")
debugFrame:SetScript("OnEvent", function(self, event, ...)
    DEFAULT_CHAT_FRAME:AddMessage("[EVENT]: " .. event)
end)

-- Register all events
for i, event in pairs({
    "PLAYER_LOGIN", "PLAYER_LOGOUT", "PLAYER_ENTERING_WORLD",
    "CHAT_MSG_SAY", "UNIT_HEALTH", "SPELLCAST_START"
}) do
    debugFrame:RegisterEvent(event)
end

💡 This technique helps you discover undocumented or rare events!
4. Tracing Function Execution

If you need to know when and how often a function is being called, you can track it like this:

local function debugTrace(funcName)
    local oldFunc = _G[funcName]
    _G[funcName] = function(...)
        DEFAULT_CHAT_FRAME:AddMessage("[TRACE] " .. funcName .. " called!")
        return oldFunc(...)
    end
end

debugTrace("CastSpellByName")
debugTrace("TargetUnit")

This method lets you hook and monitor important API functions.
5. Inspecting Addon Variables

You can print addon variables and saved data in a readable format:

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

This function recursively prints a table, making it easier to inspect complex structures.
6. Debugging Secure Frames & Protected Functions

WoW 1.12.1 has protected functions that you cannot call directly in combat. However, you can sometimes bypass these restrictions:

    Use the /run command to test functions outside combat:

/run CastSpellByName("Frostbolt")

    Works unless blocked by Blizzard's secure function rules.

Use secure hooks if a function is protected:

    hooksecurefunc("UseAction", function(slot)
        DEFAULT_CHAT_FRAME:AddMessage("UseAction Hooked: " .. slot)
    end)

        This lets you monitor protected function calls without modifying them.

7. Logging and Persistent Debugging

You may want to log debugging messages persistently instead of printing them to chat. Use SavedVariables to store logs between sessions.
Saving Debug Messages to a File

    Add this to your .toc file:

## SavedVariables: DebugLog

Use this script to store logs:

DebugLog = DebugLog or {}

function logDebugMessage(msg)
    table.insert(DebugLog, msg)
end

logDebugMessage("Addon Loaded!")

To review logs, reload the UI (/console reloadui) and inspect DebugLog:

    printTable(DebugLog)

This technique lets you track errors over multiple sessions.
8. Using System CVars for Debugging

WoW 1.12.1 has hidden system CVars useful for debugging.

    Check current CVar values:

DEFAULT_CHAT_FRAME:AddMessage("Max FPS: " .. GetCVar("maxFPS"))

Force a lower FPS to simulate performance issues:

/console maxFPS 10

Debug graphical issues with wireframe mode:

    /console gxRestart
    /console gxApi OpenGL

9. Finding Memory Leaks

Garbage collection can help manage memory leaks.

    Check memory usage:

collectgarbage("count") -- Returns KB of Lua memory usage

Manually free memory:

    collectgarbage("collect")

This helps reduce memory bloat in long-running sessions.
10. Profiling Addon Performance

If an addon causes lag spikes, profile it:

local before = GetTime()
for i = 1, 10000 do
    local x = i * 2
end
local after = GetTime()

DEFAULT_CHAT_FRAME:AddMessage("Execution Time: " .. (after - before) .. " sec")

This helps detect slow loops and inefficient functions.
Final Arcane Debugging Tip: The Hidden WorldFrame Hook

You can track and modify everything on screen by hooking WorldFrame:

WorldFrame:HookScript("OnUpdate", function()
    DEFAULT_CHAT_FRAME:AddMessage("Frame updated at: " .. GetTime())
end)

⚠️ Use with caution: This fires every frame and can spam messages.
Conclusion

These arcane debugging techniques will help you track errors, optimize performance, and modify protected functions in Vanilla WoW (1.12.1). Let me know if you need even deeper debugging secrets! 🚀
You said:
could you elaborate on hooksecurefunc?
ChatGPT said:

Certainly!
Understanding hooksecurefunc in World of Warcraft 1.12.1
What is hooksecurefunc?

hooksecurefunc is a special function in World of Warcraft 1.12.1 that allows you to safely hook and monitor protected functions without modifying them directly. This is particularly useful because many Blizzard UI functions are protected and cannot be overridden by addons.

🔹 Key characteristics of hooksecurefunc:

    Hooks a global function without replacing it (preserving its original behavior).
    Works on protected functions, which cannot be overridden normally.
    Only allows you to observe function calls, not modify return values.

How to Use hooksecurefunc
1. Basic Example: Tracking When a Spell is Cast

Let's say we want to detect when a player casts a spell using the action bar. The function UseAction(slot, checkCursor, onSelf) is responsible for activating abilities from the action bar.

We can hook it to print the action slot being used:

hooksecurefunc("UseAction", function(slot, checkCursor, onSelf)
    DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Used action slot: " .. slot)
end)

💡 Explanation:

    This will print a message every time the player clicks an action button.
    We don’t override the function—WoW still executes it as normal.

2. Hooking a UI Function: Detecting Frame Visibility

If we want to track when the character frame is opened or closed, we can hook ToggleCharacter like this:

hooksecurefunc("ToggleCharacter", function(frame)
    DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Character Frame Opened: " .. tostring(frame))
end)

💡 Use case: This is useful if your addon needs to react when certain UI elements are opened.
3. Tracking Target Changes

Another example: hook TargetUnit to detect when the player targets something.

hooksecurefunc("TargetUnit", function(unit)
    DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Targeted Unit: " .. tostring(unit))
end)

🔍 When would this be useful?

    Logging when the player targets something.
    Preventing unwanted target switching (by adding extra logic).

4. Detecting Money Spent on Repairs

We can track when the player repairs their equipment using the RepairAllItems function:

hooksecurefunc("RepairAllItems", function()
    DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Player repaired all items.")
end)

💰 Great for addons tracking gold usage!
5. Hooking a Function That Returns Values

hooksecurefunc does not allow modifying return values. If you need to modify behavior, you must use regular function hooking instead.

For example, the following will not work:

hooksecurefunc("GetMoney", function()
    return 9999999  -- This won't modify the actual return value!
end)

🔴 Solution: Use a manual function override instead:

local oldGetMoney = GetMoney  
function GetMoney()
    return 9999999 -- Overrides the function
end

⚠️ Warning: Overriding Blizzard functions like this can cause instability and is not recommended unless necessary.
When to Use hooksecurefunc vs. Overriding Functions
Situation	Use hooksecurefunc?	Override Function?
Logging function calls without modifying behavior	✅ Yes	❌ No
Tracking when UI elements are opened/closed	✅ Yes	❌ No
Hooking protected functions (e.g., UseAction)	✅ Yes	❌ No
Modifying a function’s return value	❌ No	✅ Yes (if allowed)
6. Advanced: Hooking Multiple Functions Dynamically

You can loop through a list of functions and hook them all dynamically:

local functionsToHook = { "UseAction", "TargetUnit", "RepairAllItems" }

for _, funcName in ipairs(functionsToHook) do
    hooksecurefunc(funcName, function(...)
        DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Function called: " .. funcName)
    end)
end

💡 Why is this useful?

    You can dynamically hook multiple functions without writing multiple hooksecurefunc calls.

Final Thoughts

🔹 hooksecurefunc is an essential debugging and tracking tool for WoW 1.12.1 addon developers.
🔹 It allows safely observing function calls without modifying game logic.
🔹 If you need to change return values, you must use function overrides instead.

Would you like more specific examples or deeper insights into function hooking? 🚀
You said:
it says hooksecurefunc is a Nil value
ChatGPT said:

You're absolutely right! In World of Warcraft 1.12.1 (Vanilla), hooksecurefunc does not exist yet. It was introduced in later expansions (TBC and beyond) to allow safe hooking of protected functions.

However, you can achieve similar functionality in Vanilla (1.12.1) using manual function hooking. Below, I'll explain how to manually hook functions safely, allowing you to track or extend existing functions without breaking protected ones.
1. Manually Hooking a Function (The Old-School Way)

Since hooksecurefunc is missing, we need to manually wrap Blizzard functions while preserving their original behavior.
Example: Hooking UseAction to Log Action Bar Clicks

-- Save the original function
local old_UseAction = UseAction

-- Create our hooked function
function UseAction(slot, checkCursor, onSelf)
    DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Used action slot: " .. slot)
    
    -- Call the original function
    old_UseAction(slot, checkCursor, onSelf)
end

🔹 What happens here?

    We store the original UseAction in old_UseAction.
    We create a new function with the same name.
    Our function prints a debug message, then calls old_UseAction to keep everything working normally.

2. Hooking a UI Function: Detecting When the Character Frame Opens

Since Blizzard UI functions are not protected in 1.12.1, we can override them safely.

-- Save the original function
local old_ToggleCharacter = ToggleCharacter

-- Override the function
function ToggleCharacter(frame)
    DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Character Frame Opened: " .. frame)
    
    -- Call the original function
    old_ToggleCharacter(frame)
end

📌 Use Case: This can be useful if your addon needs to react when the character frame is opened.
3. Hooking TargetUnit to Track Target Changes

You may want to track when the player changes targets. The function TargetUnit(unit) is responsible for switching targets.

-- Save the original function
local old_TargetUnit = TargetUnit

-- Create our hooked function
function TargetUnit(unit)
    DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Target changed to: " .. tostring(unit))
    
    -- Call the original function
    old_TargetUnit(unit)
end

🔎 When is this useful?

    If you want to log when the player switches targets.
    If you need to block target changes under certain conditions.

4. Tracking When a Player Repairs Items

We can track when a player repairs their gear by hooking RepairAllItems().

-- Save the original function
local old_RepairAllItems = RepairAllItems

-- Override the function
function RepairAllItems()
    DEFAULT_CHAT_FRAME:AddMessage("[HOOK] Player repaired all items.")
    
    -- Call the original function
    old_RepairAllItems()
end

💰 Great for addons tracking gold usage!
5. Hooking Multiple Functions Dynamically

Instead of writing multiple hooks manually, we can automate the process using a loop.

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

🚀 Why is this useful?

    You can easily hook multiple functions without writing repetitive code.
    This method automatically checks if the function exists before hooking it.

6. Can We Hook Protected Functions in Combat?

In Vanilla WoW, some functions are protected during combat (like CastSpellByName). If you override them, they may stop working properly.
Solution: Use Events Instead

Instead of hooking a protected function, you can listen for an event that triggers it.

For example, instead of hooking CastSpellByName, you can track when a spell is cast using UNIT_SPELLCAST_START:

local frame = CreateFrame("Frame")
frame:RegisterEvent("UNIT_SPELLCAST_START")

frame:SetScript("OnEvent", function(self, event, unit, spell)
    if unit == "player" then
        DEFAULT_CHAT_FRAME:AddMessage("[EVENT] Player is casting: " .. spell)
    end
end)

📌 This avoids modifying protected functions while still tracking spellcasting.
7. Debugging Hooked Functions

After hooking functions, you might want to check which ones are actually running.

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

👀 This method helps track function calls without modifying behavior.
8. Hooking Addon-Specific Functions

If you want to hook functions from another addon, use the same manual hooking method.

-- Example: Hooking a function from "MyOtherAddon"
if MyOtherAddon and MyOtherAddon.SomeFunction then
    local oldFunc = MyOtherAddon.SomeFunction
    MyOtherAddon.SomeFunction = function(...)
        DEFAULT_CHAT_FRAME:AddMessage("[HOOK] MyOtherAddon.SomeFunction called!")
        return oldFunc(...)
    end
end

📌 Why is this useful?

    Helps track addon conflicts or missing function calls.

Final Thoughts
🔹 What We've Learned

✔️ hooksecurefunc does not exist in 1.12.1, so we must use manual hooking.
✔️ Manually overriding functions allows us to track function calls.
✔️ If a function is protected, we can listen for events instead.
✔️ Dynamic hooking makes tracking multiple functions easier.
✔️ We can hook addon functions in the same way as Blizzard functions.
⚠️ Things to Watch Out For

❌ Never modify Blizzard's core functions without a backup.
❌ Some functions are protected in combat. Hook events instead.
❌ Too many debug messages may cause FPS drops—be selective with hooks!

Would you like deeper insights into a **specific function
You said:
could you make a dynamic function hook for all tracking of gold? 
ChatGPT said:

Certainly! In World of Warcraft 1.12.1, there are multiple ways a player can gain or lose gold, including:

    Looting mobs
    Selling items to a vendor
    Repairing gear
    Trading with other players
    Mailing gold
    Buying from the Auction House

Since hooksecurefunc does not exist in 1.12.1, we need to manually hook functions that affect gold transactions. Below is a dynamic function hook that tracks all gold-related transactions.
💰 Dynamic Gold Tracking Hook

-- Table to store original functions
local hookedFunctions = {}

-- Function to hook a Blizzard function dynamically
local function hookFunction(funcName, callback)
    if _G[funcName] then
        hookedFunctions[funcName] = _G[funcName]
        _G[funcName] = function(...)
            callback(funcName, ...)
            return hookedFunctions[funcName](...)
        end
    end
end

-- Function to detect changes in player gold
local lastGold = GetMoney()
local function checkGoldChange(reason)
    local newGold = GetMoney()
    local difference = newGold - lastGold
    if difference ~= 0 then
        local action = (difference > 0) and "Gained" or "Lost"
        DEFAULT_CHAT_FRAME:AddMessage("[GOLD TRACKER] " .. action .. " " .. math.abs(difference) .. " copper (" .. reason .. ")")
    end
    lastGold = newGold
end

-- Hook functions that involve gold transactions
hookFunction("RepairAllItems", function() checkGoldChange("Repairs") end)
hookFunction("UseContainerItem", function() checkGoldChange("Selling Item") end)
hookFunction("PickupMerchantItem", function() checkGoldChange("Buying from Vendor") end)
hookFunction("SendMail", function() checkGoldChange("Mail Sent") end)
hookFunction("PlaceAuctionBid", function() checkGoldChange("Auction House Bid") end)
hookFunction("PickupPlayerMoney", function() checkGoldChange("Trade") end)

-- Monitor loot gold gain through events
local frame = CreateFrame("Frame")
frame:RegisterEvent("PLAYER_MONEY")
frame:SetScript("OnEvent", function() checkGoldChange("Loot / Trade") end)

🔍 How It Works

✔ Hooks Blizzard functions that involve gold transactions.
✔ Checks if gold increased or decreased after each function call.
✔ Uses the PLAYER_MONEY event to catch gold changes from looting or trades.
✔ Prints messages in chat, such as:

[GOLD TRACKER] Gained 1500 copper (Loot / Trade)
[GOLD TRACKER] Lost 5000 copper (Auction House Bid)

✔ Avoids unnecessary spam by only displaying actual gold changes.
