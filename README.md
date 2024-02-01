# EDIFACT Parser

This project demonstrates the possibility to parse an EDIFACT message to an XML structure using TIBCO ActiveMatrix BusinessWorks 6.<br>
More specifically, two activities have been used in TIBCO BusinessStudio in order to realize the parsing:

* **EDIFACTPrescanner**: the activity scans the header portion of the EDIFACT message and returns a predefined XML structure with the header values which characterize the EDIFACT message
* **Translator**: the activity translates the entire EDIFACT message into an XML structure by using a `.map` file, which provides the EDIFACT-to-XML conversion rules

The final XML structure is then used to retrieve some fields of interest:
* Order number
* Customer code
* Article (or articles) code
* Article (or articles) description
* Article (or articles) quantity

## Required Components

The basic requirement for the project is to have the last version of *TIBCO ActiveMatrix BusinessWorks 6* installed on your local or virtual machine. For this project, *version 6.10.0* has been used for BusinessWorks.<br>
Additional components are required for the realization of the project:

* *TIBCO Foresight EDISIM, version 6.20.0*
* *TIBCO Foresight Instream Standard Edition, version 8.6.0*
* *TIBCO Foresight Translator Standard Edition, version 3.6.0*
* *TIBCO BusinessWorks Plug-in for EDI Standard Edition, version 6.3.1 HF-001*

## Installation

All the listed components are available on the TIBCO eDelivery site.

To install the **EDISIM** component:
* search on eDelivery for "TIBCO Foresight EDISIM"
* download the `.exe` file with the specified version
* execute the installer. During the installation, you can specify the same `TIBCO_HOME` folder used for BusinessWorks 6; or, an additional `TIBCO_HOME` folder can be used to host the software

To install the **Foresight Instream** component:
* search on eDelivery for "TIBCO ActiveMatrix BusinessWorks Plug-in for EDI Standard Edition"
* find the `.zip` file of *TIBCO Foresight Instream* in the download list
* download the `.zip` folder and extract all in the file system
* run the `TIBCOUniversalInstaller.exe` executable. Again, either the already existing `TIBCO_HOME` folder or create a new one

To install the **Foresight Translator** component:
* search on eDelivery for "TIBCO ActiveMatrix BusinessWorks Plug-in for EDI Standard Edition"
* find the `.zip` file of *TIBCO Foresight Translator* in the download list
* download the `.zip` folder and extract all in the file system
* run the `TIBCOUniversalInstaller.exe` executable. Again, either the already existing `TIBCO_HOME` folder or create a new one

The approach adopted for this project is to create for each component a separate `TIBCO_HOME` folder in order to separate each software.

To install the **BusinessWorks EDI Plug-in** component:
* search on eDelivery for "TIBCO ActiveMatrix BusinessWorks Plug-in for EDI Standard Edition"
* find the `.zip` file of the *TIBCO ActiveMatrix BusinessWorks EDI Plug-in* in the download list
* download the `.zip` folder and extract all in the file system
* run the `TIBCOUniversalInstaller.exe` executable: in this case, it is required to use the same `TIBCO_HOME` folder where BusinessWorks 6 is hosted
* search on the TIBCO Support portal the hotfix *HF-001* of the EDI Plug-in
* download the `.zip` folder of the hotfix, extract all in the file system
* copy the `TIBCOUniversalInstaller.exe` into the folder and run it. Again, use the same `TIBCO_HOME` folder where BusinessWorks 6 is hosted

The *HF-001* is necessary for the realization of the project. Without it, the *EDIFACTPrescanner* activity will not work with any EDIFACT message.

## Getting started

Once everything is installed, open a new workspace in *TIBCO ActiveMatrix BusinessWorks 6* and clone the repository. Wait until the BusinessStudio completes the workspace build.

As outlined in the [documentation](https://docs.tibco.com/pub/bwpluginedi_healthcare/6.3.0/doc/pdf/TIB_bwpluginedi_6.3_usersguide.pdf?id=0) (page 74), two environment variables must be added to let Foresight Instream and Translator work properly. Follow the steps described in the documentation to add in the PATH environment variable the /bin folders.

Focus now on the Translator activity, that uses a map file for the EDIFACT-to-XML translation. To make the translator work correctly, the map file must be correctly set.<br>
The map file used in the Translator is recovered with EDISIM, by performin the following steps:
* run the EDISIM program by executing the `EDISIM.exe` executable in the `{EDISIM_HOME}/bin` folder
* click on the "Standard Editors" link, the first appearing in the wizard, so the Standard Editor window opens
* File &rarr; Open: select the guideline for the specific EDIFACT version (**D96A** in our case)
* choose the type of EDIFACT message you want to handle and click on it (**HANMOV** in our case)
* File &rarr; Export &rarr; Export Current Guideline &rarr; To Schema: save the `.xsd` guideline where desired

The translation guideline is composed of an `.xsd` file (`D96A_HANMOV.xsd`), that is the target guideline of the translation process, and two `.map` files (`D94A_HANMOV_EX.map` and `D94A_HANMOV_XE.map`), one for the EDIFACT-to-XML conversion and the other for the opposite one. For this project, we are going to use the first `.map` file, which is responsible for the EDIFACT-to-XML conversion. However, it is *TIBCO Foresight Translator* that is responsible to handle such conversion; therefore, let these three files be available for Foresight Translator by moving them in the `{FORESIGHT_TRANSLATOR_HOME}/Database` folder.

Perform also the following steps, to recover with EDISIM the source guideline for the translation process:
* choose the EDIFACT message type in the EDISIM Standard Editor as before
* File &rarr; Export &rarr; Export Multiple Guidelines &rarr; To SEF
* select the desired standard (in our case, **EAN97V3**), click the "Add" button and export the guideline where desired

Again, it is better to move the file inside the `{FORESIGHT_TRANSLATOR_HOME}/Database` folder.<br>
Change the extension from `.SEF` to `.std` so that it can be used as a source guideline: at the end, the file should be called `EAN97V3.std`.

Open now the file `D94A_HANMOV_EX.map`, and identify these XML tags:
```xml
<SourceGuideline value="" />
```
```xml
<TargetGuideline value="" />
```
Fill the `value` attribute with the absolute path of the `EAN97V3.std` file and the `D96A_HANMOV.xsd` file respectively. The result should be something like
```xml
<SourceGuideline value="{FORESIGHT_TRANSLATOR_HOME}\Database\EAN97V3.std" />
```
```xml
<TargetGuideline value="{FORESIGHT_TRANSLATOR_HOME}\Database\D96A_HANMOV.xsd" />
```

Everything is now set up for a run. Just check if the `.xsd` file used in the translation map is also imported in the BusinessWorks project, so that you can parse the output of the *Translator* activity to an XML structure.
