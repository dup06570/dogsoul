<h1 align="center">DogSoul</h1>
<p align="center">Unity 기반 멀티플레이 로그라이트 액션 RPG</p>

<p align="center" style="line-height: 2;">
    <img src="https://img.shields.io/badge/Unity-000000?style=for-the-badge&logo=Unity&logoColor=white">
    <img src="https://img.shields.io/badge/C%23-512BD4?style=for-the-badge&logo=csharp&logoColor=white">
    <img src="https://img.shields.io/badge/Photon-0000FF?style=for-the-badge&logo=Photon&logoColor=white">
</p>

> 본 Repository는 포트폴리오 제출을 위해 실제 팀 프로젝트 Repository를 Fork한 저장소입니다.

<br/>

## 시연
> 팀 프로젝트 공식 시연 영상
[![Dogsoul 소개영상](https://img.youtube.com/vi/OtntBaEmVDU/0.jpg)](https://youtu.be/OtntBaEmVDU)

<br/>

## 목차

- [프로젝트 소개](#프로젝트-소개)
- [담당 역할](#담당-역할)
- [주요 구현](#주요-구현)
- [시스템 구조도](#시스템-구조도)
- [주요 코드](#주요-코드)


<br/>

## 프로젝트 소개

DogSoul은 소울라이크의 전투 요소에 멀티플레이와 로그라이트 성장 요소를 결합한 액션 RPG 프로젝트입니다.

플레이어는 마을에서 장비를 준비한 뒤 던전에 진입하여 몬스터와 전투하고, 아이템과 경험치를 획득하며 성장합니다.

전투에서는 공격, 회피, 락온 등의 액션 요소를 구현했으며,
Photon을 활용하여 여러 플레이어가 함께 던전을 진행할 수 있도록 멀티플레이 시스템을 구축했습니다.

| 항목 | 내용 |
| --- | --- |
| 장르 | Multiplayer Roguelite Action RPG |
| 엔진 | Unity |
| 언어 | C# |
| 네트워크 | Photon |
| 백엔드 | Firebase |
| 개발 기간 | 2025.01 ~ 2025.05 |
| 개발 인원 | 3명 |

<br/>

## 담당 역할

**클라이언트 / 네트워크 프로그래밍**

본 프로젝트에서 아래 기능을 직접 설계 및 구현했습니다.

- Inventory System 전체 설계 및 구현
- Photon 기반 멀티플레이 로비 및 Room 생성/참가 시스템 구현
- 플레이어, 아이템, 몬스터의 멀티플레이 동기화
- 던전 진입 시 절차적 맵 생성 및 멀티플레이 환경 동기화

> 팀 프로젝트이므로 아래의 `주요 구현`과 `주요 코드`에서는
> 제가 직접 개발한 부분을 중심으로 설명합니다.

<br/>

## 주요 구현

### 1. Inventory System

아이템의 획득부터 보관, 장착, 사용, 드롭까지 관리하는
Inventory System의 전체 구조를 설계하고 구현했습니다.

- 드래그 앤 드롭을 이용한 아이템 이동
- 인벤토리 / 무기 / 소비 아이템 / 가방 슬롯 관리
- 아이템 장착 및 사용 기능
- 필드 아이템 획득 및 드롭 처리
- 멀티플레이 환경의 드롭 아이템 연동

<p align="center">
    <!-- 해당 기능을 보여주는 이미지 또는 GIF가 있다면 추가 -->
</p>

<br/>

### 2. Photon Lobby & Room System

[어떤 기능인지 간략하게 설명]

마을에서 포탈을 통해 던전에 진입할 때 Photon 서버에 접속하고,
방을 생성하거나 다른 플레이어가 생성한 방에 참가하여
함께 던전을 시작할 수 있는 시스템을 구현했습니다.

- Photon Server 및 Lobby 연결
- Room 생성 및 참가
- 생성된 Room 목록 관리
- Room 내부 플레이어 관리
- 멀티플레이 던전 시작 흐름 처리

<br/>

### 3. Multiplayer Synchronization

Photon을 이용해 멀티플레이 환경에서
플레이어, 아이템, 몬스터의 상태와 행동을 동기화했습니다.

- 플레이어 캐릭터 및 행동 동기화
- 장착 무기 및 아이템 상태 동기화
- 필드 드롭 아이템 동기화
- 몬스터 생성 및 상태 동기화

<br/>

### 4. Procedural Dungeon Generation

던전 진입 시 여러 형태의 방을 조합해
맵을 절차적으로 생성하는 시스템을 구현했습니다.

멀티플레이에서는 Master Client를 기준으로 던전을 생성하고,
참가한 플레이어들이 동일한 던전에서 시작할 수 있도록
Photon의 멀티플레이 흐름과 연계했습니다.

<br/>



## 시스템 구조도

<p align="center">
  <img src="./page_image/systemStructure.png" alt="Architecture Diagram" width="650"/>
</p>

[여기에 구조도를 설명하는 내용을 2~4줄 정도 작성]

예시:

- 플레이어의 전투 및 상태 관련 기능은 각 시스템별 컴포넌트로 관리합니다.
- Photon을 통해 멀티플레이에 필요한 플레이어 상태와 행동을 동기화합니다.

<br/>

## 주요 코드

제가 담당한 기능을 확인할 수 있는 주요 코드 위치입니다.

| 기능 | 주요 코드 | 설명 |
| --- | --- | --- |
| [기능 1] | `Assets/.../...cs` | [해당 클래스 역할] |
| [기능 2] | `Assets/.../...cs` | [해당 클래스 역할] |
| [기능 3] | `Assets/.../...cs` | [해당 클래스 역할] |
| [기능 4] | `Assets/.../...cs` | [해당 클래스 역할] |

<br/>
