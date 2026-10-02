# SerpentUI

A glassmorphism UI library for Roblox client scripts, made by **rayware**. Build your own menu with themed controls, smooth transitions, and a compact floating window.

## Features

- Dark glass surfaces with a red-orange default theme.
- Main and Settings sidebar navigation with theme-colored icons.
- Animated loading screen inside the window.
- Draggable window and compact floating mode with bounce animations.
- Mouse and touch input support.
- Toggles, buttons, sliders, text inputs, labels, and sections.
- Built-in theme, glass transparency, and animation speed settings.
- Demo mode and a reusable API for custom menus.

SerpentUI builds the interface. You provide the game-specific logic through callbacks.

## Try the demo

Run this to open the built-in example menu:

```lua
loadstring(game:HttpGet("https://raw.githubusercontent.com/rayware/SerpentUI/refs/heads/main/ui"))()
```

The demo includes test controls. It does not change gameplay.

The loader requires a client environment that supports `loadstring` and `game:HttpGet`. It is not a drop-in loader for an ordinary Roblox Studio LocalScript. Compatibility depends on the environment; universal compatibility is not guaranteed.

## Create your own menu

Pass `{Demo = false}` to load the library without opening its demo. You do **not** need to remove the demo code or edit the library file.

This complete example creates a custom window:

```lua
local Library = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/rayware/SerpentUI/refs/heads/main/ui"
))({Demo = false})

local Window = Library:CreateWindow({
    Id = "MyScript",
    Title = "My Hub",
    Subtitle = "by your_name",
    Theme = "Default",
    Width = 460,
    Height = 390,
})

Window:AddSection({Name = "Features"})

local Status = Window:AddLabel({Text = "Ready"})

local Feature = Window:AddToggle({
    Name = "My feature",
    Default = false,
    Callback = function(enabled)
        Status:SetText(enabled and "Feature enabled" or "Feature disabled")
        -- Start or stop your feature here.
    end,
})

Window:AddButton({
    Name = "Test notification",
    Callback = function()
        Window:Notify("Hello from SerpentUI!", 3)
    end,
})

local selectedValue = 35
local Slider = Window:AddSlider({
    Name = "Value",
    Min = 0,
    Max = 100,
    Step = 1,
    Default = selectedValue,
    Callback = function(value)
        selectedValue = value
        Status:SetText("Value: " .. value)
        -- Use selectedValue in your own feature.
    end,
})

local NameInput = Window:AddInput({
    Name = "Window title",
    Placeholder = "Enter a name",
    Default = "My Hub",
    Callback = function(value, enterPressed)
        if value:match("%S") then
            Window:SetTitle(value)
        end
    end,
})
```

Change `Title` and `Subtitle` to brand your menu. Change each control's `Name` and `Callback` to add your own behavior. The example slider only stores a value; it does not change player speed.

Use a unique `Id` for each independent window. Creating another window with the same `Id` closes and replaces the previous one.

## Controls

All controls accept `Tab = "Main"` or `Tab = "Settings"`. The default is `Main`.

| Method | Options |
| --- | --- |
| `Window:AddToggle({...})` | `Name`, `Default`, `Callback(enabled)`, `Tab` |
| `Window:AddButton({...})` | `Name`, `Callback()`, `Tab` |
| `Window:AddSlider({...})` | `Name`, `Min`, `Max`, `Step`, `Default`, `Callback(value)`, `Tab` |
| `Window:AddInput({...})` | `Name`, `Placeholder`, `Default`, `Callback(value, enterPressed)`, `Tab` |
| `Window:AddLabel({...})` | `Text`, optional `Height`, `Tab` |
| `Window:AddSection({...})` | `Name`, `Tab` |

A slider requires `Max > Min` and `Step > 0`. Its values are rounded to the step and clamped to its range.

Input callbacks run when the text field loses focus. `enterPressed` indicates whether Enter submitted it; programmatic `SetValue` calls do not supply that second argument.

Toggle, slider, and input defaults do **not** trigger callbacks on creation. Initialize your own state to match the chosen defaults.

### Update controls from code

These examples use the handles created in the complete example above:

