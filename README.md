# media-foundation-samples

Sample implementations of different Microsoft Media Foundation components

## References

* Anton Polinger, Developing Microsoft Media Foundation Applications, ISBN: 978-0-7356-5659-8
* MSDN, Media Foundation Architecture, [http://msdn.microsoft.com/en-us/library/windows/desktop/ms696219(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/windows/desktop/ms696219(v=vs.85\).aspx)
* MSDN, Windows 8.1 Media extension sample, [http://code.msdn.microsoft.com/windowsapps/media-extensions-sample-7b466096](http://code.msdn.microsoft.com/windowsapps/media-extensions-sample-7b466096)
* tenouk.com, DLLs and linking, [http://www.tenouk.com/ModuleBB.html](http://www.tenouk.com/ModuleBB.html) 
* MSDN, Windows Runtime C++ Template Library (WRL), [http://msdn.microsoft.com/en-us/library/vstudio/hh438466.aspx](http://msdn.microsoft.com/en-us/library/vstudio/hh438466.aspx)

## Core MF concepts

An architectural overview of the Media Foundation Architecture:

![Media Foundation Architecture](http://i.msdn.microsoft.com/dynimg/IC500890.png "Media Foundation Architecture")

Two different programming models:

* __Media pipeline__ - Contains Media Source, MFT and Media Sink objects. Any object can be implemented by the application.
  1. [http://msdn.microsoft.com/en-us/library/windows/desktop/ms703912(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/windows/desktop/ms703912(v=vs.85\).aspx)
* __Source reader & Sink writer__
  1. [http://msdn.microsoft.com/en-us/library/windows/desktop/dd940436(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/windows/desktop/dd940436(v=vs.85\).aspx)
  2. [http://msdn.microsoft.com/en-us/library/windows/desktop/ff819461(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/windows/desktop/ff819461(v=vs.85\).aspx)

## Examples

See github projects:

```
https://github.com/joekickass/media-foundation-samples
```

## Plumbing

As usual when it comes to creating components that should fit into a complex framework (e.g. __MF__) there are some plumbing involved. With plumbing I mean all the bulk code that has to be written to hande the packaging of the component as well as to fulfill interfaces that allows the component to be loaded by the framework. In the case of _MF_ , below is a short list:

* DLLs, linking and exporting
* MIDL and MIDLRT
* WRL

### DLLs, linking and exporting

An MF extension is usually implemented as a DLL. The following few steps are a common procedure:

1. Create a Win (Phone) 8.x DLL project

2. Add a definition file (*.def) that defines all exported functions of the shared library (the export list)
  
   __*.def:__
   ```
   EXPORTS
       DllGetActivationFactory             PRIVATE
       DllCanUnloadNow                     PRIVATE
       DllGetClassObject                   PRIVATE
   ```

3. __dllmain.cpp__ defines the entry point of the DLL; `DllMain()`. Let it also implement the exported functions:

   __dllmain.cpp:__
   ```cpp
   #include "pch.h" // pch.h should include <wrl/module.h>

   using namespace Microsoft::WRL;

   BOOL APIENTRY DllMain(HMODULE hInstance, DWORD ul_reason_for_call, LPVOID)
   {
       switch (ul_reason_for_call)
       {
       case DLL_PROCESS_ATTACH:
           DisableThreadLibraryCalls(hInstance);
           break;
       case DLL_THREAD_ATTACH:
       case DLL_THREAD_DETACH:
       case DLL_PROCESS_DETACH:
           break;
       }
       return TRUE;
   }

   HRESULT WINAPI DllGetActivationFactory(HSTRING activatibleClassId, IActivationFactory** factory)
   {
       return Module<InProc>::GetModule().GetActivationFactory(activatibleClassId, factory);
   }

   HRESULT WINAPI DllCanUnloadNow()
   {
       return Module<InProc>::GetModule().Terminate() ? S_OK : S_FALSE;
   }

   STDAPI DllGetClassObject(REFCLSID rclsid, REFIID riid, LPVOID FAR* ppv)
   {
       return Module<InProc>::GetModule().GetClassObject(rclsid, riid, ppv);
   }
   ```

More info on DLLs and linking can be found on [http://www.tenouk.com/ModuleBB.html](http://www.tenouk.com/ModuleBB.html)

### MIDL and MIDLRT

Blah

### Windows Runtime Template Language (WRL)

In most situations, it is recommended to use C++/CX to interact with the Windows Runtime. But in the case of hybrid components that implement both COM and Windows Runtime interfaces, such as Media Foundation objects, this is not possible. C++/CX can only create Windows Runtime objects. So, for hybrid objects it is recommended that you use WRL to interact with the Windows Runtime. Be aware that Windows Runtime C++ Template Library has limited support for implementing COM interfaces.

* [http://msdn.microsoft.com/en-us/library/vstudio/hh438466.aspx](http://msdn.microsoft.com/en-us/library/vstudio/hh438466.aspx)
* [http://channel9.msdn.com/Events/Windows-Camp/Developing-Windows-8-Metro-style-apps-in-Cpp/The-Windows-Runtime-Library-WRL-](http://channel9.msdn.com/Events/Windows-Camp/Developing-Windows-8-Metro-style-apps-in-Cpp/The-Windows-Runtime-Library-WRL-)

## Important interfaces

__Media Pipeline__

* Media Source - IMFMediaSource
* MFT - IMFTransform
* Media Sink - IMFMediaSink

__Reader/Writer__

* Source Reader - IMFSourceReader
* Sink Writer - IMFSinkWriter

## Threading

MF is a free-threaded system, which means that COM interface methods can be invoked from arbitrary threads. Therefore, when calling CoInitializeEx(), you must initialize COM with the apartment-threaded object concurrency by passing in the COINIT_APARTMENTTHREADED parameter. Your objects might also need to use synchronization primitives, such as locks, to control access to internal variables by concurrently running threads.

See also [http://msdn.microsoft.com/en-us/library/windows/desktop/ee892371(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/windows/desktop/ee892371(v=vs.85\).aspx)
