---
layout: page
title: 3D Printing
permalink: /3dprinting/
---

# what's here?
These are my personal notes about 3D printing.

# jens' ender-3 profile for PLA from OWL-Sat

[jens_owl_pla.ini](http://snej.de/downloads/jens_owl_pla.ini)

    Quality 
        Layer height (mm)                   0.15
        Shell thickness (mm)                1.2
        Enable retraction                   TRUE
        Initial layer thicknes (mm)         0.2
        Initial layer line width (%)        100
        Cut off object bottom (mm)          0
        Dual extrusion overlap (mm)         0.15

    Retraction
        Minimum travel (mm)                         1.5
        Enable Combining                            Yes, on all surfaces.
        Minimal extrusion before retracting (mm)    0
        Z hop when retracting (mm)                  0
        Retraction Speed (mm/s)                     60
        Retraction Distance (mm)                    6

    Fill
        Bottom/Top thickness (mm)           1.2
        Fill Density (%)                    15
        Solid infill top                    TRUE
        Solid infill bottom                 TRUE
        Infill overlap (%)                  15
        Infill prints after perimeters      FALSE

    Speed and Temperature
        Print speed (mm/s)                  50
        Printing temperature (C)            195
        Bed temperature (C)                 50

    Support
        Support type                        None
            Structure type                      lines
            Overhang angle for support (deg)    60
            Support Fill amount (%)             15
            Support Distance X/Y (mm)           0.7
            Support Distance XZ (mm)            0.15
        Platform adhesion type              None
            Skirt (selected when Platform adhesion type is 'None')
                Skirt Line count                    5
                Skirt start distance (mm)           10
                Skirt minimal length (mm)           150
            Brim
                Brim line amount                    20
            Raft
                Extra margin (mm)                   5
                Line spacing (mm)                   3
                Base thickness (mm)                 0.3
                Base line width (mm)                1
                Interface thickness (mm)            0.27
                Interface line width (mm)           0.4
                Airgap                              0
                First Layer Airgap                  0.22
                Surface layers                      2
                Surface layer thickness (mm)        0.27
                Surface layer line width (mm)       0.4

    Filament
        Filament Diameter (mm)          1.75
        Flow (%)                        100

    Machine         
        Nozzle size (mm)                0.4

    Speed
        Travel speed (mm/s)             60
        Bottom layer speed (mm/s)       20
        Infill speed (mm/s)             0
        Top/bottom speed (mm/s)         0
        Outer shell speed (mm/s)        35
        Inner shell speed (mm/s)        35

    Cool
        Minimal layer time (sec)        5
        Enable cooling fan              TRUE
        Fan full on at heigth (mm)      0.5
        Fan speed min (%)               100
        Fan speed max (%)               100
        Minimum speed (mm/s)            10
        Cool head lift                  FALSE

    Black Magic
        Spiralize the outer contour     FALSE
        Only follow mesh surface        FALSE

    Fix horrible
        Combine everything (Type-A)     TRUE
        Combine everything (Type-B)     FALSE
        Keep open faces                 FALSE
        Extensive stitching             FALSE

    Plugin "Pause at height" (Pause the printer at a certain height)
        Pause height (mm)               5
        Head park X (mm)                190
        Head park Y (mm)                190
        Head park Z (mm)                0
        Retraction amount (mm)          5

    Plugin "Tweak At Z 4.0.2" (Change printing parameters at a given height)
        Z height to tweak at (mm)       5
        No. of layers used for change   1
        Tweak behavior                  Tweak value and keep it for the rest
        all other settings empty

    start.gcode
        ;Sliced at: {day} {date} {time}
        ;Basic settings: Layer height: {layer_height} Walls: {wall_thickness} Fill: {fill_density}
        ;Print time: {print_time}
        ;Filament used: {filament_amount}m {filament_weight}g
        ;Filament cost: {filament_cost}
        ;M190 S{print_bed_temperature} ;Uncomment to add your own bed temperature line
        ;M109 S{print_temperature} ;Uncomment to add your own temperature line
        G21        ;metric values
        G90        ;absolute positioning
        M82        ;set extruder to absolute mode
        M107       ;start with the fan off
        G28 X0 Y0  ;move X/Y to min endstops
        G28 Z0     ;move Z to min endstops
        G1 Z15.0 F{travel_speed} ;move the platform down 15mm
        G92 E0                  ;zero the extruded length
        G1 F200 E3              ;extrude 3mm of feed stock
        G92 E0                  ;zero the extruded length again
        G1 F{travel_speed}
        ;Put printing message on LCD screen
        M117 Printing...

    end.gcode
        ;End GCode
        M104 S0                     ;extruder heater off
        M140 S0                     ;heated bed heater off (if you have it)
        G91                                    ;relative positioning
        G1 E-1 F300                            ;retract the filament a bit before lifting the nozzle, to release some of the pressure
        G1 Z+0.5 E-5 X-20 Y-20 F{travel_speed} ;move Z up a bit and retract filament even more
        G28 X0 Y0                              ;move X/Y to min endstops, so the head is out of the way
        M84                         ;steppers off
        G90                         ;absolute positioning
        M81
        ;{profile_string}


