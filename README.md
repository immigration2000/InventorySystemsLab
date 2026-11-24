# InventorySystemsLab

Unreal C++ inventory systems course workspace.
The Unreal project and runtime module are named `Project`; the targets are
`Project` and `ProjectEditor`.

## Requirements

- Unreal Engine 5.8 (local engine: 5.8.4).
- Visual Studio C++ game development tools and a compatible Windows SDK.
- Git and Git LFS.

## Setup and build

Open `Project/Project.uproject`. Generate IDE project files when needed.
On a machine with write access to the repository's `.git` directory, run
`git lfs install --local`; after cloning asset content, run `git lfs pull`.

```powershell
$projectFile = Join-Path (Get-Location) 'Project\Project.uproject'
$buildLogRoot = Join-Path (Get-Location) 'Project\Saved\Logs'
New-Item -ItemType Directory -Path $buildLogRoot -Force | Out-Null
$buildLog = Join-Path $buildLogRoot ('Baseline-Editor-' + (Get-Date -Format 'yyyyMMdd-HHmmss') + '.log')
& 'C:\Program Files\Epic Games\UE_5.8\Engine\Build\BatchFiles\Build.bat' `
    ProjectEditor Win64 Development "-Project=$projectFile" "-Log=$buildLog" `
    -WaitMutex -NoHotReloadFromIDE -MaxParallelActions=1
if ($LASTEXITCODE -ne 0) { throw 'Editor build failed.' }
```

Generated binaries, caches, Saved files and IDE solutions are excluded from Git.
Source, Config, Content, project descriptors, `.vsconfig` and Build resources are
preserved. Plugin binaries are preserved for binary-only plugins.
Unreal packages and source media use Git LFS attributes.

See `Docs/BASELINE_BUILD.md` for the actual verification status and remaining work.
Course UI and inventory gameplay have not been implemented in this baseline.
