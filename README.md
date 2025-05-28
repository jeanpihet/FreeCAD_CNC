# FreeCAD_CNC
Scripts and documentation about doing open source CNC with FreeCAD

## Install FreeCAD
Use a precompiled version of freecad from github.
https://github.com/FreeCAD/FreeCAD/releases

Version >= 1.0 is required for stability and availability of the CAM module.

## Install the Marlin post processor

- Copy marlin_post.py from the scripts dir to ~/.FreeCAD/Macro/
- marlin_post.py is edited with:
  
  . 'POSTAMBLE.replace("\\n","\n").splitlines(True):' for PREAMBLE and POSTAMBLE.
 
  . UNIT_FEED_FORMAT = 'mm/min'
 
  . changes for rapid rate (G0)

- Default post processor settings

  . Edit -> Preferences -> CAM -> Job Preferences -> General
  
   . Path: ~/.FreeCAD/Macro/
  
  . Edit -> Preferences -> CAM -> Job Preferences -> Post Processor
  
   . Default post processor: marlin

   . Default Arguments
```
 --preamble "G90\nG21\nG92 X0 Y0 Z0 (This is your new home)\nG0 F800 Z3\nM0 Originpart Ok\n" --postamble "G0 F800 Z5\nM5 (Spindle off)\nG0 F800 X0 Y0\nM400 (Finish moves)\nM300 S1000 P2000 (Beep)\nG17 G90\n"
```

## Engrave from sketch

0. Create a sketch in FreeCAD

1. To start from a PDF, convert it to SVG in Inkscape

  - ! Close paths
  
  - ! Join duplicate nodes

2. New project. Part Design-> Create body

3. Import SVG in Freecad, from the Draft module, as geometry

4. From the Draft module, convert the path to sketch with Modification -> Draft to sketch. The original paths can be deleted

5. Extrude the sketch. Part->Extrude 0.2mm. Z axis, Symmetric, Create Solid, Reverse

6. In case of embedded path (paths in paths), use Part->Boolean to keep only the engraving

7. Create a stock part (cube):
   . Part Design-> New sketch; draw rectangle
   . Pad. ! Reverse = true to go -Z

8. Move the parts below the Z axis, check that parts are aligned ok on Z = 0

9. Extract the engraving from the stock: Part -> Boolean Cut. The result is the stock with the engraving solid substracted

10. Select engraving solid

11. CAM -> Job, check that the engraving solid is selected

12. Add tool from the Toolbit dock; check travel speed and spindle speed (!= 0 for the spindle to be turned on), etc.

13. In Job, check parameters in SetupSheet: Final Depth = -0.2mm; Start Depth = 0; Step Down = 0.1mm

14. CAM -> 3D Pocket, check depths, finish step, heights

  - ! Finish depth != 0, = 10nm from the final depth (?)
    
  - ! For round holes, use Spiral pattern

15. CAM -> Post process to save the G-code

## Start CNC

  - position part

  - set tool to piece reference (this will be the new origin after G92)

  - launch gcode from SD card

## G-code details
preamble
```
(*********** set 0, move spindle up before turning it on ***********)
G92 X0 Y0 Z0 (This is your new home)
G0 F800 Z3
M0 Originpart Ok
```

postamble
```
(*********** spindle off, reset to 0, move spindle up to remove the part ***********)
G0 F800 Z5
M5 (Spindle off)
G0 F800 X0 Y0
M400 (Finish moves)
M300 S1000 P2000 (Beep)
```

## Tools settings for material

### Plexi

- Endmill 3mm bit
- Fast 840 mm/min = 14 mm/s
- Normal 240 mm/min = 4 mm/s
- Step down 1-1.25 mm

### Zinc

- Engrave 0.1mm bit
- Fast 840 mm/min = 14 mm/s
- Normal 240 mm/min = 4 mm/s
- One pass 0.10 - 1 mm

### Brass

- Engrave 0.1mm bit
- Fast 800 mm/min = 13.33 mm/s
- Normal 100 mm/min = 1.66 mm/s
- Step down 0.06 x2 + finish 0.03 = 0.15mm
