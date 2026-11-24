# 초기 설정·기본 빌드 검증 완료

- 확인일: 2026-11-24, 한국 시간
- 저장소: `C:\Users\user1\Documents\ChatGPT\InventorySystemsLab`
- 프로젝트: `Project/Project.uproject`
- 프로젝트·Runtime 모듈·Game Target: `Project` 유지
- Editor Target: `ProjectEditor` 유지
- 브랜치: `master` 유지. 원격 저장소 없음.
- 시작점 커밋 메시지: `chore: establish UE 5.8 baseline with Git LFS`

## 실제 빌드 결과

**ProjectEditor Win64 Development 빌드 성공.** 사용자가 일반 PowerShell에서
실행한 빌드를 로컬 로그·receipt·새 DLL과 대조해 확인했다.

| 항목 | 확인 결과 |
|---|---|
| 엔진 | Unreal Engine 5.8.4, Changelist 58902052 |
| Target / Platform / Configuration | ProjectEditor / Win64 / Development |
| 종료 코드 | 0 — 사용자 PowerShell 출력에서 확인 |
| 로그 결과 | Result: Succeeded |
| 작업 | 7개: Project.cpp 등 컴파일, 라이브러리·DLL 링크, receipt 작성 |
| 전체 실행 시간 | 101.23초 |
| 도구 | VS 2022 MSVC 14.44.35228, Windows SDK 10.0.22621.0 |
| 필수 옵션 | -WaitMutex -NoHotReloadFromIDE -MaxParallelActions=1 |
| 빌드 방식 | 기존 산출물을 보존한 증분 빌드. 실제 컴파일·링크가 수행됨 |
| receipt | Project/Binaries/Win64/ProjectEditor.target, UE 5.8.4 및 요청 대상과 일치 |
| DLL | Project/Binaries/Win64/UnrealEditor-Project.dll, 50,688 bytes |
| DLL 수정 시각 | 2026-11-24 14:48:51 한국 시간 |
| receipt 수정 시각 | 2026-11-24 14:48:52 한국 시간 |

성공 로그: `Project/Saved/Logs/Baseline-UserShell.log`

로그 SHA256:
`28690FBAF949B1CFE6CC8C9F291606C24FC8F9A287FB37CE4F1D98CEACF0827B`

DLL SHA256:
`B6085ABC1F37C2453270B266FF588AE0FD1B57B46CAB39029F6832DA333F39A6`

로그의 핵심 완료 기록:

```text
[1/7] Compile [x64] SharedPCH.UnrealEd.Project.ValApi.ValExpApi.Cpp20.cpp
[2/7] Compile [x64] PerModuleInline.gen.cpp
[3/7] Compile [x64] Project.cpp
[4/7] Link [x64] UnrealEditor-Project.lib
[5/7] Compile Resource [x64] Default.rc2
[6/7] Link [x64] UnrealEditor-Project.dll
[7/7] WriteMetadata ProjectEditor.target [NoUba]
Result: Succeeded
Total execution time: 101.23 seconds
```

재실행용 PowerShell 명령은 저장소 루트에서 실행하며, 로그는 매번 다른 이름으로 저장한다:

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

## Git 설정

- Git LFS 3.7.1. `git lfs install --local` 성공, 종료 코드 0.
- 로컬 LFS 필터와 pre-push 훅 확인.
- `.uasset`, `.umap` 등의 LFS 속성 검사 통과. 가짜 에셋 생성 없음.
- Content 파일은 0개이므로 실제 에셋 포인터·업로드 검증은 수행하지 않았다.
- Binaries, Intermediate, Saved, DerivedDataCache, IDE 캐시·solution 제외 검사 통과.
- Source, Config, Content, .uproject, .vsconfig, Build 리소스는 보존한다.
- 바이너리 전용 플러그인을 위해 플러그인 Binaries는 일괄 제외하지 않는다.
- 시작점에는 원본 파일과 설정·문서만 포함한다. 빌드 산출물과 캐시는 포함하지 않는다.
- 커밋 뒤 Git 상태와 LFS fsck를 확인한다. 원격 push는 수행하지 않는다.

## 백업과 원본 보존

빌드 재시도 전 다음 백업을 만들고 ZIP CRC 및 파일별 SHA256을 검증했다.

- 파일: `InventorySystemsLab_PreBaseline_20261124-144012.zip`
- 범위: 저장소 원본, Git 메타데이터, 기존 UBT 진단 자료 123개 파일.
- 원본 합계: 10,733,714 bytes. 재생성 가능한 프로젝트 바이너리·캐시는 제외.
- ZIP SHA256: `3b456f57c8dbff16021dffb084ce4ab65e11f9445107c7d806445e426bb8e798`
- 백업 파일 ID: `libfile_366bc8fc55b08191bda615969a8f78ac`
- 빌드 직후 Project 하위 원본 11개 파일을 백업과 대조했다. 변경 없음.
- 커밋 전 공백 검사에서 기본 템플릿의 오류가 발견되어 DefaultEngine.ini,
  DefaultInput.ini, Project.Build.cs, Project.h의 줄 끝 공백·파일 끝 빈 줄만 정리했다.
  이후 원본 11개 파일 모두 백업과 공백 차이만 있음을 대조했다.
  이 공백 정리 후 추가 UE 빌드는 실행하지 않았다.
- 엔진 소스 수정, 기존 프로젝트/서비스 종료, 브랜치 변경, 과거 경로 정리는 수행하지 않았다.
- 명시적 Clean/Rebuild는 실행하지 않았다. UBT가 증분 빌드 중 오래된 산출물을 재생성한 것은 정상 빌드 동작이다.

## 해결 과정과 검증 한계

초기 세션에서는 Git 설정 쓰기가 거부됐고, 이후 권한 변경 뒤 LFS 설정에 성공했다.
에이전트가 실행한 빌드는 UBT의 로그 정리 단계에서
`Trace-backup-2026.11.20-02.43.56.uba` 삭제의
`UnauthorizedAccessException`으로 종료됐다.
Windows 수정 권한 반영 뒤에도 이 세션의 AppData 접근 제한은 남았다.

임시 진단 프로그램으로 설치된 UBT 시작 함수의 실제 예외를 포착했다.
엔진 소스·DLL은 변경하지 않았다.
사용자 일반 PowerShell에서 같은 Build.bat 대상·필수 옵션으로 실행한 빌드는 성공했다.
실패한 실행을 성공으로 판정하거나 과거 DLL로 결과를 대신하지 않았다.

진단 기록: `Project/Saved/Logs/Baseline-StartupException-20261124-144047.log`.
Saved의 원본 로그는 Git에서 제외하며 성공 로그의 핵심 출력과 해시를 이 문서에 보존한다.

PIE, 패키징, 네트워크, 게임플레이는 검증하지 않았다.
강의용 UI와 인벤토리 기능은 아직 구현하지 않았다.
다음 단계는 로비 창고·레이드 상자·시체 파밍의 세 UI 시안과 공통 위젯 구조 검토다.
