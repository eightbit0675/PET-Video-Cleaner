PET VIDEO CLEANER — What It Is and Why It Exists
Revision A
Bruce Philip, 2026
aka "eightbit"

Acknowledgment

Special thanks to GK2001 from the VCFED forums for the input and explanation that led to this approach. The idea of using one half of a 74LS74 D-type flip-flop to re-clock the final video signal with the 8 MHz dot clock, so that short timing glitches are ignored until the next clock edge, comes from that discussion.


Disclaimer

This project is provided for informational and hobbyist use only. Build, install, and use this board entirely at your own risk.

Commodore PET computers contain old electronics and may also contain hazardous voltages, especially around the CRT monitor and power supply areas. Use proper safety precautions, work with the computer unplugged, and do not work near the CRT high-voltage circuitry unless you are qualified to do so.

I make no guarantee that this board will fix your PET, work with every PET revision, or prevent other faults from occurring. Incorrect assembly, incorrect orientation, soldering mistakes, wiring mistakes, bad sockets, bad ICs, power problems, or differences between PET board revisions could damage the computer, the daughterboard, the monitor, or other components.

By building, installing, or using this board, you accept all responsibility for the results. I am not liable for any damage, data loss, injury, repair costs, or other problems that may occur from using this design.

Compatibility Note

This board was designed and tested only for my Commodore PET board revision 320351. Other PET revisions may have different video circuitry, IC locations, pin orientation, connector routing, or board layout. Do not assume this daughterboard is compatible with another PET revision without comparing the schematic and motherboard layout first.

Purpose

The PET Video Cleaner is a small plug-in daughterboard intended for Commodore PET boards where the video output shows faint vertical lines, seams, or timing-related artifacts that appear to come from the digital video logic rather than from the monitor itself.

The board is not an amplifier, filter, or video brightness adjustment. It is a digital timing cleanup circuit. It re-clocks the PET's final logic-level video signal using the PET's own 8 MHz video clock so that short glitches in the video logic are not passed directly to the monitor.

This modification is intended to be reversible. The original UG11 74LS20 is removed from the PET motherboard socket and installed on the daughterboard. The daughterboard then plugs into the original UG11 socket. A single added wire connects the daughterboard's CLK8 pad to UE11 pin 2.


Design Background

This board addresses a timing artifact in the original PET video circuit. The original circuit works, but it does not fully re-time the final video signal after the pixel data, blanking, inverse-video, and video gating logic are combined. Because those signals pass through different TTL paths, they do not all arrive at the final video output at exactly the same time.

Commodore apparently considered the original design acceptable for this board revision, or the artifact was not objectionable enough under the monitors, tolerances, and expectations of the time. This daughterboard adds a timing cleanup stage that Commodore did not include on this PET board revision.

The board does not redesign how the PET generates video. It simply samples the completed video signal with the PET's own 8 MHz dot clock and outputs a stable, one-pixel-delayed version of that signal.

The Problem Being Addressed

In the PET video circuit, pixel data is shifted out of the video shift register and then passes through additional logic for inverse video, video enable, and blanking. Because these signals do not all change at exactly the same instant, the final video output can briefly glitch while the gates settle.

Those short glitches may only be tens of nanoseconds wide, but the monitor can still respond to them. On screen, they can show up as faint vertical artifacts, seams, or lines, especially around character boundaries or graphics patterns.

This does not necessarily mean that the original logic ICs are bad. It can be a timing artifact caused by normal propagation delay through the PET's video logic.


Relevant PET Video Signal Path

On the dynamic PET board this project was designed around, the useful simplified video path is:

    UE11 74LS165 video shift register
            ↓
    UG10 / UH10 video and inverse-video gating
            ↓
    UG11 74LS20 final video gate
            ↓
    UG11 pin 6 = final logic-level VIDEO output
            ↓
    J7 pin 1 to the monitor

UE11 is near the beginning of the pixel data output path. UG11 is near the end. UG11 pin 6 is the final combined digital video output before it goes to the monitor connector.

That is why the daughterboard intercepts UG11 pin 6 rather than modifying an earlier point in the circuit.


What the Daughterboard Does

The original UG11 74LS20 remains part of the circuit. Almost all of its pins pass straight through the daughterboard to the original motherboard socket.

The one exception is UG11 pin 6, the final video output.

Instead of allowing UG11 pin 6 to go directly back to the motherboard video line, the daughterboard sends it into one half of a 74LS74 D-type flip-flop.

The connection is:

    UG11 / 74LS20 pin 6
            ↓
    74LS74 pin 2, D input

    CLK8 from UE11 pin 2
            ↓
    74LS74 pin 3, clock input

    74LS74 pin 5, Q output
            ↓
    Original motherboard UG11 pin 6 connection

In other words:

    Raw final video from UG11 is sampled by the 74LS74.
    The 74LS74 only updates its output on the CLK8 clock edge.
    The cleaned video output is then sent back into the motherboard's original video path.


Why CLK8 Is Needed

CLK8 is the PET's 8 MHz video dot clock. It is the timing signal that controls when pixel data is shifted and when the next video bit should be valid.

The 74LS74 needs this clock because it is not being used as a simple buffer. It is being used as a re-timing device.

Without CLK8, short glitches on the raw video line can pass directly to the monitor.

