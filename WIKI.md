# Releases Board WIKI

## 📌 프로젝트 개요 (Project Overview)
**Releases Board**는 소프트웨어 버전 배포(Release) 일정을 관리하고, 각 릴리스의 진행 상태 및 히스토리를 한눈에 파악할 수 있도록 돕는 중앙 집중형 대시보드입니다. 여러 환경(Dev, Staging, Prod 등)과 제품에 대한 배포 상태를 투명하게 관리하여 팀 간 커뮤니케이션을 원활하게 합니다.

## 🎯 주요 기능 (Core Features)
1. **대시보드 (Dashboard)**
   - 현재 진행 중인 배포 목록 및 상태 한눈에 파악
   - 주간/월간 배포 일정표 (Calendar & Kanban View)
2. **릴리스 관리 (Release Management)**
   - 새로운 릴리스(배포 티켓) 생성, 수정, 삭제
   - 버전(Version), 배포 목록, 담당자, 환경, 배포일 설정 및 관리
   - 상태 관리 (대기 중, 테스트 중, 진행 중, 완료, 롤백 등)
3. **히스토리 및 추적 (History & Tracking)**
   - 과거 배포 이력 및 성공/실패 여부 기록
   - 변경 사항에 대한 릴리스 노트(Release Notes) 자동화 및 관리
   - 연관 이슈 트래커(Jira, GitHub Issues 등) 연동 지원
4. **알림 및 권한 (Notifications & Auth)**
   - 배포 상태 변경 시 메신저(Slack, Teams 등) 알림 연동
   - 사용자 역할(Admin, Developer, Viewer 등)에 따른 권한 제어

## 🔄 릴리스 워크플로우 (Release Workflows)

프로젝트 통합 및 파이프라인 흐름은 다음과 같습니다.
- `IMPORT1` & `IMPORT2` ➞ **`PATCH` (통합)** ➞ `MAIN`
- `IMPORT3` ~~& `IMPORT4`~~ ➞ `SETUP` ➞ **`setup` (통합)** ➞ `MAIN`

**[진행 상황 특징]**
- 각 배포 단계별로 **TAG**가 부여되어 관리됩니다.
- 🚫 **`IMPORT4` ➞ `SETUP` 경로 중단**: `IMPORT4`에서 `SETUP`으로 이어지던 흐름은 통합이 완료/변경되어 더 이상 사용되지 않습니다. **(완료 / 회색 표시)**
- 🔄 **그 외 파이프라인**(`IMPORT1`, `IMPORT2`, `IMPORT3` 경로)은 현재 **동시에 (병렬로) 계속 진행 중**입니다. 
- 🔗 **PATCH / SETUP 브랜치 통합**: `IMPORT1`과 `IMPORT2`의 수정 사항은 `PATCH` 브랜치로 함께 관리되며, `IMPORT3`의 흐름은 `SETUP` 및 `setup` 브랜치로 통합 관리됩니다.
- **새로운 릴리스 흐름**: `MAIN` 브랜치를 바탕으로 향후 기타 흐름들이 계속 통합될 예정입니다.

### 📊 브랜치별 TAG 적용 현황 (DAG 표현)

```mermaid
graph LR
    %% Branch/Node Definition & Styling Defaults
    classDef importNode fill:#e1f5fe,stroke:#0288d1,stroke-width:2px,color:#01579b,rx:10,ry:10;
    classDef patchNode fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#e65100,rx:5,ry:5;
    classDef setupNode fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#1b5e20,rx:5,ry:5;
    classDef mainNode fill:#fce4ec,stroke:#c2185b,stroke-width:3px,color:#880e4f,rx:15,ry:15;
    classDef dimmed fill:#f8f9fa,stroke:#ced4da,stroke-width:1px,color:#adb5bd,stroke-dasharray: 4 4,rx:10,ry:10;

    subgraph "IMPORT Source Stream"
        I1["🏗️ IMPORT1<br/>(Base: v0.9.0)<br/>🏷️ TAG: v1.0.0-imp1"]:::importNode
        I2["🏗️ IMPORT2<br/>(Base: v0.9.0)<br/>🏷️ TAG: v1.0.0-imp2"]:::importNode
        I3["🏗️ IMPORT3<br/>(Base: v0.9.0)<br/>🏷️ TAG: v1.0.0-imp3"]:::importNode
        I4["🚫 IMPORT4 (사용 안 함)<br/>(Base: v0.9.0)<br/>🏷️ TAG: v1.0.0-imp4"]:::dimmed
    end

    subgraph "Integration Streams"
        P["🔥 PATCH (Merged)<br/>(Base: v1.0.0-imp1/2)<br/>🏷️ TAG: v1.0.1-patch"]:::patchNode
        
        S1["⚙️ SETUP (Merged)<br/>(Base: v1.0.0-imp3)<br/>🏷️ TAG: v1.0.0-setup"]:::setupNode
        S2["⚙️ setup (Merged)<br/>(Base: v1.0.0-setup)<br/>🏷️ TAG: v1.0.1-setup"]:::setupNode
    end

    subgraph "Target Stream"
        M["🚀 MAIN<br/>(Base: MERGED)<br/>🏷️ TAG: v1.1.0-RC"]:::mainNode
    end

    %% Flow Definitions
    I1 ==>|Merge| P
    I2 ==>|Merge| P
    P ==> M
    
    I3 ==>|Merge| S1
    I4 -.->|사용 안 함| S1
    S1 --> S2
    S2 ==> M
```

*참고: 위 TAG 명칭(v1.0.0 등)은 예시입니다. 실제 프로젝트의 태그 버전으로 관리됩니다.*

```
