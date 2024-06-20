# ER-NEjunction3Dmorphometry

This repository contains the codes associated with the manuscript:

> Helena Bragulat-Teixidor, Keisuke Ishihara, Gréta Martina Szücs, Shotaro Otsuka. (2024) "**The endoplasmic reticulum connects to the nucleus by constricted junctions that mature after mitosis**", EMBO Reports [DOI link](https://doi.org/10.1038/s44319-024-00175-w)

The code quantifies the 3D ultrastructure of the junctions that connect the endoplasmic reticulum (ER) to the nuclear envelope (NE) (ER-NE junctions) and junctions within the ER (ER-ER junctions).


# Usage instructions

## 1. Obtaining cross-sections along the junction axis using NeuroMorph

### Step 1: Use IMOD to create 3D models of junctions

1. Using [IMOD](https://bio3d.colorado.edu/imod/), create 3D models of junctions through manual segmentation.
2. Save the models as *.mod* files.
3. In Windows Command Prompt, set the directory to the folder containing the *.mod* files `cd fullpath\to\mymodels`.
4. Convert the *.mod* files into *.obj* files by running the command `imod2obj mymodel.mod mymodel.obj`.

### Step 2: Use Blender+NeuroMorph to obtain cross-sections along the junction axis

0. Install [Blender 2.8.3 LTS](https://www.blender.org/download/lts/2-83/) and [NeuroMorph](https://github.com/NeuroMorph-EPFL/NeuroMorph). (For MacOS Apple Silicon, choose [2.9.3 LTS](https://www.blender.org/download/lts/2-93/) and the [specified script for Centerline Processing](https://github.com/NeuroMorph-EPFL/NeuroMorph/tree/master/NeuroMorph_Blender_2.9_experimental).)
1. In Windows Command Prompt, set the directory to the folder containing the downloaded "ER-NEjunction3Dmorphometry" *.py* files `cd fullpath\to\ER-NEjunction3Dmorphometry\pyfiles`.
2. Run `Blender -P ImportPrepareMesh.py -- fullpath\to\mymodel.obj`. A new Blender session opens automatically containing two objects: the 3D mesh model centred at the origin of coordinates and a two-point line next to it.
3. Select the line in the Object mode, switch to Edit mode, and select the Move tool in the Toolbar (left).
4. Move the end points of the line so that it spans the junction neck like a centerline. The end points have to be positioned inside the mesh. It is convenient to make the mesh semi-transparent by activating the Toggle X-ray tool on the Header bar (top).
5. In the Scripting workspace, open and run <ins>CreateCenterline.py</ins>. This script subdivides the original 2-point line into 11 smaller segments.
6. In the Layout workspace, select the line in the Object mode, switch to Edit mode, and select the Move tool in the Toolbar (left).
7. Move the points of the centreline so that they follow the junction axis more accurately.
8. In the Scripting workspace, open and run <ins>GetSections.py</ins>.
9. In the Layout workspace, select the mesh in the Object mode, and click on an empty space in the Main Window. Press the F3 button on the keyboard to open the hotkey pop-up menu, search for "Get Sections - ER-NEjunction3Dmorphometry" and press Enter to execute. This runs the </ins>GetSections.py</ins> script to generate cross-sections along the centreline at regular intervals.
10. Check the results of the NeuroMorph operations.
	- Did the shape of the 3D mesh model change, especially around the junction neck?
	- Are the cross-sections in the right orientation/position?
	- Delete cross-sections that look obviously incorrect.
	- If any of the previous points is unsatisfactory, quit Blender and start again by trying any of the following:
		- Changing the search radius value in <ins>GetSections.py</ins> at line 54 (bpy.context.scene.search_radius = 60).
		- Changing the position of the points of the centerline (Step 7).
		- Modifying the manual segmentation of the junction in IMOD to obtain a refined 3D mesh model.
	- If the result is satisfactory, delete all objects except the cross-sections spanning along the junction axis from the junction base. (i.e. delete the mesh, the centerline, the centerline intersection markers, and cross-sections that might be below the base of the junction). Save the entire workspace as *mymodel_crosssectionsfrombase.blend*, and close the program. Note that to run the code in the subsequent sections successfully, the file name has to end with "_crosssectionsfrombase".

### Step 3: Export the cross-sections as GLB files

1. In Windows Command Prompt, set the directory to the folder containing the downloaded "ER-NEjunction3Dmorphometry" *.py* files `cd fullpath\to\ER-NEjunction3Dmorphometry\pyfiles`.
2. Run `Blender -P ExportCrossSections.py -b -- fullpath\to\mymodel_onlycrosssections.blend`.

	- Windowless (-b)
	- All cross-sections will be exported as GLB files
	- The original Blender filename will be respected
	- To convert all files in the folder to GLB using the PowerShell (Windows), run:
		```
		$files = Get-ChildItem -Path fullpath\to\folder\containing\onlycrosssections
		foreach ($file in $files) {
		Blender -P fullpath\to\ExportCrossSections.py -b -- $file.FullName
		}
		```
	- To convert all files in the folder to GLB using the bash loop (MacOS), run:
		```
		for f in ./cross-sections/1_blend/ER-NEtelophase/*; do Blender -P ./organellemorphometry/organellemorphometry/ExportCrossSections.py -b -- $(realpath "$f"); done
		```

## 2. Morphological analysis of the cross-sections

1. (First time only) Install Anaconda or the lightweight [Miniconda](https://docs.conda.io/en/latest/miniconda.html#latest-miniconda-installer-links)/[Mambaforge](https://github.com/conda-forge/miniforge#mambaforge) on your computer, and create a new environment: `conda create -n junction3Dmorphometry-env -c conda-forge python=3.9 seaborn=0.11 notebook=6.4 trimesh=3.10 shapely=1.8`.
2. Activate the environment: `conda activate junction3Dmorphometry-env`.
3. Launch Jupyter notebook: `jupyter-notebook`.
4. Launch <ins>CrossSectionAnalysis_fromBase.ipynb</ins> and analyze the GLB files.


## 3. Creating top view of junctions and quantifications

We create plots for the membrane profiles of the top view of junctions using the cross-section with the smallest surface area.

### Step 1: Export the coordinates of the translation of 3D mesh models from the original position in IMOD to the position at the origin of coordinates in Blender
1. Open the Command Prompt, and set it to the directory containing the 3D mesh models (*.obj* files) obtained from IMOD in the section 1.1, by running `cd fullpath\to\mymodels`.
2. Export the original coordinates of the 3D mesh models before translating them to the origin of coordinates in Blender in the section 1.2.
	- To export the coordinates of all the models using the PowerShell (Windows):
		```
		$files = Get-ChildItem -Path C:\path\to\ModelFiles\* 
		foreach ($file in $files) {
		& Blender -P C:\Path\to \ImportPrepareMesh.py -- $file.FullName C:\Path \to\save\translation\output}
		```
	- To export the coordinates of all the models using the bash loop (MacOS):
		```
		for FILE in *; do Blender -P /fullpath/to/ImportPrepareMesh.py -- $FILE /output/to/translation; done
		```
	- Make sure to use the same Blender version as the NeuroMorph operations in the section 1.2. The translations in ImportPrepareMesh.py are version dependent.
	- The coordinates are exported as *.csv* files.

### Step 2: Obtain the cross-section with the smallest surface area
1. Open the Command Prompt, and set it to the directory containing the downloaded OrganelleMorphometry files by running `cd C:directory\OrganelleMorphometry`.
2. Obtain the cross-section with the smallest surface area for each junction, and convert from *.obj* to *.mod* files that can be overlaid on the original EM data.
	- To convert all files in the folder (e.g. 10) to *.mod* files using the PowerShell (Windows), run:
		```
		for ($f = 0; $f -le 10; $f++) { & Blender -P path\to\SelectCorrectPlane.py -b -- $f path\to\MinimumCrossectionFiles }
		```
	- To convert all files in the folder (e.g. 10) to *.mod* files using the bash loop (MacOS), run:
		```
		for f in {0..10}; do Blender -P SelectCorrectPlane.py -b -- $f ER-NEtelophase ; done
		```
	- The input and output paths are specified in the *.py* file. This step must be done on Blender 2.9.3LT.
	- The cross-section with the minimum surface area is translated using the coordinates obtained in section 2.1 to the position that can be overlaid on the original EM tomogram. The possibility to overlay the cross-section on the EM tomogram allows refining its shape into a smoothened top view membrane profile.
	- The files are exported as *.mod* files.

### Step 3: Use IMOD to refine the top view profile
1. Open the EM tomogram in IMOD and the corresponding *.mod* file of the translated cross-section with the smallest surface area.
2. Open the Slicer View and press the C button on the header toolbar (or alternatively the hot key Shift+W) in order to set the angles and position that show the plane of the cross-section.
3. Draw manually the top view membrane profile as a new object, verifying on the Zap window that the points are added acrosst the middle of the lipid bilayer at the junction neck.
4. Delete the object of the unrefined cross-section, and save the object of the refined top view profile as a *.mod* file.

### Step 4: Obtain plots of the top view profiles
1. Open the Command Prompt, and set it to the directory containing the *.mod* files of the refined top view profiles by running `cd C:directory/ModelFiles`.
2. Convert the *.mod* files into a space-delimited txt file by running the command `model2point –ob model.mod model.txt`.
	- To convert all files in the folder from *.mod* to *. txt* files using the bash loop (MacOS), run:
		```
		$files = Get-ChildItem -Filter *.mod
		foreach ($file in $files) {
		$outputFile = "$($file.FullName).txt"
		model2point $file.FullName $outputFile
		}
		```
	- To convert all files in the folder from *.mod* to *. txt* files using the bash loop (MacOS), run:
		```
		for f in ./*.mod; do model2point "$f" "$f.txt"; done
		```
3. (First time only) Install Anaconda or the lightweight [Miniconda](https://docs.conda.io/en/latest/miniconda.html#latest-miniconda-installer-links)/[Mambaforge](https://github.com/conda-forge/miniforge#mambaforge) on your computer, and create a new environment: `conda create -n organelle-env -c conda-forge python=3.9 seaborn=0.11 notebook=6.4 trimesh=3.10 shapely=1.8`.
4. Activate the environment: `conda activate organelle-env`.
5. Launch Jupyter notebook: `jupyter-notebook`.
6. Launch <ins>TopView.ipynb</ins> to create the plots for the top view membrane profiles.

## 4. Creating side view of junctions

We create plots for the membrane profiles of the side view of junctions using manually traced membrane profiles.

### Step 1: Use IMOD to create the membrane profiles of the side view of junctions

1. Rotate the EM tomogram to the orientation and position of the sagittal plane of the junction in the Slicer mode of IMOD. It may be convenient to open the *.mod* file of the 3D mesh model to facilitate the identification of the position and orientation of the sagittal plane.
2. Manually trace the membranes across the middle of the lipid bilayer. Use different objects (in the open object type) to trace each type of membrane:
	- Object 1: Left side of the junction, drawn from the outer nuclear membrane to the endoplasmic reticulum.
	- Object 2: Right side of the junction, drawn from the outer nuclear membrane to the endoplasmic reticulum.
	- Object 3: Inner nuclear membrane. (Not used for the current analysis.)
	- Object 4: Two-point line in which the first point defines the centre of the junction, and the second point indicates the direction of the line along the junction axis.
3. Save the models as *.mod* files.

### Step 2: Obtain plots of the side view profiles
1.	Open the Command Prompt, and set it to the directory containing the *.mod* files of the side view profiles by running `cd C:directory\ModelFiles`.
2.	Convert the *.mod* files into a space-delimited txt file by running `model2point –ob model.mod model.txt`. Note the `-ob` option to include the Object IDs in the output.
	- To convert all files in the folder from *.mod* to *. txt* files using the PowerShell (Windows), run:
		```
		$files = Get-ChildItem -Filter *.mod
		foreach ($file in $files) {
		$outputFile = "$($file.FullName).txt"
		model2point -ob $file.FullName $outputFile
		}
		```
	- To convert all files in the folder from *.mod* to *.txt* files using the bash loop (MacOS), run:
		```
		for f in ./*.mod; do model2point -ob "$f" "$f.txt"; done
		```
3. (First time only) Install Anaconda or the lightweight [Miniconda](https://docs.conda.io/en/latest/miniconda.html#latest-miniconda-installer-links)/[Mambaforge](https://github.com/conda-forge/miniforge#mambaforge) on your computer, and create a new environment: `conda create -n organelle-env -c conda-forge python=3.9 seaborn=0.11 notebook=6.4 trimesh=3.10 shapely=1.8`.
4. Activate the environment: `conda activate organelle-env`.
5. Launch Jupyter notebook: `jupyter-notebook`.
6. Launch <ins>RotateSideView.ipynb</ins> to create the plots of the side view membrane profiles. Concretely, this notebook imports the *.txt* file, finds the common plane among the profiles, projects all profiles to this plane, and centers/rotates the profiles according to Object 4.
