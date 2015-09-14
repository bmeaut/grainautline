---
layout: default
---

# GrainAutLine user manual

This documentation is a summary of the functions of GrainAutLine. They are divided into the following categories:

* Application control: menus, elements of the user interface
* Global functions: functions which are always available, independently of the currently selected processor.
* Processors: special functions which can be selected and activated

Before going on, we will have to clarify the Processing State Descriptor concept, which is the principal representation of the work you are doing.)

## The Processing State Descriptor (PSD) concept

The PSD is an abstract collection of all information stored at a given time during your work. It consists of the followings:

* Original image: this is the starting point.
* Blobs: our goal is usually to get a separate blob for every grain in the image. Blobs are areas of the image. They may be divided, merged, altered, selected etc.
* Selection: a subset of the blobs which are currently selected. It is used to mark some blobs for the next operation.
* Aux: this is a drawing which can be directly modified by the user. It is the input and/or output of several processing steps.

A key concept of GrainAutLine is that every operation performed on the image is a transformation from a PSD into another one. See the processor descriptions and the examples later for further insights.

## Application control

This is a short description on how to control the basic functions of the application.

### Menus

Currently, the menu of GrainAutLine is very simple: you can open a file (may be an image for new work, or a PSD if you continue previous work), save a PSD, and run the selected processor (entirely or step-by-step).

* File / Open: opens a previously saved state (PRC file), or an image to start a new one.
* File / SaveAs: saves the current state of the work into a PRC file.

### Other user interface elements

The main windows area consists of the following parts:

* Editor: this is the largest part and it is where the work is done.
* Processor selector (drop-down list): you can change the active (currently selected) processor here.
* Processor properties: if the active processor has settings, you can see them here.
* Supplementary view: to help the drawing, if the mouse is moving, the part of the original image just below the mouse is shown here.
* Layers: the editor shows its content in several layers. You can toggle their visibility here.

The Layers also have an opacity which may be changed if the mouse is over the layer name. The 0% opacity if full transparency. There is a slight difference in the operation of 99% and 100% opacity: 100% covers everything below it, while 99% introduces a blending which makes deeper layers visible, especially if they also have high opacity. (100%  opaque layers have still a transparent color which does not cover the layers below them.)

### Pan and zoom in the editor

You can zoom with Ctrl-Scroll and move the image with the middle mouse button.

### Colors used in the editor area

The editor blends the visible layers. This means that if there are multiple things visible in the same location, their color is blended.

* The original image preserves its original colors.
* Yellow: the Aux layer shows the Aux with yellow.
* Green. red: the selection layer shows the selection or its inverse with green, and optionally the border lines with red. See the subsection of selection visualization.
* Blue: the Blobs layer uses blue. If you choose to make the blobs have different color (see global functions "C" below), blobs will have various blue shades.

Blobs can be shown in different colors:

* C: starts color mode, pressed multiple times rotates the blob color assignments
* M: returns to monochrome mode 

### Selection visualization modes

There are 3 modes for visualizing the set of selected blobs on the Selection layer:

* Normal mode shows the selected blobs with green.
* Inverted mode shows the unselected blobs with green. This may be useful if you want to see the current blob in full details on the original image, but you still want to know where its boundaries are.
* Complex mode is like the inverted, but it also shows the border lines with red. This allows you to see the border lines even inside the currently selected blob.

You can change between these modes by pressing F9.

### Selecting blobs

There are two ways to select blobs:

* Ctrl + Left click always toggles the selection of the blob under the mouse.
* If enabled (toggled by F8), if the right mouse button is pressed, the blob under the mouse will be the single selected one. This allows you fast selection, but does not allow multiple blobs to be selected at the same time.
* S: Selects all blobs
* U: Unselects all blobs

### Best practices

Currently, the most appropriate selection and visualization mode is the following:

* Inverted or Complex selection visualization (changed by F9), together with selection by right button press and mouse movement (toggled by F8).

## Global functions

Global functions are always available, independently of the currently selected processor. (We use LClick and RClick as abbreviations of left and right mouse clicks.)

* Shift+Click: Hold down the shift key and a mouse button to draw on the Aux.
   * Shift+LClick: draw on Aux
   * Shift+RClick: erase from Aux
* Ctrl+LClick: toggle selection of the blob under the mouse
* Delete key clears the entire Aux.
* If the right mouse button is pressed and the mouse is moved, the blob under the mouse is highlighted. This is meant to make the boundaries easier to see.
* C: shows the blobs with different colors. Repeated pressing C rotates the color assignments. (Use this if two neightbouring blobs accidentally get a very similar color.)
* M: shows the blobs with the same color.
* S: selects all blobs
* U: unselect all blobs
* F9: changes selection visualization modes (normal, inverted, complex)
* F8: toggles selection by mouse movement with right button pressed

## Processors

In the following, all processors are described. Processors are usually selected and activated, but they may also react on mouse events and they may have their own settings.

### Adaptive Double Threshold Segmentation (ADTS)

Takes the input image and applies the ADTS image processing algorithm to retrieve the grain boundaries. Result is drawn on the Aux.

* Input: Original image
* Output: Aux
* Unaffected: Blobs, Selection
* Parameters: TBD

### AddSubAux

Adds or subtracts the content of the Aux to or from the Blobs. Blobs connected by the Aux are merged into one and blobs cut into peaces be the Aux are split into multiple blobs. If there are no blobs upon running this processor, a single blob covering the whole image is used as a starting point.

This processor is the most elementary tool for modifying the blobs. Using this, you can modify boundaries, add or subtract areas from blobs.

