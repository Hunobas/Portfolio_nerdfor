# 박태훈 | Unity 클라이언트 프로그래머

> 1998년생, 군필 (의경 만기제대)  
> 📧 hunobas.dev@gmail.com

> 액션 게임의 타격감과 캐릭터 시스템에 관심이 많으며,  
> 기획자가 코드 없이 콘텐츠를 빠르게 추가할 수 있는 데이터 기반 설계를 즐깁니다.

---

## 너드포 지원동기

Monster Hunter 시리즈를 600시간 이상, Elden Ring · Bloodborne · God of War 시리즈를 각각 40시간 이상 플레이하며 하드코어 액션의 타격감과 몰입감에 매료되어 왔습니다.

그 경험이 쌓이며 자연스럽게 **"이 액션을 직접 만드는 사람"** 이 되고 싶다는 목표가 생겼습니다. 글로벌 IP의 매력적인 캐릭터와 밀도 높은 전투를 Unity로 구현하는 Project D에 지원하게 된 이유입니다.

---

## 핵심 역량

### 1. 액션 & 애니메이션 시스템

**My Little Puppy (드림모션 인턴, Unity, 38명)** 에서 NPC 루트모션 FSM 관련 버그 3건을 해결했습니다. 공통적으로 외부 모듈이 캐릭터 트랜스폼을 오염시키는 패턴이었으며, 수정 후 캐릭터 트랜스폼 업데이트 메서드의 코드 컨벤션을 팀에 제안했습니다.

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

- 🐶 [드림모션 경력기술서 | Notion](https://ethereal-judo-1f1.notion.site/My-Little-Puppy-1c6486e2cdb980fcbc33f487a01bd7fc)

---

### 2. Unity 시스템 설계 & 기획자 친화적 파이프라인

**목성의 노래 (Unity, 5명 팀)** 에서 FSM 기반 인터랙션 아키텍처를 설계하고, 기획자가 코드 없이 퍼즐 로직을 추가할 수 있는 ScriptableObject 기반 구조를 구현했습니다.

뱀서라이크 개인 프로젝트 **TOGU: Planet Survivors** 에서는 3주 만에 빈 템플릿부터 아키텍처 설계·구현을 완성했습니다.

| 시스템 | 구현 내용 |
|--------|----------|
| 오브젝트 풀링 | 제네릭 타입 지원, GC 호출 빈도 80% 감소 확인 |
| 데이터 기반 설계 | 코드 수정 없이 몬스터/아이템 추가 가능한 파이프라인 |
| AI 시스템 | 전략 패턴 기반 적 행동 교체 |

- [신규 무기/아이템 추가 가이드 (Notion)](https://ethereal-judo-1f1.notion.site/223486e2cdb980c5a807f920ebad70a6)
- [신규 몬스터 추가 가이드 (Notion)](https://ethereal-judo-1f1.notion.site/223486e2cdb98001869cef28bb9bfbb5)

---

### 3. 네트워크 연동 경험 (우대사항)

- **Photon + Firebase** 기반 체스 멀티플레이어 토이 프로젝트 — 실시간 방 매칭 및 게임 상태 동기화 구현  
  [GitHub](https://github.com/Hunobas/Chess_App_Unity)
- **크래프톤 정글 팀 프로젝트 THE RATTUS** — WebSocket 기반 풀스택 웹 게임, 아키텍처 설계 주도  
  [GitHub](https://github.com/younggun339/jungleTwo)

---

### 4. 렌더링 최적화

400만 버텍스 + 300개 머티리얼 씬의 FPS 불안정 문제를 해결했습니다. MeshBaker로 방 단위 텍스처 아틀라스 + 메쉬 콤바인 적용, 오클루전 컬링 설정.

<img width="1548" height="591" alt="최적화 전후" src="https://github.com/user-attachments/assets/6b35a453-6a45-4258-9635-3bcff6062e97" />

| 지표 | Before | After |
|------|--------|-------|
| Batches | 2,623 | 910 |
| FPS | 30~60 | 120+ |

- [최적화 개발일지 | Velog](https://velog.io/@po127992/목성의-노래-MeshBaker-최적화-삽질기-텍스처-아틀라스만-vs-콤바인-메쉬까지)

---

## 프로젝트 요약

| 프로젝트 | 엔진 | 기간 | 규모 | 역할 |
|----------|------|------|------|------|
| [목성의 노래](https://github.com/Hunobas/Song-Of-Jupitor) | Unity | 2025.06~2026.01 | 5명 | FSM 아키텍처, 렌더링 최적화, 퍼즐 로직 |
| [My Little Puppy](https://ethereal-judo-1f1.notion.site/My-Little-Puppy-1c6486e2cdb980fcbc33f487a01bd7fc) | Unity | 2025.01~03 | 38명 | 루트모션 버그 해결, 슈퍼점프, 에디터 확장 |
| [TOGU: Planet Survivors](https://github.com/Hunobas/Planet) | Unreal 5.4 | 2025.04~06 | 1명 | 전체 아키텍처 설계 및 구현 |

---

## 플레이 이력

| 게임 | 플레이타임 |
|------|------------|
| Monster Hunter: World | 564h |
| Monster Hunter: Wilds | 52h |
| God of War 시리즈 | 70h+ |
| Bloodborne | 40h+ |
| Elden Ring | 40h+ |

---

## Contact

- 휴대폰: 010-3702-1279
- 이메일: hunobas.dev@gmail.com
- 블로그: [Velog](https://velog.io/@po127992/posts)
- 깃허브: [GitHub](https://github.com/hunobas)
