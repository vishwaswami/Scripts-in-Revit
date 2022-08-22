1) This script automatically assign values to the "Mark" Parameter of Strcutural Beams(Steel and Concrete), which is then used as name tage for Beams. So this scripts number the beams in a sequence. 

2) To run this script efficiently, beams should be modelled
	a) In a Plane(XY), horizontal beams should be modelled from left to right.
	b) Vertical beams should be modelled from bottom to top.

3) Refer attacjed pdf for more clarification on use of scripts.

4) Below packages should be installed in dynamo to run these scripts:
	a) Archi-lab
	b) archilab_Bumblebee
	c) Clockwork
	d) Dynamo4AMLib
	e) GeniusLoci
	f) Revit

5) Add shared parameter file "BeamAnnotationSharedCoordinates.txt" in Revit Project.
	a) Is Cantilever -> Add this parameter to Beams. Assign 'y' if the beam is cantilever to skip it from numbering(Before running the Script)
	b) Is Transer Column - > Add this parameter to Column. Assign 'y' if column is tarnsfer Columns to assign them different numbering.

6) Steps to follow while running scripts:
	a) Run "Beam1_RUN BEAM ANNOTATION SCRIPT ON ALL BEAMS" on all beam of particular floor/Level.
	b) Run "Beam2_CORRECTION DONE TO HORIZONTAL BEAMS" to apply corrections to horizontal beams(refer README-BEAMS-STEPS TO FOLLOW.pdf for more information)
	c) Run "Beam3_ CORRECTION DONE TO VERTICAL BEAMS" to apply correction to vertical beams."
	d) Run "Beam4_DELETE MARK VALUES FOR BEAMS" to erase assigned values to parameters.

7) Run the scripts using Dynamo Player. Below are the General inputs required from user:
	a) Enter name for Cantilever Beams - > Enter the name for cantilever beams as per Company's standards, those will be skipped from numbering.
	b) Enter Suffix for the steel Beam -> Enter suffix for steel beam as per company standards
	c) Enter Level Number -> Enter the Current level
	d) Enter Suffix for Horizontal Beams -> Enter suffix for horizontal beams(in XY plane) as per Company's Standards(eg HB)
	e) Enter Suffix for Vertical Beams -> Enter suffix for Vertical beams(in XY plane) as per Company's Standards(eg VB)
	f) Number Slider for Horizontal Beams - > Select number to start numbering from
	g) Number Slider for Vertical Beams - > Select number to start numbering from
	h) Select Model Elements -> Select all elements of a particular floor (Script will filter Structural Framing automatically) 
