# BedrockShield

Geyser + Floodgate + Folia 환경에서 Bedrock 플레이어에게 발생하는 서버 측
위치 끼임, 고스트 블록/아이템, 접속 설정 오류를 진단하고 완화하는
Folia-safe 플러그인입니다.

> BedrockShield는 Xbox 인증을 끄거나 위조 계정을 허용하지 않습니다.
> Microsoft/Xbox 인증 서버에 도달하기 전에 발생한 휴대폰 계정 오류는 서버
> 플러그인으로 우회할 수 없습니다.

## 지원 환경

| 구성 요소 | 대상 버전 |
|---|---|
| 서버 | Folia 26.2  |
| JVM | Java 26 권장 (최소 Java 25) |
| Geyser API | 2.12.1-SNAPSHOT |
| Floodgate API | 2.2.5-SNAPSHOT |
| ViaVersion | 5.12.2 이상 권장 |

현재 Geyser는 Minecraft Java 26.3 클라이언트를 모방합니다. 따라서 Folia
26.2 백엔드에서는 ViaVersion이 필요합니다.

- [Paper/Folia Java 요구사항](https://docs.papermc.io/paper/getting-started/)
- [Geyser 지원 버전](https://geysermc.org/wiki/geyser/supported-versions/)
- [Floodgate 공식 설치 안내](https://geysermc.org/wiki/floodgate/setup/)

## 주요 기능

### Bedrock 상태 안정화

- 접속, 리스폰, 텔레포트 후 인벤토리 재동기화
- 완전한 블록 내부에 끼인 플레이어를 위쪽 안전 공간으로 이동
- 초기 위치 동기화 중 월드 최저 높이 아래로 떨어진 플레이어 구조
- 반블록, 계단, 수영, 활강, 탑승 상태는 끼임으로 오인하지 않도록 제외

### 고스트 블록 및 아이템 복구

- 다른 플러그인이 취소한 블록 설치/파괴 결과를 Bedrock 클라이언트에 재전송
- 취소된 클릭, 드래그, 드롭, 손 교체, 핫바 변경 뒤 인벤토리 재전송
- 플레이어별 쿨다운으로 불필요한 패킷 반복 방지

### Geyser/Floodgate 통합 진단

- Geyser 런타임 인증 방식이 `floodgate`인지 확인
- Floodgate `key.pem` 존재 여부 확인
- Folia 26.2에서 ViaVersion 설치 여부 확인
- Geyser 수신 주소, 포트, 지원 Bedrock/Java 버전 표시
- Floodgate 로그인과 충돌할 가능성이 있는 플러그인 감지
- Bukkit 로그인 거절, 서버 킥, Geyser 세션 종료 사유 기록
- 오류 내용에 맞는 한국어 해결 힌트 제공

### 안전한 인증 설정 복구

`/bshield repairauth`는 다음 작업만 수행합니다.

1. Geyser API로 실제 `config.yml` 위치를 가져옵니다.
2. 경로가 Geyser 설정 폴더 내부인지 확인합니다.
3. 원본 설정의 타임스탬프 백업을 만듭니다.
4. `java.auth-type` 또는 구형 `remote.auth-type` 값만 `floodgate`로 변경합니다.
5. 나머지 설정과 주석은 그대로 보존합니다.

설정 변경 후에는 서버를 완전히 재시작해야 합니다. Floodgate 키는 로그나
명령 출력에 노출하지 않습니다.

## 설치

1. 서버를 Java 26으로 실행하는 것을 권장합니다. Paper/Folia의 공식 최소·기준
   버전은 Java 25이며 BedrockShield는 Java 25 바이트코드로 빌드됩니다.
2. 최신 Geyser-Spigot, Floodgate-Spigot과 ViaVersion 5.12.2 이상을
   `plugins` 폴더에 설치합니다.
3. `BedrockShield-1.1.0.jar`를 `plugins` 폴더에 넣습니다.
4. 서버를 완전히 재시작합니다.
5. 콘솔 또는 게임에서 `/bshield doctor`를 실행합니다.
6. 인증 방식 오류가 표시되면 `/bshield repairauth`를 실행하고 다시
   완전히 재시작합니다.

Geyser 포트는 서버의 Java 포트와 다른 포트를 사용해야 하며 TCP와 UDP가
모두 열려 있어야 합니다.

## 명령어

모든 관리 명령은 기본적으로 OP에게만 부여되는 `bedrockshield.admin`
권한이 필요합니다.

| 명령어 | 설명 |
|---|---|
| `/bshield status` | 연동 상태와 복구 횟수를 표시합니다. |
| `/bshield doctor` | Geyser, Floodgate, ViaVersion, 키 및 충돌 플러그인을 검사합니다. |
| `/bshield errors` | 최근 Bedrock 접속 거절·킥·연결 종료 사유를 표시합니다. |
| `/bshield repairauth` | Geyser 설정을 백업하고 Floodgate 인증 방식으로 교정합니다. |
| `/bshield fix [player]` | 대상 플레이어의 위치와 인벤토리를 수동 복구합니다. |
| `/bshield reload` | BedrockShield의 `config.yml`을 다시 읽습니다. |

`/bedrockshield`를 `/bshield` 대신 사용할 수도 있습니다.

## Terracotta/Xbox 오류 확인

1. Bedrock 휴대폰에서 서버 접속을 한 번 시도합니다.
2. 서버에서 `/bshield errors`를 실행합니다.
3. 기록이 있으면 표시된 해결 힌트와 같은 시각의 서버 로그를 확인합니다.
4. 기록이 전혀 없다면 접속 요청이 Geyser까지 도달하지 않은 것입니다.
   이 경우 Xbox 로그인, 휴대폰 네트워크, 방화벽 또는 Geyser TCP/UDP 포트를
   확인해야 합니다.

`Terracotta`는 Minecraft에서 원인이 특정되지 않은 Bedrock 오류로 분류됩니다.
BedrockShield는 서버에 도달한 연결의 실제 종료 사유를 수집하지만 Xbox 인증을
비활성화하지는 않습니다.

## 설정

첫 실행 후 `plugins/BedrockShield/config.yml`에서 지연 시간과 각 복구 기능을
조정할 수 있습니다.

```yaml
stabilization:
  join-delay-ticks: 10
  respawn-delay-ticks: 5
  teleport-delay-ticks: 4
  inventory-resync: true

safe-position:
  enabled: true
  vertical-search-blocks: 4
  rescue-below-world: true
  below-world-margin: 8

desync-repair:
  cancelled-inventory-actions: true
  cancelled-block-actions: true
  delay-ticks: 2
  cooldown-millis: 150

connection-diagnostics:
  log-failures: true
```

## 빌드

JDK 25가 설치된 환경에서 다음 명령을 실행합니다.

```bash
./gradlew clean build
```

Windows:

```powershell
.\gradlew.bat clean build
```

결과 JAR은 `build/libs/BedrockShield-1.1.0.jar`에 생성됩니다.

## Folia 안전성

- 플레이어 관련 작업은 `EntityScheduler`에서 실행됩니다.
- 다른 리전으로 이동할 수 있는 위치 복구는 `teleportAsync`를 사용합니다.
- 접속 기록과 쿨다운은 동시성 안전 컬렉션으로 관리합니다.
- Bukkit 메인 스레드 스케줄러에 의존하지 않습니다.

## 제한 사항

- Geyser에 도달하지 않은 Xbox/Microsoft 로그인 실패는 자동 복구할 수 없습니다.
- 방화벽이나 공유기 포트를 플러그인이 직접 열 수 없습니다.
- 프록시 구성에서는 모든 Floodgate 인스턴스의 키가 같고
  `send-floodgate-data`가 활성화되어야 합니다.
- 로그인 플러그인 충돌은 감지할 수 있지만 해당 플러그인을 자동으로
  비활성화하지 않습니다.

## English summary

BedrockShield is a Folia-safe compatibility plugin for Geyser and Floodgate. It
mitigates post-login position/inventory desync, repairs ghost blocks after
cancelled actions, records Geyser disconnect reasons, verifies the Floodgate
runtime configuration, and can safely back up and repair Geyser's `auth-type`.
It never disables Xbox authentication.

