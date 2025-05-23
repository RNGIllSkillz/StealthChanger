Toolchangers start with number 0, and count up. So for all the configs, make sure you change all sections. IE, in your T1 config, make sure its extruder1, fan1, etc. See the examples.  **NOTE: The only except to this rule is T0 extruder has no number.**

1. [Toolhead Configuration](#toolheads-configuration)
2. [CANbus](#canbus)
3. [Offsets](#offsets)
4. [Dock Positions](#dock-positions)
5. [close_y and safe_y](#close_y-and-safe_y)
6. [Docking Path](#Docking-Path)

## Toolheads Configuration

Please see the repo files for more info on these. [Configuration Files Repo](https://github.com/DraftShift/StealthChanger/blob/main/Klipper)

You need to add the info in `printer.cfg` from our repo to your `printer.cfg`

You need to have a separate toolhead config for each toolhead, then link those in your `printer.cfg`, as well as removing/moving the current extruder/hotend config from your `printer.cfg` file. See the examples in the repo for more information and a starting place. You will have to edit them with your own values.

**NOTE:** for some odd reason `t_command_restore_axis` was set to blank recently, this should be `t_command_restore_axis: z` please make sure that is your setting.

**NOTE:** `_PARK_ON_COOLING_PAD` is a useless macro in `PRINT_START` in `macros.cfg` please make sure to comment out or remove that line, currently it is line 129.

## CANbus

[Esoterical CANbus](https://github.com/Esoterical/voron_canbus) this is a pretty definitive guide for canbus implementation on klipper.

**Example CANbus layout**

![Example CANbus Layout](https://github.com/DraftShift/StealthChanger/blob/main/media/can_example.jpg?raw=true)

## Offsets

There are 2 places to set offsets in your toolhead config files. There is gcode_(x/y/z)_offset in the [tool] section, and (x/y/z)_offset in the [tool_probe] section. These do different things. The ones in the [tool] section are always relative to a specific tool. IE, if you homed with T0, then the gcode offsets are relative to T0. When you do multi-material prints, you will have to choose 1 tool to always do the homing, even if you don't use it, because all the other offsets need to be set against it. The [tool_probe] offset is only ever applied when homing with that tool. The gcode offsets are applied when changing a tool.

You should set all the `[tool_probe]` offsets as well though. If you are only using a single tool, you can home with that tool, and it will use the offset in the `[tool_probe]` section. You won't need to home with your primary tool first then before you start a print. 

To try and further clarify. Lets say you have 3 tools, T0, T1, and T2. We will say that T0 is your "primary" tool. Setup the probe z_offset like you would any other printer. Make sure the offset is set in `z_offset` (IE, make sure when you home and go to Z 0, the nozzle is actually in that spot). Its location in x/y/z is always going to be 0 in relation to the other tools. So after T0 is homed, T1 and T2 (may) need to have their `gcode_(x/y/z)_offset` changed so they match the location that the primary tool actually is. There are 2 main ways of doing this. You can use a test print to print layers on top of or next to each other (something like this: [Nozzle Alignment Assist](https://www.printables.com/model/109267-nozzle-alignment-assist)) or you can use a camera (something like [kTAMV Klipper Tool Alignment](https://github.com/TypQxQ/kTAMV) or [IDEX Nozzle Calibration Tool](https://github.com/Life0fBrian/Brians-IDEX-Nozzle-Calibration-tool)).

## Dock Positions

When setting your dock position in your tool config, X and Y are pretty self explanatory. Put the tool in the right spot for it to sit. However, Z should be the point where 1 more mm down will untrigger the tap module. 

## close_y and safe_y

It is important to get this right. If you don't, you will have changing issues, or crash your tools into your docks. This is where we recommend you start:

- close_y = park_y + 30
- safe_y = park_y + thickness of your thickest tool (plus a little buffer)

IE, if you Y park position is -15. Your close Y should be 15. If your Y park position is 0, it should be 30. Safe Y should be slighty further out. 

## Docking Path

The docking path can be confusing for people to understand. Let's try to disect it. Take this path:

params_sc_path: [{'y':9.5 ,'z':4}, {'y':9.5, 'z':2}, {'y':5.5, 'z':0}, {'z':0, 'y':0, 'f':0.5}, {'z':-10, 'y':0}, {'z':-10, 'y':16}]

When looking at this, understand that the movements are relative to the park position. When dropping off a tool, it will go left to right, and when picking up a tool, it will go right to left on movements. Let's say your park position is x=20 y=-10, z=240 and close_y=20. When dropping off a tool, it will move up and start the sequence essentially at x=20 y=20 z=240. Now add in the first set of items. 

*  {'y':9.5 ,'z':4} Move to x=20 y=-0.5 z=244
*  {'y':9.5, 'z':2} Move to x=20 y=-0.5 z=242
*  {'y':5.5, 'z':0} Move to x=20 y=-4.5 z=240
*  {'z':0, 'y':0, 'f':0.5}. This will slowly move you (f is speed in gcode) to docking position of x=20 y=-10, z=240
*  {'z':-10, 'y':0} Drop the z down to z=230 so that it starts unhooking.
*  {'z':-10, 'y':16}. Move to z=230 y=6, we are clear of the tool.

It will then move to the close_y and x of the tool its picking up, and run in reverse order.
