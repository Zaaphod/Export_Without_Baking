# Export_Without_Baking
This is a Grasshopper Script to Export geometry to various colored layers in various file formats directly without baking geometry to Rhino.
There is a discussion about this here: https://discourse.mcneel.com/t/c-script-to-export-to-colored-layers-from-grasshopper-without-baking/164356?u=james_richters

What I am trying to accomplish with this script is an easy way to export various geometries in different colors and on different layers from Grasshopper without baking anything. It uses the Grasshopper trees to keep things organized
just attach the geometry for each layer, give the layers names and colors and export!

There is a bug in Grasshopper when using the export command that I have not been able to solve completely, but I at least identified it.
if a Grasshopper C# script contains:
```
Rhino.RhinoApp.RunScript("_-Export " + exportOptions + " \"" + filepath + "\"" + " _Enter", true);
```
when you run the script with whatever input you use to enable the script, if that input goes back to false before the Export finishes, then it leaves Grasshopper in this very odd state where it’s locked and you can’t do anything with anything. you can’t move components, or edit them or anything, it’s completely locked. Strangely, you can run the script again, and if you do, it will always unlock. I’ve been trying to figure it out and its defiantly only Rhino.RhinoApp.RunScript("_-Export " that causes this (that I know of)
The larger the file you are trying to export, the more you notice this.

I have tried a LOT to keep the input active trying to implement timers and all kinds of schemes to work around this bug, but in the end, the way Grasshopper’s solver works, Timers and things like that just won’t work. I think that is why whenever you see a script like this it’s connected to a Boolean Toggle,
because with the toggle, it stays true until you toggle it back. but the toggle is also a pain. first of all, you need to double click it… that’s a pain, also you MUST remember to shut it back off of it will export every time you make a change. so that’s another pain.

So, I tried something else… an Analog input!

since it takes time to rotate the knob all the way around, and all the way back, the input is kept active until the Export is finished… you are the timer!

so, this solves another issue I was having as well. I wanted to open a Rhino Document, process geometry from the document then extracts the document name and export a new file in a new directory with the same name, but with " - Processed" appended to the end of it. The problem is, when you just load a new document into Rhino, Grasshopper doesn’t run the scripts that could get the new file name, the AnalogInput solves this issue as well, and in fact give you a way to check the way the exported filename will be assembled and give you a chance to look at the name before doing the export.

So you start with the minimum value on the Analog Input and as you rotate it, it builds the export filename:

I tried to make this a universal script, so you can give it your own names or let it get the Rhino document names as needed. I tried to set default values for everything so it would work easily and let you tweak it as needed. Rotating it more, I get the Rhino Document name because I don’t have the filename input connected

then as you rotate it, it prepends as and/or appends to the file name, adds the extension and checks to see if you ended up with the same name and path as your Rhino file, and if so, it adds “- Exported” to it so you don’t overwrite your file.

if you like the exported path and filename, you just keep rotating it right until it exports, if you don’t like it, rotate all the way left and start over. Note that once you start the export you need to leave the knob all the way to Maximum until it’s done otherwise you still get the Grasshopper lockout problem.

After I got that working, I thought that it would be cool to also use the Boolean Button… so I made it autodetect the Button or the Knob! to do this, I made the knob in the range of 2.55 to 3.45 that prevents it from having a 0 or a 1, so 0 and 1 can be a digital input on the same pin… I also default the input to -1 to mean it’s disconnected.

So when you are setting it up, connect the knob, turn it slowly, make sure you like the directories and filename options… once you get it all working right, switch it out to the button and just remember to hold it down until the export finishes.

Here’s a description of the inputs and outputs:
Geometeries - where you connect a tree of Geometry
LayerNames - where you connect a tree of layer names
LayerColors - where you connect a tree of layer colors
Directory - the directory to export into the \ at the end is optional… it will come out right either way
FilePrepend - something added to the beginning of the file name, intended for use when you are extracting the rhino filename
Filename - what you want the filename to be, if you leave it blank it will use the Rhino document name, if there is no Rhino document loaded it will make use a default name if this input is not attached.
Extension - the extension of the exported file
ExportOptions - a string of options for use with the _-Export command the same as you would use on the Rhino command line.
RhinoCommands - a list of Rhino Commands to run on the geometry right before exporting, I often need to _Explode things, _Convert, and _SimplifyCrv, this gives a way to do those processes right before export without editing the script itself.
AnalogTrigger - This is the input to trigger the Export, you can connect a Boolean like the Boolean Toggle or the Button, but keep in mind you need to hold the button down until the script finishes or Grasshopper will get locked up. You can also connect an Analog input like the Control Knob or the Number Slider. Analog inputs need to be in the range of 2.55 to 3.45 (this is what looked best with the Control Knob. they are 0.05 away from the bottom position in each direction.) the reason they are in this range is that it keeps the Analog input away from 0 and 1 which and it gives me a way to detect what you have connected to it and removes ambiguity.
out - is the default c# output for compiler messages.
Messages - are messages I write out as the script is executing, these are also on the command line, but the Messages output is a bit easier to read and I clear it as needed to keep it from getting too cluttered.
ExportDirectory - The directory the file will be exported to, this populates at when the Directory is processed in the script as the Analog value increases.
ExportFilename - The filename without extension that will be exported to, this changes as the analog value increases, and it adds on the Prepend and Append. etc…
ExportExtension - is the extension that will be exported to.
ExportFile - is a preview to the entire full path of the file to be exported to.
