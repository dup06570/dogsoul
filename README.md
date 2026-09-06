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
[![DogSoul 소개영상](https://img.youtube.com/vi/OtntBaEmVDU/0.jpg)](https://youtu.be/OtntBaEmVDU)

<br/>

## 목차

- [프로젝트 소개](#프로젝트-소개)
- [담당 역할](#담당-역할)
- [주요 구현](#주요-구현)
- [시스템 구조도](#시스템-구조도)
- [Troubleshooting](#troubleshooting)
- [개발 회고](#개발-회고)

<br/>

## 프로젝트 소개

DogSoul은 소울라이크의 전투 요소에 멀티플레이와 로그라이트 성장 요소를 결합한 액션 RPG 프로젝트입니다.

플레이어는 마을에서 장비를 준비한 뒤 던전에 진입하여 몬스터와 전투하고, 아이템과 경험치를 획득하며 성장합니다.

전투에서는 공격, 회피, 락온 등의 액션 요소를 구현했으며,
Photon을 활용하여 여러 플레이어가 함께 던전을 진행할 수 있도록 멀티플레이 시스템을 구축했습니다.

| 항목 | 내용 |
| --- | --- |
| 장르 | Multiplayer Roguelite Action RPG |
| 엔진 | Unity 2022.3.56f1 |
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

> 팀 프로젝트이므로 아래의 `주요 구현`에서는
> 제가 직접 개발한 부분을 중심으로 설명합니다.

<br/>

## 주요 구현

### 1. Inventory System

아이템의 획득, 보관, 장착, 사용, 드롭까지 처리하는
Inventory System 전체를 설계하고 구현했습니다.

공통 `ItemPanel`을 기반으로 Inventory, Weapon, Backpack, UseItem 등
각 패널의 역할을 분리하고, 아이템이 패널 사이를 이동할 때
기존 패널에서 데이터를 제거한 뒤 새로운 패널의 규칙에 따라 처리하도록 구성했습니다.

- 드래그 앤 드롭 기반 아이템 이동
- 무기 / 가방 / 소비 아이템 장착 및 사용
- 아이템 획득 및 필드 드롭
- 가방에 따른 최대 소지 공간 관리
- 창고 및 상점과 Inventory 간 아이템 이동

#### 핵심 코드

`InventoryPanel`에 아이템이 들어올 때 기존 Panel에서 아이템을 제거하고,
Inventory 데이터와 소지 공간을 함께 갱신합니다.

```csharp
public override void InsertItem(ItemIcon itemIcon)
{
    if (itemIcon.itemPanel != null)
    {
        itemIcon.itemPanel.TakeOutItem(this, itemIcon);
    }

    itemIcon.transform.parent = content.transform;
    itemIcon.itemPanel = this;

    InventoryController.Instance.inventory.Add(
        itemIcon.GetComponent<ItemIcon>().item
    );

    InventoryController.Instance.SetInventorySizeRate();
    InventoryController.Instance.RemoveItemsUntilUnderMaxWeight();
}
```

**관련 코드**
- `Scripts/UI/Inventory/InventoryController.cs`
- `Scripts/UI/Inventory/ItemIconPanel/ItemPanel.cs`
- `Scripts/UI/Inventory/ItemIconPanel/InventoryPanel.cs`
- `Scripts/UI/Inventory/ItemIconPanel/Weaponpanel.cs`
- `Scripts/UI/Inventory/Item/ItemIconInteract.cs`

<br/>

### 2. Photon Lobby & Room System

마을에서 포탈을 통해 던전에 진입할 때 Photon 서버에 접속하고,
Room을 생성하거나 다른 플레이어가 만든 Room에 참가하여
함께 던전을 시작할 수 있는 멀티플레이 흐름을 구현했습니다.

- Photon Server 및 Lobby 연결
- Room 생성 / 참가
- Room 목록 갱신
- Room 내부 플레이어 목록 관리
- 공개 / 비공개 Room 처리
- Master Client를 통한 멀티플레이 던전 시작

#### 핵심 코드

Photon의 Callback을 이용해 서버 접속 후 Lobby로 진입하고,
Room 생성 및 참가를 위한 네트워크 흐름을 구성했습니다.

```csharp
public void JoinedServer()
{
    PhotonNetwork.GameVersion = gameVersion;
    PhotonNetwork.ConnectUsingSettings();
}

public override void OnConnectedToMaster()
{
    PhotonNetwork.JoinLobby();
}

public override void OnJoinedLobby()
{
    NetworkController.Instance.SetNickName();
}
```

Room 목록이 변경되면 Photon에서 전달받은 `RoomInfo`를 기준으로
UI를 생성하거나 제거하도록 구성했습니다.

```csharp
public override void OnRoomListUpdate(List<RoomInfo> roomList)
{
    foreach (RoomInfo info in roomList)
    {
        if (info.RemovedFromList)
        {
            NetworkController.Instance.RemoveRoomUI(info);
            continue;
        }

        if (!roomDictionary.ContainsKey(info.Name))
        {
            roomDictionary.Add(
                info.Name,
                NetworkController.Instance.CreateRoomUI(info)
            );
        }
    }
}
```

**관련 코드**
- `Scripts/Network/NetworkController.cs`
- `Scripts/Network/NetworkCallback.cs`
- `Scripts/Network/CreateRoomSettingPanel.cs`
- `Scripts/Network/PlayerRoom.cs`
- `Scripts/Network/RoomPanel.cs`

<br/>

### 3. Multiplayer Synchronization

Photon의 `PhotonView`와 RPC를 활용하여
플레이어, 필드 아이템, 몬스터가 멀티플레이 환경에서 동일한 상태를
유지하도록 구현했습니다.

플레이어는 `PhotonView.IsMine`을 기준으로 자신의 캐릭터에만
입력과 이동 제어 권한을 부여했으며,
네트워크 객체의 상태 변경이 필요한 경우 RPC를 사용해 다른 Client에 전달했습니다.

- 로컬 / 원격 플레이어 입력 권한 분리
- 플레이어 공격 행동 동기화
- 드롭 아이템 데이터 및 활성 상태 동기화
- 몬스터 네트워크 생성 및 사망 상태 동기화

#### 핵심 코드

드롭 아이템 데이터 변경 시 Master Client가 변경 내용을 처리하도록 하고,
일반 Client에서 발생한 변경은 Master Client에 요청한 뒤
RPC로 다른 Client에 전달하도록 구성했습니다.

```csharp
public override void SetItem(Item item)
{
    string itemData = ChangeData(item);

    if (PhotonNetwork.IsMasterClient)
    {
        UpdateItem(item);
        photonView.RPC(
            "UpdateItemPhoton",
            RpcTarget.OthersBuffered,
            itemData
        );
    }
    else
    {
        photonView.RPC(
            "RequestUpdateItemPhoton",
            RpcTarget.MasterClient,
            itemData
        );
    }
}
```

몬스터는 Room Object로 생성하여 참가한 플레이어가 동일한
네트워크 객체를 공유하도록 했습니다.

```csharp
public override void SetSpawn()
{
    foreach (var dictionary in monsterSpawnArray)
    {
        float randomValue = UnityEngine.Random.Range(0f, 1f);

        if (randomValue <= dictionary.Value)
        {
            PhotonNetwork.InstantiateRoomObject(
                $"Prefabs/Enemys/Multiplay/{dictionary.Key.name}",
                transform.position,
                Quaternion.identity
            );
        }
    }
}
```

**관련 코드**
- `Scripts/Player/Control/PlayerController_M.cs`
- `Scripts/UI/Inventory/Item/DropItem_M.cs`
- `Scripts/UI/Inventory/Money/MoneyDropItem_M.cs`
- `Scripts/Map/Dungeon/MonsterRandomSpawner_M.cs`
- `Scripts/Enemy/EnemyHealth.cs`

<br/>

### 4. Procedural Dungeon Generation

Room, Hallway, Stair, Trap Room 등 여러 종류의 맵 모듈을 연결하여
던전을 절차적으로 생성하는 시스템을 구현했습니다.

멀티플레이에서는 각 Client가 따로 맵을 생성하지 않고
**Master Client가 던전을 생성**하도록 구성했으며,
생성된 맵 구성 요소를 Photon Room Object로 생성하여
모든 플레이어가 동일한 던전을 공유하도록 했습니다.

#### 핵심 코드

Master Client는 던전 생성과 오브젝트 배치를 담당하고,
다른 Client는 던전 생성이 완료될 때까지 대기한 뒤
Master Client에 자신의 Spawn 위치를 요청합니다.

```csharp
public override void StartGeneration()
{
    photonView = GetComponent<PhotonView>();
    NetworkController.Instance.AllPanelActiveFalse();

    if (PhotonNetwork.IsMasterClient)
    {
        networkEventReceiver.playerCount =
            NetworkController.Instance.playerCount;

        Generate_MultiPlay();
        FillWall();
        NvigationBake();
        PlayerSpawn();
        SpawnRandomObject();
        BossRoomSetting();
    }
    else
    {
        networkEventReceiver.RequestPlayerSpawn();
        StartCoroutine(WaitHostReady());
    }

    isGenerated = true;
}
```

Client의 Spawn 요청은 Photon Event로 Master Client에 전달하고,
던전 생성 완료 후 계산된 Spawn 위치를 해당 Client에 다시 전달하도록 구현했습니다.

```csharp
public void RequestPlayerSpawn()
{
    PhotonNetwork.RaiseEvent(
        (byte)NetworkEventCode.RequestPlayerSpawn,
        PhotonNetwork.LocalPlayer.ActorNumber,
        new RaiseEventOptions
        {
            Receivers = ReceiverGroup.MasterClient
        },
        SendOptions.SendReliable
    );
}
```

**관련 코드**
- `Scripts/Map/Dungeon/DungeonGenerator_M.cs`
- `Scripts/Map/Dungeon/NetworkEventReceiver.cs`
- `Scripts/Map/Dungeon/DungeonPart.cs`
- `Scripts/Map/Dungeon/EntryPoint.cs`

<br/>

## 시스템 구조도

> 아래 구조도는 DogSoul 팀 프로젝트 전체의 시스템 구성을 나타냅니다.

<p align="center">
  <img src="./page_image/systemStructure.png" alt="Architecture Diagram" width="650"/>
</p>

- Village와 Dungeon으로 게임 영역을 구분하고, Portal을 통해 멀티플레이 던전에 진입하는 구조입니다.
- Firebase는 로그인 및 플레이어 데이터 저장에 사용했습니다.
- 제가 담당한 Photon 시스템은 Portal에서 Room 생성/참가를 처리하고, Dungeon에서는 플레이어와 네트워크 객체의 동기화를 담당합니다.
- 또한 Dungeon 진입 시 절차적 맵 생성과 플레이어 스폰 흐름을 구현했습니다.

<br/>

## Troubleshooting

### 공격 애니메이션 동기화 누락 문제

**문제**

초기에는 Photon의 Animator 동기화를 이용해 공격 상태를 다른 Client에 전달했습니다.
하지만 짧게 실행되는 공격 상태가 동기화 시점 사이에서 누락되면서,
다른 Client에서 공격 애니메이션이 정상적으로 재생되지 않는 문제가 발생했습니다.

**해결**

Animator 상태 전체의 동기화에 의존하는 대신,
플레이어가 공격을 시작하는 시점에 RPC를 직접 전송하도록 변경했습니다.

로컬에서는 공격 Trigger를 실행한 뒤 `RpcAnimator()`를 다른 Client에 전달하고,
원격 Client에서는 현재 장착 무기를 기준으로 `Attack` 또는 `RangedAttack`
Trigger를 직접 실행하도록 구성했습니다.

```csharp
if (!weapon.isRanged)
    animationHandler.SetTrigger(AnimationHandler.AnimParam.Attack);
else
    animationHandler.SetTrigger(AnimationHandler.AnimParam.RangedAttack);

// 공격 시 다른 Client에 공격 이벤트 전달
photonView.RPC(nameof(RpcAnimator), RpcTarget.OthersBuffered);

// 원격 Client에서 공격 Trigger 실행
[PunRPC]
private void RpcAnimator()
{
    if (!weaponStats.isRanged)
        animationHandler.SetTrigger(AnimationHandler.AnimParam.Attack);
    else
        animationHandler.SetTrigger(AnimationHandler.AnimParam.RangedAttack);
}
```

**결과**

공격 상태 전체를 지속적으로 동기화하는 대신 공격 시작 이벤트를 직접 전달함으로써,
다른 Client에서도 공격 애니메이션이 안정적으로 실행되도록 개선했습니다.
<br/>

## 개발 회고

DogSoul은 처음으로 진행한 팀 게임 프로젝트였고, 당시에는 기능 구현과 빠른 연동에 집중하면서 `InventoryController`가 아이템 보유 상태, UI 갱신, 장착, 드롭, 소지 공간 계산 등 여러 책임을 함께 가지도록 설계했습니다.

프로젝트가 커지면서 하나의 클래스가 많은 역할을 담당할수록 기능 수정 시 영향을 확인해야 하는 범위가 넓어진다는 점을 경험했습니다. 이후 프로젝트에서는 데이터 관리, UI, 기능 실행의 책임을 분리하고 각 클래스가 명확한 역할을 가지도록 설계하는 것을 더 중요하게 고려하게 되었습니다.

이 경험을 통해 기능이 동작하는 것뿐 아니라, 이후의 변경과 확장을 고려한 구조 설계가 중요하다는 점을 배웠습니다.
