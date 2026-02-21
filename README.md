# 박태훈 | Unity 클라이언트 프로그래머

> 1998년생, 군필 (의경 만기제대)  
> 📧 hunobas.dev@gmail.com

---

## 너드포 지원동기

Monster Hunter 시리즈를 즐겨 플레이하며 하드코어 액션의 밀도 있는 타격감과 호쾌한 애니메이션의 매력에 오랫동안 빠져 있었습니다.

이 경험이 쌓이며 자연스럽게 **"이 액션을 직접 만드는 사람"** 이 되고 싶다는 목표가 생겼고, 글로벌 IP 특유의 매력적인 캐릭터와 밀도 높은 전투를 Unity로 구현하는 Project D에 지원하게 되었습니다.

---

## 핵심 역량

### 1. 액션 시스템 개발

**My Little Puppy (드림모션 인턴, Unity, 38명)** 에서 슈퍼점프 액션의 조작감을 개선했습니다.

![My Little Puppy 슈퍼점프 액션 시연](https://github.com/user-attachments/assets/d66c3ec8-9fe0-4302-88af-85fa3471fbdb)

기존 구현의 점프 높이가 일정하고 피드백이 없어 조작감이 단조롭다는 피드백이 있었습니다.

| 항목 | 개선 내용 |
|------|----------|
| **물리** | 차징 시간 기반 높이 보간, 런타임 커브로 속도 제어 |
| **애니메이션** | 차징 → 도약 → 착지 단계별 블렌딩 연동 |
| **피드백** | 패드 진동 강도를 차징 시간에 비례, 착지 시 이펙트 재생 |

- 🐶 [드림모션 경력기술서 | Notion](https://ethereal-judo-1f1.notion.site/My-Little-Puppy-1c6486e2cdb980fcbc33f487a01bd7fc)

---

### 2. 네트워크 연동 시스템 개발

**THE RATTUS (크래프톤 정글, Socket.IO / WebSocket | 5인 팀)**

<img width="1042" height="587" alt="THE RATTUS 아키텍처" src="https://github.com/user-attachments/assets/8ea2a253-b519-4eda-ae3d-5767059cb9c1" />

실시간 멀티플레이 웹게임을 빈 템플릿에서 출시까지 완성했습니다. 담당: **아키텍처 설계, 클라이언트-서버 실시간 상태 동기화**.

**아키텍처 구성:**

- **클라이언트 (React, port 3000)** ↔ NGINX 역방향 프록시
- **게임 서버 (NestJS + matter.js, port 8080)** — Socket.IO로 게임 시뮬레이션 상태를 실시간 브로드캐스트
- **모션 캡처 서버 (Flask + OpenCV + MediaPipe, port 5000)** — 웹캠 입력을 분석해 게임 이벤트로 변환, Socket.IO로 게임 서버에 전달
- **회원 서버 (Go + MySQL, port 8787)** — 인증 및 사용자 데이터 담당

- [GitHub](https://github.com/younggun339/jungleTwo)

---

### 3. Unity 클라이언트 개발

#### 3-1. FSM 기반 플레이 모드 아키텍처 (목성의 노래, Unity/C# | 5인 팀)

![GameState 버그 영상](https://github.com/user-attachments/assets/fa973d2f-df58-483d-ae3b-05d5104e9bc6)

**문제:** 5가지 플레이 모드(Normal/Panel/Cinema/Dialog/Pause)가 상호 배타적이어야 하는데, 패널 진입 중 컷씬이 재생되면 컷씬 종료 후에도 **조작 불가** 상태가 되는 버그 발생. 상태 코드가 여러 파일에 분산되어 디버깅에 평균 60분 소요.

**해결:** 중앙 집중식 FSM으로 모든 플레이 모드를 단일 책임 관리.
```csharp
public void ChangePlayMode(IPlayMode next)
{
    if (next == null || ReferenceEquals(_activeMode, next)) return;

    // 시네마 모드는 일시정지 이외 전환 요청 무시
    if (IsPlayingCinema && !ReferenceEquals(next, PauseMode)) return;

    // 패널 모드 강제 종료 후 전환
    if (IsOperatingPanel && PanelMode.Controller != null)
        PanelMode.Controller.EndPanelForcely();

    var prev = _activeMode;
    prev?.OnExit(next);
    _activeMode = next;
    _activeMode.OnEnter(prev);
    InputManager.Instance?.UpdateCursorLock();
}
```

각 모드의 `OnExit` 훅이 UI 상태/입력 바인딩/커서 잠금을 자동 정리합니다.

| 개선 항목 | Before | After |
|------|--------|-------|
| 상태 충돌 버그 | 주 2~3건 | **0건** |
| 디버깅 소요 시간 | 평균 60분 | 평균 30분 |
| 신규 모드 추가 | — | `IPlayMode` 구현만으로 20분 이내 |

📂 [GameState.cs](https://github.com/Hunobas/Song-Of-Jupitor/blob/7386ab978fc3115a13a700758c7a618567bc168a/Scripts/System/GameState.cs#L15)

---

#### 3-2. 렌더링 성능 최적화

400만 버텍스 + 300개 머터리얼 씬에서 FPS 30~60 불안정 문제를 해결했습니다.

<img width="1548" height="591" alt="최적화 전후" src="https://github.com/user-attachments/assets/6b35a453-6a45-4258-9635-3bcff6062e97" />

| 단계 | Batches | FPS |
|------|---------|-----|
| 기준선 | 2,650 | 30~60 |
| MeshBaker 텍스처 아틀라스 + 메쉬 콤바인 | 750 | 80~100 |
| 오클루전 컬링 추가 | 601 | **120+** |

> 텍스처 아틀라스만 적용했을 때는 Verts가 72% 감소했으나 Draw Call이 74% 증가해 오히려 성능이 하락했습니다. **가설 → 측정 → 반증 → 재설계** 사이클로 최적 조합을 확인했습니다.

📜 [최적화 개발일지](https://velog.io/@po127992/목성의-노래-MeshBaker-최적화-삽질기-텍스처-아틀라스만-vs-콤바인-메쉬까지)

---

#### 3-3. 사운드 아키텍처 설계

Unity 기본 AudioSource로는 페이드인/아웃·크로스페이드·Duck(주요 사운드 재생 시 나머지 볼륨 자동 감소)을 매번 코루틴으로 작성해야 했습니다.

![preview](https://github.com/user-attachments/assets/43a81247-8927-4e6c-a465-2a5db6d5be0f)

영상 편집의 트랙/클립 개념을 차용, `SoundSource`(트랙, Inspector 설정)와 `SoundEntry`(클립 엔티티, Fluent API 제어)를 분리했습니다.
```csharp
_soundSource.PlayByNameOrNull("GeneratorStartUp")
    .WithFadeIn(0.5f)
    .WithFadeOut(1.0f)
    .WithPriority(SoundPriority.High)
    .WithOptions(PlayOptions.DuckOthers)
    .OnFinish(() => _soundSource.PlayByNameOrNull("GeneratorLoop")
        .WithLoop().WithOptions(PlayOptions.DuckOthers));
```

크로스페이드 시 페이드아웃되는 클립은 오브젝트 풀의 `AudioSource`로 이관하여 메모리 누수 없이 무한 크로스페이드를 지원합니다.

📂 [SoundManager](https://github.com/Hunobas/Song-Of-Jupitor/blob/main/Scripts/Sound/SoundManager.cs) | [SoundSource](https://github.com/Hunobas/Song-Of-Jupitor/blob/main/Scripts/Sound/SoundSource.cs) | [SoundEntry](https://github.com/Hunobas/Song-Of-Jupitor/blob/main/Scripts/Sound/SoundEntry.cs)

---

### 4. 애니메이션 & 디버깅

**My Little Puppy (드림모션 인턴)** 에서 NPC 루트모션 FSM 관련 버그 3건을 해결했습니다. 공통적으로 외부 모듈이 캐릭터 트랜스폼을 오염시키는 패턴이었으며, 이를 바탕으로 캐릭터 트랜스폼 수정 메서드의 코드 컨벤션을 팀에 제안했습니다.

<details>
<summary><b>사례 1: 루트모션 회전이 FPS에 따라 달라지는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|------|---------|
| ![문제](https://github.com/user-attachments/assets/c038982c-4e66-4c04-a3c1-a4878d3d48c5) | ![정상](https://github.com/user-attachments/assets/f0d4974a-a54d-4b7d-870c-e7b6ad794d5c) |

**원인:** 루트모션 각도 업데이트와 `ActorPosDir` 선형 보간이 중복 적용  
**해결:** 루트모션 전용 위치/각도 업데이트 메서드 분리

</details>

<details>
<summary><b>사례 2: 루트모션 종료 시 캐릭터가 앞으로 튀는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|------|---------|
| ![문제](https://github.com/user-attachments/assets/4e19abda-d7a0-41ea-a84e-c6ffce35ca67) | ![정상](https://github.com/user-attachments/assets/53eb6482-83c2-4f4d-bbe5-2e215c2b7f15) |

**원인:** Walk → RootMotion → Idle 전환 시 속도값이 초기화되지 않고 잔류  
**해결:** 루트모션 종료 시점에 Idle 블렌딩 속도 0 초기화

</details>

<details>
<summary><b>사례 3: 컷씬 일시정지 시 NPC 위치가 튀는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|------|---------|
| ![문제](https://github.com/user-attachments/assets/436f5f08-5091-4b73-bb91-3871885ca447) | ![정상](https://github.com/user-attachments/assets/9bb6594b-afca-43f8-b29a-31fe61e320e3) |

**원인:** 컷씬 일시정지 모드에서 `LateUpdate` 트랜스폼 재조정 처리 누락  
**해결:** 일시정지 진입 전 모드가 컷씬이면 `HandleGameActors()` 호출 추가

</details>

---

## 프로젝트 요약

| 프로젝트 | 엔진 | 기간 | 규모 | 역할 |
|----------|------|------|------|------|
| [목성의 노래](https://github.com/Hunobas/Song-Of-Jupitor) | Unity | 2025.07~12 | 5명 | FSM 아키텍처, 렌더링 최적화, 사운드 시스템 |
| [My Little Puppy](https://ethereal-judo-1f1.notion.site/My-Little-Puppy-1c6486e2cdb980fcbc33f487a01bd7fc) | Unity | 2025.01~03 | 38명 | 슈퍼점프 액션, 루트모션 버그 해결, 에디터 확장 |
| [THE RATTUS](https://github.com/younggun339/jungleTwo) | Web | 2024 | 5명 | 아키텍처 설계, 실시간 상태 동기화 |
| [TOGU: Planet Survivors](https://github.com/Hunobas/Planet) | Unreal 5.4 | 2025.04~06 | 1명 | 전체 아키텍처 설계 및 구현 |

---

## Contact

- 휴대폰: 010-3702-1279
- 이메일: hunobas.dev@gmail.com
- 블로그: [Velog](https://velog.io/@po127992/posts)
- 깃허브: [GitHub](https://github.com/hunobas)
