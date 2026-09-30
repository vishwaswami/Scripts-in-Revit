# Automated Beam Numbering for Revit (Dynamo)

A set of four Dynamo scripts that automatically number the structural beams on a Revit floor plan and write the result to each beam's **Mark** parameter, ready for tagging on drawings. Concrete beams, steel beams, horizontal and vertical runs, and cantilevers each follow their own naming rule, so a full level goes from untagged to fully numbered in one run from Dynamo Player.

**Example output on level 2:** `2HB1, 2HB2 …` for horizontal beams, `2VB1, 2VB2 …` for vertical beams, `ST1, ST2 …` for steel sections and `CANT.` for cantilevers.

\---

## Why this project

Beam marks on structural drawings are usually typed in by hand, one beam at a time, on every floor. It is slow, and numbers are easily skipped, duplicated or left out of sequence when the model changes. These scripts replace that with a rule-based process. The numbering follows a consistent drawing convention, uses the company's own prefixes and suffixes through simple inputs, and can be re-run whenever the layout changes.

## The scripts

|#|Script|What it does|
|-|-|-|
|1|`Beam1\_RUN\_BEAM\_ANNOTATION\_SCRIPT\_ON\_ALL\_BEAMS.dyn`|Main script. Numbers every horizontal, vertical, steel and cantilever beam on the selected level|
|2|`Beam2\_CORRECTION\_DONE\_TO\_HORIZONTAL\_BEAMS.dyn`|Re-sequences a group of staggered horizontal beams, using a reference beam and an offset band|
|3|`Beam3\_\_CORRECTION\_DONE\_TO\_VERTICAL\_BEAMS.dyn`|Same correction for vertical beams|
|4|`Beam4\_DELETE\_MARK\_VALUES\_FOR\_BEAMS.dyn`|Clears the Mark value on all selected beams so a level can be renumbered|

Supporting files:

|File|Purpose|
|-|-|
|`BeamAnnotationSharedCoordinates.txt`|Revit shared parameter file with `Is Cantilever` (beams) and `Is Transfer Column` (columns)|
|`README-BEAMS-STEPS\_TO\_FOLLOW.pdf`|Illustrated step-by-step guide with screenshots|

## How it works

All three numbering scripts share the same core logic, built from standard Dynamo nodes plus a few package nodes.

**1. Filter the selection.** The user selects everything on a level. `FilterElements.IsFraming` keeps only Structural Framing, so columns, walls and slabs in the selection are ignored.

**2. Pull out cantilevers.** Each beam's `Is Cantilever` value is read. Beams set to `y` are removed from the numbering and their Mark is set to the user's cantilever label (for example `CANT.`).

**3. Split by material.** The beam's structural material name is checked. Concrete beams go into the directional numbering, and steel beams are handled separately.

**4. Number steel beams by section.** Steel beams are sorted by section size, read from the digits in their type name by a small Python node. Beams of the same section are grouped, and each group gets a steel mark in size order (`ST1`, `ST2` …).

**5. Split by direction.** Each concrete beam's location line is tested with `Vector.IsParallel` against the X and Y axes, giving a horizontal set and a vertical set.

**6. Sort in drawing order.** Start-point coordinates are rounded to two decimals and used as sort keys.

* Horizontal beams: sorted by Y, then X, so numbering runs top to bottom, left to right.
* Vertical beams: sorted by X, then Y, so numbering runs left to right, bottom to top.

**7. Write the marks.** A code block builds each mark as `Level + Suffix + Number`, starting from the user's chosen start number, and `Element.SetParameterByName` writes it to **Mark**.

### Correction scripts (Beam2 and Beam3)

Sorting uses each beam's start coordinate. When beams in the same row are slightly offset from each other in plan, their Y values differ and the numbering jumps between rows. The correction scripts fix this:

1. The user selects the beams to correct and one reference beam.
2. The user enters a negative and a positive offset in metres (sliders from 0 to 50 m).
3. Every selected beam whose start coordinate falls within that band around the reference beam is treated as one row, sorted along the row, and renumbered from the chosen start number.

The correction can be run on as many groups as needed.

### Reset script (Beam4)

Filters the selection down to Structural Framing and sets **Mark** to an empty value on every beam.

## User inputs (Dynamo Player)

The scripts are built to run from **Dynamo Player**, so users never need to open the graph.

