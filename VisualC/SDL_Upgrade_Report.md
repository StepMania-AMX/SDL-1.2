# SDL 1.2 Project File Upgrade Report

## 1. Introduction

This report details the analysis of the Visual Studio project files for the forked SDL 1.2 library. The goal is to identify the necessary modifications to ensure compatibility with the StepMania project when building with Visual Studio 2022. The analysis compares the new project files (`SDL-1.2`) with the original ones (`SDL-1.2.15`) and uses `StepMania.vcxproj` as a reference for a valid configuration.

## 2. Executive Summary

The new SDL project files have been updated to use the Visual Studio 2022 toolset (`v143`), which is a necessary first step. However, several critical settings were lost or changed during this process, leading to build failures. The most important changes to be made are:

-   **Revert Output Directories**: The output paths for the generated libraries must be changed back to match the location expected by the main `StepMania.vcxproj` file.
-   **Correct Runtime Library**: The C++ runtime library must be set to static linkage (`/MT` and `/MTd`) instead of the incorrect DLL linkage (`/MD` and `/MDd`).
-   **Restore Compiler Optimizations**: Key compiler flags, such as `/arch:SSE2` and multiprocessor compilation, must be re-enabled to match the performance settings of the main project.
-   **Standardize Configurations**: Simplify and align the build configurations, removing legacy and redundant options.

## 3. Detailed Analysis

### 3.1. `SDL.vcxproj` Analysis

| Setting | Old (SDL-1.2.15) | New (SDL-1.2) | Recommendation | Justification |
| :--- | :--- | :--- | :--- | :--- |
| **Platform Toolset** | `v141_xp` | `v143` | **Keep `v143`** | Required for Visual Studio 2022 compatibility. It is advisable to upgrade `StepMania.vcxproj` to `v143` as well for solution-wide consistency. |
| **Project Configurations** | Multiple legacy configs | Simplified `Debug`/`Release` for `Win32`/`x64` | **Keep new configurations** | The simplified configurations are cleaner and sufficient for the project's needs. |
| **Output Directory** | `$(SolutionDir)3rdparty\_lib\$(ProjectName)\$(Platform)\` | `$(Platform)\$(Configuration)\` | **Revert to old path** | `StepMania.vcxproj` linker settings point to the old path. Reverting is essential to ensure the libraries are found during the build. |
| **Runtime Library** | `MultiThreadedDebug` (`/MTd`), `MultiThreaded` (`/MT`) | `MultiThreadedDLL` (`/MDd`), `MultiThreadedDLL` (`/MD`) | **Revert to old settings** | `StepMania.vcxproj` uses the static runtime library. Mismatched runtimes will cause linker errors. |
| **Compiler Optimizations** | `StreamingSIMDExtensions2`, `MultiProcessorCompilation` | Disabled | **Restore old settings** | Ensures performance consistency with the main application and improves build times. |
| **Post-Build Event** | Copies DLL to `Program` folder | None | **Remove** | Not needed if the output directory is set correctly. The main project will handle final file placement. |
| **Source Files** | Missing `version.rc` | Includes `version.rc` | **Use new file** | Provides version information for the compiled library. |

### 3.2. `SDLmain.vcxproj` Analysis

| Setting | Old (SDL-1.2.15) | New (SDL-1.2) | Recommendation | Justification |
| :--- | :--- | :--- | :--- | :--- |
| **Platform Toolset** | `v141_xp` | `v143` | **Keep `v143`** | Required for Visual Studio 2022 compatibility. |
| **Project Configurations** | Multiple legacy configs | Simplified `Debug`/`Release` for `Win32`/`x64` | **Keep new configurations** | Modernizes the project and removes unnecessary complexity. |
| **Output Directory** | `$(SolutionDir)3rdparty\_lib\$(ProjectName)\$(Platform)\` | `$(Platform)\$(Configuration)\` | **Revert to old path** | Critical for the StepMania linker to find the `SDLmain.lib` file. |
| **Runtime Library** | `MultiThreadedDebug` (`/MTd`), `MultiThreaded` (`/MT`) | `MultiThreadedDLL` (`/MDd`), `MultiThreadedDLL` (`/MD`) | **Revert to old settings** | `SDLmain` is a static library and must be linked with the same runtime as the main executable. |
| **Preprocessor** | `NO_STDIO_REDIRECT` | Missing from main configs | **Add `NO_STDIO_REDIRECT`** | This was a standard definition in the old project and should be restored to maintain behavior. |
| **Compiler Optimizations** | `StreamingSIMDExtensions2`, `MultiProcessorCompilation` | Disabled | **Restore old settings** | Ensures performance consistency and faster builds. |

## 4. Recommendations

The following is a checklist of the actions required to fix the SDL project files.

-   **For both `SDL.vcxproj` and `SDLmain.vcxproj`:**
    -   [x] Change the `OutDir` property for all configurations to `$(SolutionDir)3rdparty\_lib\$(ProjectName)\$(Platform)\`.
    -   [x] Change the C/C++ `Runtime Library` to `Multi-threaded (/MT)` for `Release` configurations and `Multi-threaded Debug (/MTd)` for `Debug` configurations.
    -   [x] Enable `Enable Enhanced Instruction Set` to `Streaming SIMD Extensions 2 (/arch:SSE2)`.
    -   [x] Set `Multi-processor Compilation` to `Yes`.

-   **For `SDLmain.vcxproj`:**
    -   [x] Add `NO_STDIO_REDIRECT` to the preprocessor definitions for all configurations.

## 5. Conclusion

By implementing the changes outlined in this report, the SDL 1.2 project files will be correctly configured for use with the StepMania project in Visual Studio 2022. These modifications align the build settings, ensure proper library linkage, and restore key performance optimizations, which will resolve the build failures.
