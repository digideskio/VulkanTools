#![Vulkan Validaotr](images/vkvalidator.png)
This document is an overview of how to use the LunarG Vulkan Validator.
The Vulkan Valiidator is a tool that can be used to:
 1. Determine the state of Vulkan components on your system
 2. Validate that your Vulkan Loader and drivers are installed properly
 3. Capture your system state to use as an attachment when submitting bugs.

## Building
The Vulkan Validator is part of the LunarG [VulkanTools](https://github.com/LunarG/VulkanTools) GitHub respository.
Since it is one of many tools in this repository, it is built with all the other
tools by following the instructions in the [BuilidVT.md](https://github.com/LunarG/VulkanTools/blob/master/BUILDVT.md) file at the top of the
source tree.

## Running
Just call the built Vulkan Validator executable.

```
vkvalidator
```

This will output two things:
 - A command-line ouptut indicating the overall status
 - An HTML file (called vkvalidator.html) containing the details which will be output to one of two locations:
  1. If the current directory is writeable, the HTML will be placed in that location.
  2. Otherwise, it will be saved to your home folder.

Your home folder is the following location (based on your OS):
 - Windows: Wherever your environment variables %HOMEDRIVE%\%HOMEPATH% point to.
 - Linux: It will be placed in your home folder ("~/.").

## Common Command-Line Outputs

#### "SUCCESS: Validation completed properly"

##### Problem:
The LunarG Vulkan Validator could detect no problems with your setup.

##### Possible Reason:
Your system is likely setup properly.  If you have trouble running Vulkan from another location, it could be that your environment variables aren't setup properly.

##### Next Step:
Re-run Vulkan Validator from the location your Vulkan application/game is supposed to run.


#### "ERROR: Failed to find Vulkan Driver JSON in registry"

##### Problem:
This is a Windows-specific error that indicates that no Vulkan Driver JSON files were referenced in the appropriate place in the Windows Registry.

##### Possible Reason:
This can indicate that a Vulkan driver failed to install properly.

##### Next Step:
You should follow up with your Graphics Driver.  See more details below.


#### "ERROR: Failed to find Vulkan Driver JSON"

##### Problem:
The Vulkan loader on your system failed to find any Vulkan Driver JSON files in the appropriate places.

##### Possible Reason:
This can indicate that a Vulkan driver failed to install properly.

##### Next Step:
You should follow up with your Graphics Driver.  See more details below.


#### "ERROR: Failed to find Vulkan Driver Lib"

##### Problem:
The Vulkan loader on your system found at least one Vulkan Driver JSON file, but the Driver library referenced in that file appears invalid.

##### Possible Reason:
This can indicate that a Vulkan driver failed to install properly.

##### Next Step:
You should follow up with your Graphics Driver.  See more details below.


#### [WINDOWS] Dialog box pop's up indicating "vulkan-1.dll is missing from your computer."

##### Problem:
The Vulkan loader "vulkan-1.dll" couldn't be found on your system.  This file is typically installed with some Vulkan driver installs,
some Vulkan-capable games, or the LunarG Vulkan SDK.

##### Possible Reason:
The last Vulkan Runtime install that executed on your system failed to behave properly.  Or, you have never installed a Vulkan loader
on your system.

##### Next Step:
To rememdy this, visit [LunarXchange](https://vulkan.lunarg.com/), and install the latest Vulkan SDK.  If that does not work, file an
Issue to have one of their engineers assist you.


## Vulkan Graphics Driver Problems
If the problem is a likely with your Graphics Driver, it could be for several reasons:
 1. The hardware you have doesn't support Vulkan.
 2. Your hardware supports Vulkan, but you haven't yet installed a driver with Vulkan support.
 3. Everything supports Vulkan, but the driver failed to install properly or does not support Vulkan on your current Operating System.
    - Sometimes, the company may provide Vulkan support for some devices on one Operatings System (say Windows), while still waiting to complete Vulkan on other systems.
    - So, please verify with them that all criteria are met.