|Input|Example|Used in|Meaning|
|-|-|-|-|
|Name for cantilever beams|`CANT.`|1, 2, 3|Label for beams flagged `Is Cantilever = y`|
|Suffix for steel beams|`ST`|1, 2, 3|Prefix for steel section marks|
|Level number|`2`|1, 2, 3|Added to the start of every mark on this floor|
|Suffix for horizontal beams|`HB`|1, 2|Direction code for X-direction beams|
|Suffix for vertical beams|`VB`|1, 3|Direction code for Y-direction beams|
|Start number, horizontal|`1`|1, 2|First number of the horizontal sequence (0 to 500)|
|Start number, vertical|`1`|1, 3|First number of the vertical sequence (0 to 500)|
|Select model elements|—|all|Select everything on the level|
|Reference element|—|2, 3|Beam the offset band is measured from|
|Offset (negative / positive)|`1`|2, 3|Band width in metres either side of the reference beam|

## Requirements

* **Autodesk Revit 2021 with Dynamo 2.6.** Beam1 to Beam3 were saved in Dynamo 2.6.1 and Beam4 in Dynamo 2.3.
* **Dynamo packages** (Packages → Search for a Package):

|Package|Version|Needed by|
|-|-|-|
|archi-lab.net|2020.23.12|1, 2, 3|
|Clockwork for Dynamo|0.90.8|1, 2, 3|
|Autodesk Analytical Modeling 2020 Dynamo (Dynamo4AM)|1.0.54|all|
|Genius Loci|2021.6.3|1, 2|
|Synthesize toolkit|11.7.3|2, 3|

## How to use

1. Install the packages listed above.
2. In Revit, go to **Manage → Shared Parameters** and load `BeamAnnotationSharedCoordinates.txt`.
3. Go to **Manage → Project Parameters** and add `Is Cantilever` to the **Structural Framing** category. Add `Is Transfer Column` to **Structural Columns** if you use it.
4. Model beams in plan following these rules, which the sorting depends on:

   * horizontal beams are drawn **left to right**
   * vertical beams are drawn **bottom to top**
5. Set `Is Cantilever = y` on every cantilever beam.
6. Open **Manage → Dynamo Player**, browse to the folder with the scripts, fill in the inputs and run **Beam1** on one level at a time.
7. Check the numbering. If a row is out of sequence, run **Beam2** (horizontal) or **Beam3** (vertical) on those beams.
8. Place beam tags that read the **Mark** parameter.
9. To start again, run **Beam4** to clear the marks.

## Limitations

* **Plan beams only.** Beams must lie in the XY plane and run parallel to the X or Y axis. Beams at an angle in plan are not numbered.
* **Modelling direction matters.** Sorting uses the start point, so a beam drawn right to left or top to bottom can be numbered out of order.
* **Material check is name-based.** A beam is treated as concrete when its material name contains `C`. Steel materials whose names contain a capital `C` may be misclassified, so material naming should follow a consistent standard.
* **Steel sizes come from the type name.** The digits in the type name are used for sorting. Type names must contain the section size.
* **Python engine.** The steel-size Python node is written for IronPython 2.7, the default in Dynamo 2.x. In Dynamo 2.13 and later (Revit 2022 onwards), set the node to IronPython 2 or update the code for CPython 3.
* **`Is Transfer Column`** is included in the shared parameter file but is not yet used by these scripts.

## Planned improvements

* Read both end points and use the lower-left one, so modelling direction no longer matters.
* Automate the correction step by grouping beams whose coordinates fall within a set tolerance.
* Place beam tags automatically after numbering.
* Add column numbering, with a separate sequence for transfer columns.
* Update the Python node for CPython 3 and test on newer Revit versions.

## Repository structure

```
├── Beam1\_RUN\_BEAM\_ANNOTATION\_SCRIPT\_ON\_ALL\_BEAMS.dyn
├── Beam2\_CORRECTION\_DONE\_TO\_HORIZONTAL\_BEAMS.dyn
├── Beam3\_\_CORRECTION\_DONE\_TO\_VERTICAL\_BEAMS.dyn
├── Beam4\_DELETE\_MARK\_VALUES\_FOR\_BEAMS.dyn
├── BeamAnnotationSharedCoordinates.txt
├── README-BEAMS-STEPS\_TO\_FOLLOW.pdf
└── README.md
```

## Skills demonstrated

Dynamo visual programming · Revit automation · List and data management · Geometric sorting and classification · Shared parameters · Python in Dynamo · Structural drawing production · Building tools for non-programmer users (Dynamo Player)