With CLK8, the 74LS74 samples the video once per pixel clock and holds that value steady until the next clock edge. Any brief glitch that happens between clock edges is ignored by the output.

The tradeoff is that the video is delayed by one 8 MHz clock period. That is approximately one pixel. This may shift the displayed image very slightly, but it keeps the video signal aligned to the PET's own pixel timing.


Why UG11 Pin 6 Is Used

UG11 pin 6 is used because it is the final combined logic-level VIDEO signal. By the time the signal reaches UG11 pin 6, the PET has already applied the upstream video logic, including the relevant gating and inverse-video handling.

If the modification were installed earlier in the circuit, it could clean only part of the signal and still allow later gate-delay glitches to appear. Placing the 74LS74 after UG11 pin 6 means the completed video signal is re-clocked immediately before it returns to the monitor path.


Main Connections

The daughterboard has three main functional areas:

1. UG11 motherboard plug

This plugs into the original UG11 socket on the PET motherboard.

2. 74LS20 socket

The original UG11 74LS20 is moved from the PET motherboard into this socket.

3. 74LS74 socket

This is the added flip-flop used to re-clock the video signal.


Pass-through connections:

    P1 / motherboard UG11 pins 1–5  →  74LS20 pins 1–5
    P1 / motherboard UG11 pins 7–14 →  74LS20 pins 7–14

Intercepted video connection:

    74LS20 pin 6  →  74LS74 pin 2
    74LS74 pin 5 →  motherboard UG11 pin 6

Clock connection:

    CLK8 pad → 74LS74 pin 3
    External wire from CLK8 pad → UE11 pin 2

Power and ground:

    74LS74 pin 14 → +5V
    74LS74 pin 7  → GND
    C1 104 / 0.1 uF capacitor → between +5V and GND near the 74LS74

74LS74 control pins:

    Pin 1, /CLR1 → +5V
    Pin 4, /PRE1 → +5V
    Pin 10, /PRE2 → +5V
    Pin 13, /CLR2 → +5V

Unused second half of the 74LS74:

    Pin 11, CLK2 → GND
    Pin 12, D2   → GND
    Pins 8 and 9 → no connection

Unused output on the first half:

    Pin 6, /Q1 → no connection


Important Physical Orientation

For this board, all three 14-pin footprints are oriented with the notch pointing down when viewed from the top.

With the notch down:

    Pin 1 is at the bottom-right.
    Pins 2–7 run upward along the right side.
    Pins 8–14 run downward along the left side.
    Pin 14 is at the bottom-left.

Required Parts

    1 × PET Video Cleaner PCB
    1 × 74LS74 DIP-14 logic IC
    1 × DIP-14 socket for the 74LS20
    1 × DIP-14 socket for the 74LS74
    1 × 104 / 0.1 uF ceramic capacitor
    1 × set of machine-pin DIP-style pins or equivalent for the motherboard plug
    1 × insulated wire for CLK8 connection to UE11 pin 2

The original PET UG11 74LS20 is reused on the daughterboard.


Pre-Power Continuity Checks

Before installing the board in the PET, check the assembled daughterboard with a meter.

With no power connected:

    +5V to GND should not be shorted.

    P1 pin 1  → U1 pin 1  should have continuity.
    P1 pin 2  → U1 pin 2  should have continuity.
    P1 pin 3  → U1 pin 3  should have continuity.
    P1 pin 4  → U1 pin 4  should have continuity.
    P1 pin 5  → U1 pin 5  should have continuity.
    P1 pin 7  → U1 pin 7  should have continuity.
    P1 pins 8–14 → U1 pins 8–14 should have continuity.

    P1 pin 6 → U1 pin 6 should NOT have direct continuity.

    U1 pin 6 → U2 pin 2 should have continuity.
    U2 pin 5 → P1 pin 6 should have continuity.
    CLK8 pad → U2 pin 3 should have continuity.
    C1 should be connected between +5V and GND.


Installation Summary

1. Power off and unplug the PET.

2. Remove the original 74LS20 from UG11.

3. Install that original 74LS20 into the daughterboard's 74LS20 socket.

4. Install a 74LS74 into the daughterboard's 74LS74 socket.

5. Install the 104 / 0.1 uF capacitor at C1.

6. Plug the daughterboard into the original UG11 motherboard socket, matching the notch-down orientation.

7. Connect a wire from the daughterboard CLK8 pad to UE11 pin 2.

8. Re-check orientation and continuity before applying power.


Expected Result

If the artifact is caused by timing glitches in the PET's digital video logic, the daughterboard should reduce or eliminate those glitches by presenting a stable, clocked video signal to the monitor path.

The display may be shifted slightly by about one pixel because the 74LS74 delays the signal by one clock cycle. That is expected.


Limitations

This board will not fix every PET display problem.

It is intended for timing-related digital video artifacts. It will not repair:

    bad monitor circuitry
    poor brightness or focus adjustment
    analog video problems
    bad CRT components
    bad power rails
    defective RAM or character ROM
    unrelated video logic faults
    socket corrosion or broken traces

If the PET has unstable power, missing video, bad sync, monitor collapse, or major logic faults, those problems should be diagnosed separately.


Reversibility

The modification is intended to be reversible.

To return the PET to stock configuration:

1. Remove the CLK8 wire.
2. Unplug the daughterboard from UG11.
3. Reinstall the original 74LS20 directly into the UG11 motherboard socket.

