# ReachExtender (fork)

> **원작자 · 원본**
> **Mattdokn** — https://hub.sp-tarkov.com/files/file/1758-reach-extender/
>
> 이 레포는 위 원작의 **포크**입니다. 기능은 그대로고, **SPT 4.1에서 빌드·동작하도록
> 포팅**한 것이 전부입니다.

루팅/문 상호작용 사거리를 늘려주는 플러그인입니다. 스피어 캐스팅으로 조준이 살짝
빗나가도 집히게 하고, 바닥 아래 아이템도 설정한 거리까지 주울 수 있습니다.

현재 기준 **SPT 4.1**.

![Sphere casting](Reach%20Extender%20Sphere%20Casting.gif)

---

## 4.1 포팅에서 바뀐 것

**타입 이름 하나:** `EFTPhysicsClass` → `PhysicsExtensions` (3곳)

독립된 두 출처로 교차 확인했습니다:

| 출처 | 결과 |
|---|---|
| SPT 4.1 wiki `Class_Name_Mappings.md` | `` `EFTPhysicsClass` `` → `` `PhysicsExtensions` `` |
| assembly-tool `GClass-Mappings.json5` | 같은 타입, `NewName: "PhysicsExtensions"` |

중첩 타입 목록(`Class421`, `SyncTransformsClass`, `GClass745` …)까지 두 표가 줄 단위로
일치해서, 같은 타입을 가리키는 게 확실합니다. 네임스페이스는 그대로(전역)입니다.

나머지(`EFTHardSettings`, `GameWorld`, `FindInteractable`,
`InteractiveLootMaskWPlayer`, `LootMaskObstruction`, `LOOT_RAYCAST_DISTANCE` 등)는
원래부터 실명이라 리네임 표에 없습니다 = 그대로입니다.

### 이 리네임이 전부라는 근거

레포에 4.0 시절 `Assembly-CSharp.dll`이 같이 들어 있어서, 그걸로 실제 컴파일을 돌려
확인했습니다:

| | 결과 |
|---|---|
| 원본 소스 × 4.0 어셈블리 | **빌드 성공** |
| 리네임 후 × 4.0 어셈블리 | `error CS0103: The name 'PhysicsExtensions' does not exist` **한 개만** |

즉 4.0에서 깨지는 이름은 딱 이것 하나고, 나머지 코드는 4.0/4.1 양쪽에서 그대로
유효합니다.

---

## 빌드 설정도 갈아엎었습니다

- 구식 csproj → SDK 스타일, `net471` → `netstandard2.1`
- **레포에 들어있던 `References/` 폴더(14MB) 삭제.** 4.0 시절 DLL들이라 그걸로 빌드하면
  4.1에 없는 타입을 향해 컴파일된 플러그인이 나옵니다. 게다가 4.0 `Assembly-CSharp`는
  이 프로젝트의 모든 이름을 아무 불평 없이 해결해주기 때문에 **조용히** 잘못된 결과가
  나옵니다. 참조는 실제 설치본에서 가져오도록 바꿨습니다
- `Properties/AssemblyInfo.cs` 삭제, 값은 csproj로 이전
- 릴리스 zip을 압축 대상 폴더 **밖에** 쓰도록 수정 (안에 쓰면 자기가 쓰는 파일을 읽으려
  들어서 "다른 프로세스에서 사용 중" 오류가 납니다)

## 빌드

```
dotnet build ReachExtender.sln
```

경로는 `SptRoot`에서 나옵니다. 기본값 `E:\SPT 4.1`, `-p:SptRoot=...` 또는 동명의
환경변수로 덮어쓸 수 있습니다. 경로가 틀리면 이유를 말해줍니다:

```
error : SptRoot 'D:\wrong' does not look like an SPT install
        (missing EscapeFromTarkov_Data\Managed\Assembly-CSharp.dll).
```

빌드하면 `$(SptRoot)\BepInEx\plugins\`로 바로 복사되고 `Release\ReachExtender.zip`도
만들어집니다.

## 설정 (F12)

| 항목 | 기본값 |
|---|---|
| Loot/Container Interaction Distance | 2.0 (게임 기본 1.3) |
| Door Interaction Distance | 1.0 |
| How far to reach below floors to loot | 0.75 (0 = 끄기) |
| Sphere Cast Radius | 0.1 (0 = 끄기) |
| Show debug sphere | 꺼짐 |
