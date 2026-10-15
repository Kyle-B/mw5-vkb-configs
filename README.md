MW5 Configs for VKB Gladiator EVO
=================================
MW5 doesn't have the best support for HOTAS, here is how I've got things working reasonably well with absolute positioned aiming.

**Please be sure you back up your config**, you will need to swap back when going back to using it as a normal joystick. To get absolute aiming working, we'll convert the right joystick to be treated as a mouse.

One other feature implemented by this config is automatic torso centering when the stick returns to zero, this requires setting up a bind in-game.

Setup
-----

### VKB Device Config
[VKBDevCfg and firmware downloads can be found here.](https://vkbsimcontrollers.com/pages/downloads)

This guide and config was written using firmware `v2_20_C` and VKBDevCfg `v0.93.89`.

#### BACKUP YOUR CURRENT CONFIG
In `VKBDevCfg` select the joystick, and from the `Actions` tab, press `Save`. This will export your current joystick setup to be restored later.

#### Load the prepared joystick config
*If you'd rather set things up yourself, or if you have a different model and can't directly use the EVO settings see the section at the end.*

In `VKBDevCfg` select the joystick, and from the `Actions` tab, press `Load`. Select `VKBsim-Gladiator-EVO-R-AbsMouse-MW5.cfg` from this repository.

**IMPORTANT: You need to adjust the mouse settings to match your monitor dimensions.**

This can be done by going to the `Global` tab (bottom left of the window). In there is the `Mouse` section, on the right hand side is a `Width` and `Height` that you can set by clicking the `SCREEN` button.

Press `Set` from the `Actions` tab to write this to the joystick. You should also press `Save` and overwrite `VKBsim-Gladiator-EVO-R-AbsMouse-MW5.cfg` to have your monitor resolution when loading these settings in the future.

### MW5 HOTAS Setup

#### Setup Remap
Place `HOTASMappings.Remap` into `%LOCALAPPDATA%\MW5Mercs\Saved\SavedHOTAS`

If your throttle isn't using the Omni-Throttle (OT) firmware, you will need to change the `NAME:` and `PID:` lines on the throttle to match what you see in VKBDevCfg.
The OT and normal firmware only have different default deadzones and no other functional difference, but the device does show up differently.

The `AXIS:` directives in this file are where you can tweak deadzones and inversions if you need to.

#### Enable automatic torso centering
In the Joystick controls menu, bind "Center Torso Over Legs" to En1 Up (right hand encoder on the base of the stick, scrolled up). This will show up in game as `Joystick 8`.

This button will fire once when the stick is centered. This is slightly buggy, but works the majority of the time and will help resolve any drift automatically. If it doesn't seem to fire, move the joystick out from center and let it reset.

#### Bind the rest of your controls
A normal HOTAS setup would be:
```
Leg Rotation Axis: Throttle Axis 2
Throttle Axis: Throttle Axis 1
```
The `Look` Axis are unused, because our joystick is pretending to be a mouse.

Bind the rest of the controls to buttons of your preference. Just remember that "Joystick 8" is in use, if you'd like to change which button is used for the centering, see the details below.

### Test it out!
At this point you should be good to go, fire up an Instant Action and adjust your binds or axis as needed.


## VKB Configuration Details
If you don't have an EVO + Omni-Throttle or want to tweak how the configuration works, here is what I've set in VKBDevCfg.

### Enabling Absolute Mouse Positioning
#### Global->Mouse Section
This enables the joystick to output as a mouse, but doesn't setup anything to actually emit mouse events yet.
```
Mouse: Absolute
Active: Always On
Width/Height: (Press SCREEN to set to your monitor dimensions)
```

#### Profile->Logical Axes (Upper section)
We'll turn off the X & Y axis at the "Visible" level, this allows us to setup the mouse movement and avoid any risk of confusion when configuring binds in game. The joystick still processes the axis, but doesn't report it at the HID level.
```
Uncheck Vs (Visible in HID) for 1 & 2 (X & Y)
```

#### Profile->POVs
We now need a new "POV hat" that outputs the mouse movements, we'll bind this to the X & Y axes. This can be any free POV slot you'd like, I'm using `#2`:
```
POV Type: Common
Output: Mouse Abs
VC#: 1
2D: Checked
X: 1
Y: 2
DZ: 5, Cross (Personal preference, I haven't messed with this much)
Active: Always
```

#### Test
With these settings you should be able to `Action->Set` to write to the device and it will work as a mouse. Your EVO R should appear at the top as `HID Composite Device : [2]` now, indicating that it's operating as both a mouse and a joystick.
If it's working at this point, it should be fine for use in the game, but the next section is a large QoL upgrade.

### Setting up auto torso centering
If you test the settings up above in game you may notice occasionally you'll get stuck off center when the stick has returned to the deadzone. We can solve this by using the "Center Torso Over Legs" bind, but it'd be nice if this could fire when the stick returns to 0 on it's own. Turns out we can do this entirely in VKBDevCfg.

The basic idea is we'll fire a button when the both of the stick axes are ~0, and then bind that in game to "Center Torso Over Legs". Due to how the Remap file works and the limited number of allowed buttons by MW5, it's easiest to take over an unused button for our purposes. I'm not using the encoder `En1` for anything, so here I'll be using `En1 Up` (VKB Button 23, In-game button 8), you can use any other unused button you'd like, just make sure to adjust all the configs as needed.

This is done using Booleans, we'll setup 2x`Comparator 2`s and 1x`2 And`.

#### Profile->Boolean
Double click an empty square to pull up the functions window. The first one should be configured like:
```
Main/Center top dropdown: Comparator 2

Layer: Virtual
Input: 1, 1, 1
Inv: Unchecked, Unchecked, Checked
Hysteresis %: 0, or 0.1 (I've used 0)

Output/Right Dropdown: Virtual
VBut N: 121 (this can be any unused button)

 V1--|A|><|
 1%--|2|  |--V121
-1%--|V|  |
```
Press the Green Copy button in the center, and then press OK.

Open up another free slot in the grid, press the Green Paste button and change the values:
```
First input: 2

Output VBut N: 122 (another free button)

 V2--|A|><|
 1%--|2|  |--V122
-1%--|V|  |
```
Press OK and then open up another slot in the grid for the final `2 And`:
```
Main/Center top dropdown: 2 And

Layer: Virtual
But N: 121, 122

Output/Right Dropdown: Virtual
VBut N: 123

Bottom right section:
Checked, Pulse selected in dropdown
Time, mS: 3
MPL: x10

V121--|  & |
      |   ⎍|--V123
V122--|    |
```

Press OK

#### Profile->Buttons->Logical layer
At this point if you use "Set" to write this config to the device and look at the buttons in the Test tab you should see `121` and `122` lighting up while the stick is centered. If you move the stick away from center and release, `123` should flash as the stick returns to center. We'll now setup 123 to fire the same button as `En 1 Up` so it works with MW5's remap and is bindable in game.

Double click 123 in the logical layer grid (bottom row, 3rd column).
Uncheck "AutoMapping"
Change the BUT1 field to 23 (En1 UP).

### Test
Use `Action->Set` to write to the device. Now you should be able to see 121 and 122 working as before, but now 23 will flash when the stick returns to center. Using the encoder will still work and flash 23 when scrolling up as well. We can use this to bind it to "Center Torso Over Legs" in game, and now you shouldn't get out of sync while playing the game!

Make sure you save the new config so you can swap between them later.

### Limitations
For some reason the booleans don't always trigger as they should, and you'll still end up slightly off center. Wiggling the stick and letting it return to center will cause it to reset, which was good enough for me. If someone with more VKBDevCfg knowledge than me knows a better way to do this, I'd be happy to hear it.