Goal: Create an equivalent of `mcr.microsoft.com/dotnet/framework/sdk:4.8.1` where all dependencies can be installed in non standard, non root locations for CNB

Reference dockerfile: https://github.com/microsoft/dotnet-framework-docker/blob/main/src/sdk/4.8/windowsservercore-ltsc2019/Dockerfile

## Reference Assemblies

**Status**: Resolved

Used for building .NET framework. Sourced in docker file like so

```
RUN powershell " `
    $ErrorActionPreference = 'Stop'; `
    $ProgressPreference = 'SilentlyContinue'; `
    @('4.0', '4.5.2', '4.6.2', '4.7.2', '4.8', '4.8.1') `
    | %{ `
        Invoke-WebRequest `
            -UseBasicParsing `
            -Uri https://dotnetbinaries.blob.core.windows.net/referenceassemblies/v${_}.zip `
            -OutFile referenceassemblies.zip; `
        Expand-Archive referenceassemblies.zip -DestinationPath \"${Env:ProgramFiles(x86)}\Reference Assemblies\Microsoft\Framework\.NETFramework\"; `
        Remove-Item -Force referenceassemblies.zip; `
    }"
```

MSBuild resolves these via `GetReferenceAssemblyPaths` target ([Task](https://github.com/dotnet/msbuild/blob/main/src/Tasks/GetReferenceAssemblyPaths.cs), [Target](https://github.com/dotnet/msbuild/blob/f422d8d7dfe0a7115b11b31470215ad6b7723138/src/Tasks/Microsoft.Common.CurrentVersion.targets#L1259)). 

```
Target Name=GetReferenceAssemblyPaths Project=FunnyQuotesLegacyCommon.csproj
    TargetFrameworkDirectory = 
    Using "GetReferenceAssemblyPaths" task from assembly "Microsoft.Build.Tasks.Core, Version=15.1.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a".
    GetReferenceAssemblyPaths
        Parameters
            TargetFrameworkMoniker = .NETFramework,Version=v4.8
            RootPath = c:\temp\Framework
        OutputProperties
            _TargetFrameworkDirectories = c:\temp\Framework\.NETFramework\v4.8\
            _FullFrameworkReferenceAssemblyPaths = c:\temp\Framework\.NETFramework\v4.8\
            TargetFrameworkMonikerDisplayName = .NET Framework 4.8
    TargetFrameworkDirectory = c:\temp\Framework\.NETFramework\v4.8\;;
    RemoveAssemblyFoldersIfNoTargetFramework = true
    DesignTimeFacadeDirectoryRoots
        c:\temp\Framework\.NETFramework\v4.8\
    DesignTimeFacadeDirectories
        c:\temp\Framework\.NETFramework\v4.8\Facades\
    TargetFrameworkDirectory = c:\temp\Framework\.NETFramework\v4.8\;;;c:\temp\Framework\.NETFramework\v4.8\Facades\
```



**Solution:** Set MSBuild property `TargetFrameworkRootPath` to folder that maps to files that exist in `C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework`
