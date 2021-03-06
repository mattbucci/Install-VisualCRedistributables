# Usage

Deploying the Microsoft Visual C++ Redistributables in any complex desktop environment kinda sucks because there are so many versions that might be required. I got tired of updating my MDT deployment share with the redistributables manually, so wrote a module to automate the process.

This module will download the Visual C++ Redistributables listed in an external XML file into a folder structure that represents major release, processor architecture and update release \(e.g. SP1, MFC, ATL etc.\). The module allows you to download, install or import Visual C++ Redistributables into the Microsoft Deployment Toolkit or System Center Configuration Manager.

_NOTE:_ Validation of the Redistributables listed in the XML file is required, as not all may need to be installed in your environment.

The basic structure of the XML file should be as follows \(an XSD schema is included in the repository\):

```xml
    <Redistributables>
        <Platform Architecture="x64" Release="" Install="">
            <Redistributable>
                <Name></Name>
                <ShortName></ShortName>
                <URL></URL>
                <ProductCode></ProductCode>
                <Download></Download>
                <Install></Install>
            </Redistributable>
        </Platform>
        <Platform Architecture="x86" Release="" Install="">
            <Redistributable>
                <Name></Name>
                <ShortName></ShortName>
                <URL></URL>
                <ProductCode></ProductCode>
                <Download></Download>
                <Install></Install>
            </Redistributable>
        </Platform>
    </Redistributables>
```

Each major version of the redistributables is grouped by `<Platform>` that defines the major release, processor architecture and install arguments passed to the installer.

The properties of each redistributable are defined in each `<Redistributable>` node:

* Name - the name of the redistributable as displayed on the download page. Not used in the script, but useful for reading the XML file.
* ShortName - the redistributable will be downloaded to Release\Architecture\ShortName
* URL - this is the URL to the page at microsoft.com/downloads. Not used in the script, but useful for referencing the download as needed
* ProductCode - this is the MSI Product Code for the specified VC++ App that will be used to import the package into Configuration Manager
* Download - this is the URL to the installer so that the script can download each redistributable

## Functions

This module includes the following functions:

### Get-VcList

This function reads the Visual C++ Redistributables listed in an internal manifest or an external XML file into an array that can be passed to other VcRedist functions. Running `Get-VcList` will return the supported list of Visual C++ Redistributables. The function can read an external XML file that defines a custom list of Visual C++ Redistributables.

### Export-VcXml

Run `Export-VcXml` to export the internal Visual C++ Redistributables manifest to an external XML file. Use `-Path` to define the path to the external XML file that the manifest will be saved to. By default `Export-VcXml` will export only the supported Visual C++ Redistributables.

### Get-VcRedist

To download the Visual C++ Redistributables to a local folder, use `Get-VcRedist`. This will read the array of Visual C++ Redistributables returned from `Get-VcList` and download each one to a local folder specified in `-Path`. Visual C++ Redistributables can be filtered for release and processor architecture.

### Install-VcRedist

To install the Visual C++ Redistributables on the local machine, use `Install-VcRedist`. This function again accepts the array of Visual C++ Redistributables passed from `Get-VcList` and installs the Visual C++ Redistributables downloaded to a local path with `Get-VcRedist`. Visual C++ Redistributables can be filtered for release and processor architecture.

![Microsoft Visual C++ Redistributables installed on the local PC](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/VisualCPrograms.PNG)

### Import-VcMdtApp

To install the Visual C++ Redistributables as a part of a reference image or for use with a deployment solution based on the Microsoft Deployment Toolkit, `Import-VcMdtApp` will import each of the Visual C++ Redistributables as a seperate application that includes silent command lines, platform support and the UninstallKey for detecting whether the Visual C++ Redistributable is already installed. Visual C++ Redistributables can be filtered for release and processor architecture.

Each Redistributables will be imported into the deployment share with application properties for a successful deployment.

![Microsoft Visual C++ Redistributables applications imported into an MDT share](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/MdtVisualCApplications.png)

The folder structure in the deployment share, will look thus:

![Visual C++ Redistributables in the deployment share Application folder](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/MdtVisualCApplicationsFolder.PNG)

### Import-VcCMApp

To install the Visual C++ Redistributables with System Center Configuration Manager, `Import-VcCmApp` will import each of the Visual C++ Redistributables as a seperate application that includes the application and a single deployment type. Visual C++ Redistributables can be filtered for release and processor architecture.

![Microsoft Visual C++ Redistributables applications imported into ConfigMgr](https://raw.githubusercontent.com/aaronparker/Install-VisualCRedistributables/master/images/VcRedistConfigMgr.PNG)

## Examples

To retrieve the list of Visual C++ Redistributables from the embedded manifest, run `Get-VsList`.

```powershell
PS C:\> Get-VcList


Name         : Visual C++ 2008 Service Pack 1 Redistributable Package MFC Security Update
ProductCode  : {5FCE6D76-F5DC-37AB-B2B8-22AB8CEDB1D4}
URL          : https://www.microsoft.com/en-us/download/details.aspx?id=26368
Download     : https://download.microsoft.com/download/5/D/8/5D8C65CB-C849-4025-8E95-C3966CAFD8AE/vcredist_x64.exe
Release      : 2008
Architecture : x64
ShortName    : SP1MFC
Install      : /Q

Name         : Visual C++ 2008 Service Pack 1 Redistributable Package MFC Security Update
ProductCode  : {9BE518E6-ECC6-35A9-88E4-87755C07200F}
URL          : https://www.microsoft.com/en-us/download/details.aspx?id=26368
Download     : https://download.microsoft.com/download/5/D/8/5D8C65CB-C849-4025-8E95-C3966CAFD8AE/vcredist_x86.exe
Release      : 2008
Architecture : x86
ShortName    : SP1MFC
Install      : /Q
```

This array can be passed to other functions to perform various tasks. For example, to download the 32-bit 2010, 2012, 2013 and 2017 Redistributables, use the following command:

```powershell
Get-VcList | Get-VcRedist -Path "C:\Temp\VcRedist" -Release 2010, 2012, 2013, 2017 -Architecture x86
```

To install the Visual C++ Redistributables that have been downloaded to C:\Temp\VcRedist, run:

```powershell
Get-VcList | Install-VcRedist -Path "C:\Temp\VcRedist"
```

The module can import the Visual C++ Redistributables into an MDT deployment share. First, download the Visual C++ Redistributables installers locally, then import them into the share with `Import-VcMdtApp`:

```powershell
$VcList = Get-VcList | Get-VcRedist -Path "C:\Temp\VcRedist"
Import-VcMdtApp -VcList $VcList -Path "C:\Temp\VcRedist" -MdtPath "\\server\share\Reference"
```

The module can be used to import the Visual C++ Redistributables as applications with a single deployment type into ConfigMgr. This includes copying the downloaded installers to a network path.

```powershell
$VcList = Get-VcList | Get-VcRedist -Path "C:\Temp\VcRedist"
Import-VcCmApp -VcList $VcList -Path "C:\Temp\VcRedist" -CMPath "\\server\share\VcRedist" -SMSSiteCode LAB
```