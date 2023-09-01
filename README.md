# Introduction

The Echo AIO Test System is a modular platform for acoustic and audio testing. Echo provides a cross-platform shared API library for integrating an Echo AIO with your own custom or third-party software.

The API is packaged as shared library - EchoAIOInterface.dll for Windows (32-bit and 64-bit) and EchoAIOInterface.dylib for macOS. The macOS library supports Apple Silicon as well as Intel. API functions follow standard C calling conventions for their respective platforms.

To use the API, load the shared library just like any other. Be sure to call AIO_initialize to set up the DLL and then AIO_shutdown before closing the library.


## Python example
Here's how to use the API from a Python script. You'll need to install the ctypes library to access the shared library. This example demonstrates how to read the input gain setting.

```Python
#
# ctypes provides C compatible data types and allows calling functions in DLLs or shared libraries.
#
import ctypes
import os

#
#    Look in the same directory as this script for the DLL. The default install location is C:\Progam Files\Echo AIO\CLI + API
#
thisScriptFile = __file__
thisScriptDir = os.path.dirname(thisScriptFile)
echoaio = ctypes.WinDLL(thisScriptDir + '\EchoAIOInterface.dll')

#
#    Always call AIO_initialize() before calling any other function to set up the library.
#
echoaio.AIO_initialize()

#
#    Check if the AIO is connected
# 
if (echoaio.AIO_isAIOConnected()):
    #
    #    Read the constant current state
    #    Note the use of ctypes.byref to pass the constantCurrentEnabled parameter by reference
    # 
    AIO_OK = 0
    inputChannel = 1  # MIC2
    constantCurrentEnabled = ctypes.c_int(0)
    status = echoaio.AIO_getConstantCurrentState(inputChannel, ctypes.byref(constantCurrentEnabled));
    if (status == AIO_OK):
        print("Input channel " + str(inputChannel + 1) + " CCP state is " + str(constantCurrentEnabled.value))
    else:
        print("Unable to read CCP state on input channel " + str(inputChannel + 1))
else:
    print("AIO not found")

#
#    Call AIO_shutdown() before unloading the library to release memory and resources
#
echoaio.AIO_shutdown.restype = None
echoaio.AIO_shutdown()
```
<br />

## C++ example

For C++, accessing the API is similar. First, you will need to load the library and then get the function pointers from the library. Here's a C++ example demonstrating how to do this on Windows.
```C
#include <iostream>
#include <Windows.h>
#include "../../EchoAIOInterface.h"

//
// APILibrary encapsulates loading API DLL and functions
//
#define LOAD_FUNCTION(functionName) loadFunction(#functionName, (void**)&functionName)

struct APILibrary
{
    APILibrary()
    {   
        //
        // Look for the DLL in the working directory
        //
        std::string libraryName = "EchoAIOInterface.dll";
        libraryHandle = LoadLibraryA(libraryName.c_str());
        if (nullptr == libraryHandle)
        {
            throw std::runtime_error("Unable to load API library " + libraryName);
        }

        //
        // Load function pointers from DLL
        //
        LOAD_FUNCTION(AIO_initialize);
        LOAD_FUNCTION(AIO_isAIOConnected);
        LOAD_FUNCTION(AIO_getInputGain);
        LOAD_FUNCTION(AIO_shutdown);
    }

    ~APILibrary()
    {
        if (nullptr != libraryHandle)
        {
            FreeLibrary(libraryHandle);
        }
    }

    void (*AIO_initialize)() = nullptr;                     // void AIO_initialize();
    int (*AIO_isAIOConnected)() = nullptr;                  // int AIO_isAIOConnected();
    int (*AIO_getInputGain)(int, int* const) = nullptr;     // int AIO_getInputGain(int inputChannel, int* const gain);
    void (*AIO_shutdown)() = nullptr;                       // void AIO_shutdown();

private:
    HMODULE libraryHandle = nullptr;

    void loadFunction(std::string functionName, void** functionPointer)
    {
        *functionPointer = GetProcAddress(libraryHandle, functionName.c_str());
        if (nullptr == *functionPointer)
        {
            throw std::runtime_error(std::string{ "Unable to load function " } + functionName);
        }
    }
};

//
// Demonstrate how to read the input gain setting using the API
//
void readInputGainFromAIO(APILibrary const& apiLibrary)
{
    //
    // Always call AIO_initialize first
    //
    apiLibrary.AIO_initialize();

    //
    // Check if an AIO is connected
    //
    if (apiLibrary.AIO_isAIOConnected())
    {
        //
        // Read the input gain setting
        //
        int inputChannel = 0; // MIC1
        int gain = 0;
        int status = apiLibrary.AIO_getInputGain(inputChannel, &gain);
        if (ECHO_AIO_OK == status)
        {
            std::cout << "Input channel " << inputChannel + 1 << " gain is " << gain;
        }
        else
        {
            std::cout << "Unable to read input gain; error " << status;
        }
    }
    else
    {
         std::cout << "No AIO connected";
    }
    
    //
    // Always call AIO_shutdown before unloading the DLL
    //
    apiLibrary.AIO_shutdown();
}

//
// main
//
int main()
{
    try
    {
        APILibrary apiLibrary;

        readInputGainFromAIO(apiLibrary);
    }
    catch (std::runtime_error& e)
    {
    	std::cout << "Error: " << e.what() << std::endl;
    }
}
```
<br />
Copyright (c) 2023 [Echo Test + Measurement](https://echotm.com/)