* Input: Aux, Blobs
* Output: Blobs
* Unaffected: -
* Side effects: Selection is cleared as the Blob set may change. Aux is cleared, as it has served its purpose.
* Parameters:
   * Add: if checked, the Aux is added to the Blobs. Otherwise, the Aux is subtracted.
   * Move result to Aux: see below

By selecting the "Move result to Aux" checkbox, the above operation is extended with an additional one: the border lines are all moved to the Aux and all blobs are erased. It can be used to edit all borders in the Aux easily. After that, that Aux can be subtracted. As the blobs are removed, the subtraction creates the blob set segmented by the contents of the Aux.

By selecting the "Move result to Aux" checkbox, the above operation is extended with an additional one: the border lines are all moved to the Aux and all blobs are erased. It can be used to edit all borders in the Aux easily. After that, that Aux can be subtracted. As the blobs are removed, the subtraction creates the blob set segmented by the contents of the Aux.

### BlobMerger

Merges the blobs connected by the Aux in a smarter way than how the AddSubAux does. Blobs connected by a single connected area of the Aux are "glued" together by filling the gaps between them.

For example if several boundaries are detected where there is none, the small blobs can be merged quickly this way: with strokes on the Aux, connect the ones to merge (you may draw several strokes), and run this processor.

* Input: Aux, Blobs
* Output: Blobs
* Unaffected: -
* Side effects: Selection is cleared as the Blob set may change. Aux is cleared, as it has served its purpose.
* Parameters:
   * Radius: maximal distance the "glue" is allowed to connect.

### Fill Holes

Fills the inner holes in the selected blobs. It is used to remove border-like noises from the inner areas of the blobs after segmentations.

* Input: Blobs, Selection
* Output: Blobs
* Unaffected: -
* Side effects: Selection is cleared as the Blob set is partially replaced.

### Query

Diagnostic processor, not meant to be used in the production environment. The functionality may change arbitrarily. Basically, it is used to retrieve various information about specific locations in the image during debugging of new algorithms.

### AuxMorphology

Erodes the Aux in a smart way: the lines in the Aux are not allowed to be cut (preserves borders), and the erosion is only applied in areas where the original image is light. Assuming the grain borders to be dark, this processor can clean up drawing mistakes by removing the parts of border lines which are definitely not grain borders.

* Input: Aux
* Output: Aux
* Unaffected: Blobs, Selection
* Parameters:
   * Image threshold: color threshold above which the erosion is allowed.

### AutoCut

Slices a single selected blob into peaces closing the narrow passages between bigger, clear areas. The result is a new Aux which can be subtracted from the blobs to achieve the result. (This approach allows the user to check the result before really applying it.)

The algorithm identifies big enough clear areas (called sources) and searches for connections between them. If there is a narrow path between two sources, the narrowest passage is marked on the Aux.

Currently, AutoCut expects exactly 1 selected blob.

* Input: Blobs, Selection
* Output: Aux, Selection
* Unaffected: -
* Parameters:
   * Min source radius: minimum radius of a clear area to be considered a source.
   * Min source area: minimal area of a source area to be used.
   * Max passage width: Maximal width of the narrowest passage to be filled.
   * Max source distance: Maximal distance between source areas to search for a path between them.

### Skeleton

Creates the skeleton of the Aux. The skeleton is retrieved by narrowing the lines of the Aux to be a single pixel wide.

* Input: Aux
* Output: Aux

## Examples on processing

### Segmentation and minor corrections

In this example, we load an image, apply a standard segmentation algorithm, and then modify the segmentation at some points to clean up noises.

* File - Open File...

The image appears in the application.

* Select the processor "Adaptive Double Threshold Segmentation"
* Run the processor

At this point, many detected border lines appear, overlaid the image. (They are drawn on the Aux.)

* Some discontinuities may be completed by drawing on the Aux with Shift+LClick, or some lines may be removed with Shift+RClick.
* Use the corrected Aux as blob boundaries:
  * Select the AddSubAux processor and run it.

Now, the blobs are shown in the editor. While you press the right mouse button, the blob below the mouse is highlighted. At this point, you may notice that some grains still belong to the same blob. (You can make them have different colors by pressing "C". Pressing "C" multiple times rotates the colors.)

* Draw the separating boundary with Shift+LClick on the Aux. If you made a mistake, you can always corrent it with the eraser by using Shift+RClick.
- Run the AddSubAux processor again (with unchecked "Add" setting)

Now you can see the separated blobs my moving the mouse over them, as the automatic highlighting shows them as two separate blobs.

There may be some further noises inside the blobs, so you may want to select some of them and remove the inner holes.

* Select some blobs with holes in them with Ctrl+LClick.
* Select and run the processor "Fill Holes".

Now you have a clean set of blobs which represent the grains in the image. This can be used for further statistical and classification tasks, but they are out of scope of the GrainAutLine application.

### Compare results with older PSD and apply corrections

* Load the reference PSD you do not want to modify.
* Select the PsdCompare processor and run it. It will store the reference PSD.
* Load the other PSD you want to edit during the comparison.
* Move all borders into the Aux (so you can edit them): with an empty Aux, select the AddSubAux processor, select the property "Move result to aux" and run a subtraction. (This will subtract nothing, but after that, move all the boundaries to the Aux.)
* Select the PsdCompare processor. (Currently, you need to run it with unchecked "Store referenced PSD" to make the reference appear.)
* Modify layer opacities as follows: turn off Blobs and Selection, make Original image half transparent (makes it darker). Now you can work on Aux to apply corrections while seeing the reference on the layer "refPSD".