```lua
Feature:SetValue(true)        -- Update and call its callback if changed.
Feature:SetValue(false, true) -- Update silently.
print(Feature:GetValue())

Slider:SetValue(60)
print(Slider:GetValue())

NameInput:SetValue("New title")
print(NameInput:GetValue())

Status:SetText("Updated!")
```

Every returned control handle supports `:Destroy()` to remove that control. Stop any feature it started before removing its control: destroying the control does not call its callback with `false`.

### Add content to Settings

Appearance settings are already included. You can append your own controls:

```lua
Window:AddButton({
    Name = "About this script",
    Tab = "Settings",
    Callback = function()
        Window:Notify("Made by your_name", 3)
    end,
})
```

This version has exactly two tabs: **Main** and **Settings**. Custom tabs and `AddTab` are not currently supported.

## Themes

| Name | Appearance |
| --- | --- |
| `Default` | Dark glass with red-orange accents |
| `Red` | Dark glass with red-pink accents |
| `Blue` | Dark glass with blue-cyan accents |
| `Yellow` | Dark glass with yellow-gold accents |
| `Green` | Dark glass with green accents |
| `White` | Light glass with dark text |
| `Black` | Near-black glass with silver accents |

Choose a theme in Settings or change it from your script:

```lua
Window:SetTheme("Blue")
```

Names are case-sensitive. The library's glass effect uses translucent GUI layers, gradients, borders, and highlights. It does not apply a fullscreen blur. Settings are not saved between runs in this version.

## Window methods

| Method | Action |
| --- | --- |
| `Window:SetTitle("My Hub")` | Update the main and floating-window titles |
| `Window:SetTheme("Default")` | Apply a supported theme |
| `Window:SelectTab("Settings")` | Switch tabs while the main window is open and idle |
| `Window:SetMinimized(true)` | Collapse into the draggable floating window |
| `Window:SetMinimized(false)` | Restore the main window |
| `Window:Notify("Ready", 3)` | Show a temporary message in the main window's footer |
| `Window:OnDestroy(callback)` | Register cleanup for closing or replacing the window |
| `Window:Track(connectionOrFunction)` | Register a connection or cleanup function for disposal |
| `Window:Destroy()` | Close with an animation and dispose of the window |
| `Window:Destroy(true)` | Dispose immediately |

Tab and minimize requests are ignored while the intro or another transition is running. Footer notifications are not separate popups and are not visible while minimized.

## Stop continuous features properly

A toggle does not automatically stop loops or connections created by your callback. Keep a reference to your work and stop it on disable and window destruction.

Add this after creating `Window`:

```lua
local RunService = game:GetService("RunService")
local connection

local function stopFeature()
    if connection then
        connection:Disconnect()
        connection = nil
    end
end

Window:AddToggle({
    Name = "Continuous feature",
    Default = false,
    Callback = function(enabled)
        stopFeature()

        if enabled then
            connection = RunService.Heartbeat:Connect(function(deltaTime)
                -- Put your repeated action here.
                -- Keep this callback short and do not yield.
            end)
        end
    end,
})

Window:OnDestroy(stopFeature)
```

For a connection that should last for the entire window's lifetime, use `Window:Track(connection)`. Restore any game properties changed by your features in your own cleanup code. Keep cleanup callbacks short and avoid yielding.

## Troubleshooting

- **The demo appears instead of my menu:** load with `({Demo = false})`, then call `Library:CreateWindow(...)`.
- **Nothing appears with `Demo = false`:** this mode only returns the API. You still need to create a window.
- **A control does nothing:** put the action in its `Callback`. SerpentUI does not implement your game logic.
- **A feature keeps running after disabling it:** stop its loop or disconnect its connection in the `false` branch.
- **A callback fails:** check the client console for `[SerpentUI callback]` and the error message.
- **The loader fails:** check the Raw URL and whether your environment supports the loader APIs.
- **An obfuscated copy fails:** verify that the obfuscator preserves `...`, the returned library table, and the public method names. Obfuscator compatibility has not been established.

When reporting a UI issue, include your device, a screenshot or video, the console error if available, and a minimal script that reproduces it.

---

**SerpentUI — made by rayware.**
