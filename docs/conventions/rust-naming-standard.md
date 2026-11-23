# Rust 이름 짓기 표준

이 문서는 주인(kayden.kim)의 모든 Rust 프로젝트에 공통으로 적용하는 이름 짓기 표준이다. 대상은 타입, trait, 모듈, 파일, crate 이름이다. 핵심 원칙은 이름에 역할을 나타내는 방법을 모듈 경로, suffix, 둘 다 가운데 역할마다 하나로 고정하는 것이다. 도메인 개념은 경로가 맥락을 주고 suffix를 붙이지 않는다. 계층 경계를 넘는 타입은 이름이 개념과 역할을 스스로 말한다. 두 저장소(narumq-v2, vnc-relay)의 현황 비교는 부록 B, 충돌 목록은 부록 C, 결정 기록은 부록 D에 있다.

| 항목 | 값 |
| --- | --- |
| 상태 | Active (2026-11-24 주인 채택. 두 저장소에는 각 저장소의 컨벤션에 반영한 뒤 적용한다) |
| 적용 대상 | Rust, `**/*.rs`, `Cargo.toml`의 타입·trait·모듈·파일·crate 이름 (주인의 모든 Rust 프로젝트) |
| 기준 버전 | [Rust API Guidelines: Naming](https://rust-lang.github.io/api-guidelines/naming.html), [Clippy 설정 문서](https://raw.githubusercontent.com/rust-lang/rust-clippy/master/book/src/lint_configuration.md)(master, 조사일 2026-11-24), Rust edition 2024(두 저장소 기준) |
| 관련 아키텍처 문서 | 없음. 조사한 두 저장소의 코드 지도는 narumq-v2 `docs/architectures/project-structure.md`, vnc-relay `docs/architectures/code-map.md`다 |
| 관련 ADR | 없음 |
| 적용 시작일 | 2026-11-24 |
| 규칙 준수 확인 방법 | 코드 리뷰. 자동 검증은 아직 없다([5절](#5-검증-명령과-통과-기준)) |

## 1. 규칙 우선순위

[컨벤션 작성 규칙](AGENTS.md#규칙-우선순위)을 따른다. 이 표준은 4순위(프로젝트 컨벤션)에 속한다. 컴파일러, rustfmt, 공식 관례(Rust API Guidelines)가 정한 것은 바꾸지 않고, 그들이 선택지를 남긴 부분(역할 suffix, 모듈 경로 반복, crate 이름)만 정한다. 각 프로젝트의 컨벤션은 이 표준을 링크하고 프로젝트 고유 항목만 더한다([부록 E](#부록-e-문서-위치-판단과-하지-않은-일)).

### 1.1 Rust 관행과 근거

원문을 이번 조사에서 읽은 항목과 읽지 않은 항목을 구분한다.

| 근거 | 내용 | 이 표준에서 쓰는 곳 | 확인 |
| --- | --- | --- | --- |
| API Guidelines C-CASE | 모듈은 `snake_case`, 타입·trait은 `UpperCamelCase`. 약어는 한 단어(`Uuid`). `snake_case`에서 단어는 `_`로 나눈다(`btree_map`). crate 이름은 기준이 불분명하다. `-rs`·`-rust`를 붙이지 않는다 | 2.5절 | 원문 확인 |
| API Guidelines C-CONV, C-GETTER | `as_`(비용 없음, 빌림→빌림) `to_`(비용 있음) `into_`(소유권 이동). getter에 `get_`를 쓰지 않는다. 예외는 단일하고 뚜렷한 대상이 있는 `Cell::get`류 | 2.6절, X5·X6 | 원문 확인 |
| API Guidelines C-ITER, C-ITER-TY | 이터레이터 타입은 만드는 메서드 이름을 따른다(`IntoIter`). 이 이름은 "소유 모듈을 앞에 붙였을 때(`vec::IntoIter`)" 가장 의미가 통한다 | 2.2절 "경로로 표현" 예외 | 원문 확인 |
| API Guidelines C-WORD-ORDER | 오류 타입은 동사-목적어-Error 순서(`ParseIntError`, `RecvTimeoutError`). 순서보다 crate 안 일관성이 중요하다 | 7절 `Error` | 원문 확인 |
| Clippy `module_name_repetitions` | 공개 모듈 안의 공개 항목 이름이 모듈 이름으로 시작하거나 끝나면 경고한다. `restriction` 그룹이라 기본으로 꺼져 있다. `io::Error`·`fmt::Error`를 모범으로 든다. 모듈 이름과 똑같은 이름은 `allow-exact-repetitions` 기본값(`true`)으로 허용한다 | 2.3절, X1 | 소스·설정 문서 확인 |
| Clippy `module_inception` | 부모와 같은 이름의 모듈을 경고한다. `style` 그룹. `allow-private-module-inception` 기본값은 `false` | 2.5절 | 소스·설정 문서 확인 |
| The Rust Book 7.4 | 함수는 부모 모듈을 import하고(`hosting::add_to_waitlist()`), struct·enum은 전체 경로로 import한다. 같은 이름 둘은 부모 모듈 import 또는 `as`로 푼다(`fmt::Result`, `io::Result`). `pub use`는 내부 구조와 사용자가 생각하는 구조를 다르게 노출할 때 쓴다 | 2.3절, X14·X15 | 원문 확인 |
| 2018 edition 모듈 파일 | `foo.rs` + `foo/`. clippy `mod_module_files`는 `mod.rs`를 막는다 | 2.5절 | 두 저장소의 규칙(`rust-coding-conventions.md:128`, `Cargo.toml:89`)으로 확인. lint 설명 원문은 읽지 않았다 |
| 표준 라이브러리 관행 | trait을 능력·agent noun으로 이름 짓는다(`Read` `Write` `Hasher` `Allocator`). 오류와 결과는 모듈 경로로 한정한다(`io::Error` `fmt::Result` `io::Result`) | 2.2절, 7절 | 일반 지식. Book 7.4의 `fmt::Result`·`io::Result` 예만 원문 확인 |
| API Guidelines C-BUILDER | 복잡한 값은 `TBuilder` 타입으로 만든다. 더 나은 이름이 있으면 그 이름을 쓴다(`std::process::Command`) | 2.2절 빌더 행 | 원문 확인 |
| RFC 445 확장 trait 관례 | 확장 trait은 `FooExt`로 이름 짓는다. `Foo`는 확장하는 타입 또는 trait이다 | 2.2절 확장 trait 행 | 원문 확인 |
| `Handle`·`Client` suffix | `std::thread::JoinHandle` 같은 관행이다. std 문서는 읽지 않았다 | 2.2절 핸들·클라이언트 행 | 관행. 원문 근거 없음 |

## 2. 규칙

### 2.1 원칙

1. 타입은 두 부류로 나눈다. **개념 타입**은 업무 개념 자체다(`Lifecycle`, `AllowList`). **역할 타입**은 계층 경계에서 일을 맡는 타입이다(`AuditService`, `AuditSink`, `TlsListener`).
2. 개념 타입은 모듈 경로가 맥락을 준다. 이름에 역할 suffix를 붙이지 않는다.
3. 역할 타입은 이름이 **개념 prefix + 역할 suffix**로 스스로 뜻을 말한다. 경로를 보지 않아도 읽혀야 한다.
4. 역할 suffix는 닫힌 어휘표(7절)에서 고른다. 새 단어는 이유를 표에 적고 추가한다. 같은 역할에 단어를 둘 쓰지 않고, 같은 단어에 역할을 둘 주지 않는다.
5. 역할을 경로와 suffix에 두 번 쓰지 않는다. 모듈 이름이 역할 단어(`repository` `service` `dto`)이면 타입에 같은 suffix를 붙이지 않는다.
6. 이름은 5단어 미만으로 짓는다(두 저장소의 기존 권장).

### 2.2 역할마다 이름 표현 방식 (경로 / suffix / 둘 다 중 하나)

방식 기호: **P**는 경로만, **S**는 suffix(prefix 포함)만, **PS**는 둘 다다. PS(`repository::OrderRepository`)는 어떤 역할에도 쓰지 않는다.

| 역할 | 방식 | 이름 형태 | 좋은 예 | 나쁜 예 | 근거 |
| --- | --- | --- | --- | --- | --- |
| 도메인 개념(엔티티·값·enum) | P | `<개념>` | `session::Lifecycle`, `AllowList` | `SessionEntity`, `LifecycleModel` | 표준 라이브러리(`Vec` `String`)는 역할 suffix를 붙이지 않는다 |
| 값 모양 단어 | P(개념 이름의 일부) | `<개념>Id` `Key` `Kind` `State` `Policy` `Table` `List` | `TenantId`, `SessionState`, `TargetPolicy` | `TenantIdentifier` | 두 저장소가 이미 같게 쓴다 |
| 이터레이터형 보조 타입 | P | `Iter` `IntoIter` | `sessions::Iter` | `SessionsIterator` | C-ITER-TY |
| 오류 | S | `<개념>Error` | `TlsError`, `SessionError` | 단독 `Error`, `TlsErr` | C-WORD-ORDER, 표준 `Error` trait과 겹침 방지(X7) |
| 유스케이스 입구 | S | `<개념>Service` | `AuditService` | `Service`, `AuditUseCase`, `AuditManager` | 두 저장소 공통 어휘 |
| 포트 trait | S | `<개념><포트 역할>` | `AuditSink`, `AccountStore`, `TokenSource`, `TargetGateway` | `AuditPort`, `IAuditSink`, `AuditPublisherTrait` | 직접 import하고 같은 스코프에서 함께 쓴다(2.3절 조건 2·4) |
| 어댑터 구현체 | S | `<기술><포트 이름>` | `FileAuditSink`, `TcpTargetGateway`, `SystemClock` | `AuditSinkImpl`, `DefaultAuditSink` | trait과 같은 이름 방지(조건 3) |
| 생성 옵션 | S | `<대상>Config` | `TlsListenerConfig` | `Config`(공개 단독 이름) | 조건 1·2 |
| 검증을 마친 설정 모델 | S | `Settings`, `<부분>Settings` | `RelaySettings` | `RelaySettingsDto` | vnc-relay 현황. 부록 D의 D6 |
| 경계 입력 | S | `<동작><개념>Command`, `<개념>Query` | `RecordAuditCommand`, `AuditRecordQuery` | `RecordCommandDto` | 부록 D의 D7 |
| 경계 출력(읽기 전용) | S | `<개념>View` | `SessionView` | `SessionDto` | 두 저장소 공통 어휘 |
| 어댑터가 소유한 요청·응답 | S | `<개념>Request` `<개념>Response` | `LogsResponse` | `LogsDto` | vnc-relay 현황 |
| 어댑터 내부 저장·전송 형식 | S | `<개념>Wire` | `AuditEventWire` | 공개 `Wire` | narumq-v2 현황. 어댑터 밖으로 내보내지 않는다 |
| 사건 | S | `<개념>Event` | `SourceEvent` | `SourceMessage` | vnc-relay 현황 |
| 시점 고정 상태 | S | `<개념>Snapshot` | `RelaySnapshot` | `RelayStateCopy` | vnc-relay 현황 |
| 빌더 | S | `<대상>Builder`. 더 나은 이름이 있으면 그 이름 | `ServerBuilder` | `ServerFactory` | C-BUILDER |
| 확장 trait | S | `<대상>Ext` | `StreamExt` | `StreamHelper` | RFC 445 |
| 핸들·클라이언트 | S | `<대상>Handle`, `<프로토콜>Client` | `JoinHandle`, `JwksClient` | `JwksManager` | 관행. 원문 근거 없음 |
| 시험 대역 | prefix | `Fake` `Stub` `Spy` + 포트 이름 | `FakeClock` | `MockClockImpl` | 두 저장소 공통 |
| 모듈 안 비공개 보조 타입 | P | 짧은 일반 이름 | `State`, `Shared`, `Inner` | `AuditPublisherInnerState` | 2.3절 B 조건 |

### 2.3 suffix(역할을 담은 이름)가 필요한 조건

**A. 아래 중 하나라도 해당하면 S 방식을 쓴다(suffix 필수).**

1. 타입이 자기 모듈 밖으로 나간다. `pub use`로 올리거나, 다른 crate가 직접 import한다(조립 코드 포함).
2. 같은 스코프에서 다른 같은 이름의 타입과 함께 import될 수 있다. 다른 모듈의 같은 역할 타입, 표준 라이브러리 또는 외부 crate의 동명 타입이 포함된다.
   - 현황: narumq-v2의 `tls::Listener`·`websocket::Listener`·`Error` 13개, vnc-relay의 도메인 `Listener`와 어댑터 `Listener`.
3. trait과 구현체가 같은 이름이 된다. 구현체에 기술·구현 성격 prefix를 붙인다(`Clock` → `SystemClock`).
4. 계층 경계를 넘는다. 포트, 서비스, 경계 데이터, 오류는 읽는 쪽이 계층 모듈 경로를 보지 않아도 역할을 알아야 한다.

**B. 아래를 모두 만족하면 P 방식을 쓸 수 있다(suffix 없음).**

1. 업무 개념이거나, 한 모듈 안에서만 쓰는 타입이다. 비공개이거나 `pub(super)`·`pub(crate)` 보조 타입이다.
2. 같은 스코프에서 같은 이름과 함께 import될 가능성이 없다.
3. 모듈 밖으로 `pub use`하지 않는다.

**C. 경로로 표현하는 도메인 개념 타입은 모듈 이름을 되풀이하지 않는다.** 되풀이하면(`session::SessionState`) 사용 쪽이 모듈 경로를 이미 알고 있으므로 중복이다. 이 규칙은 개념 타입에만 적용한다. S 방식 역할 타입은 개념 prefix가 모듈 이름과 겹쳐도 된다(`audit::AuditService`). 이름이 모듈 경로 없이도 뜻을 가져야 하기 때문이다.

**D. lint 전제.** `clippy::module_name_repetitions`는 공개 모듈의 공개 항목만 본다. 이 표준의 S 방식은 모듈 이름과 겹치는 이름을 허용하므로 이 lint를 켜지 않는다(두 저장소도 켜지 않았다). 이름 중복 검사는 코드 리뷰로 한다. lint 설정 변경은 이 문서의 범위가 아니다.

### 2.4 계층별 규칙

계층은 클린 아키텍처의 고리를 crate·모듈로 옮긴 것이다.

| 클린 아키텍처 고리 | 코드 위치 | 타입 이름 | trait 이름 | 모듈·파일 이름 | 금지 |
| --- | --- | --- | --- | --- | --- |
| domain(엔티티·값) | crate `<제품>-domain` | 개념 이름(P). 값 모양 단어 허용. 오류는 `<개념>Error` | 없는 것이 기본. 규칙 확장점이 필요하면 능력 이름 | 개념 단위 `snake_case`, `pub mod`, 파일 이름 = 모듈 이름 | `Service` `Manager` `Dto` `View` `Impl` |
| application(유스케이스) | crate `<제품>-application` | `<개념>Service`, `Command` `Query` `View` `Snapshot` `Event` `Settings` `Config` | 포트 trait은 `ports` 모듈 | 기능 단위 `snake_case` | `UseCase` `Interactor` `Handler` `Manager` `Impl` |
| ports(application이 소유하는 trait) | application의 `ports` 모듈. 하위 파일은 개념 단위, 이름은 `ports::` 아래로 평면 노출을 권장 | 포트가 소유한 데이터는 `Command` `Query` `View` `Event` | `<개념><Source·Sink·Store·Gateway·Control·Clock 등>` | `ports.rs` + `ports/<개념>.rs` | `Port` suffix, `I` prefix, 기술 이름 |
| adapters(구동되는 쪽. 포트 구현) | crate `<제품>-adapters` 또는 `<제품>-adapter-<기술>` | `<기술><포트 이름>`. 내부 형식은 `Wire` `Request` `Response` | 포트 trait은 만들지 않는다(application 소유) | 기술 이름 또는 `<개념>_<기술>`. HTTP 경로 묶음은 `<개념>_api` | `Impl` `Default<포트>` |
| interface(구동하는 쪽: 진입점·리스너·CLI) | adapters 안의 모듈. 실행 인자는 조립 crate | `<기술>EntryPoint`, `<이름>Listener`. HTTP 핸들러는 타입이 아닌 함수 | 필요 시 `Listener` | `<개념>_api`, `<기술>_http` | `Controller` `Handler` `Presenter` suffix |
| infrastructure(런타임·OS 연동) | 별도 crate를 만들지 않고 adapters에 둔다. 기술 이름이 이미 맥락이다 | `<기술>…`(`Tokio` `System` `Fs` `Rustls` `Mmdb`) | 해당 없음 | 기술 이름 | `Infrastructure` suffix |
| composition root(조립) | 실행 crate `<제품>` | 새 역할 타입을 만들지 않는다. CLI 타입은 `Cli` `Args` 등 clap 관례 | 해당 없음 | `main.rs` + 조립 파일 | 조립 코드에 업무 규칙 |

### 2.5 모듈·파일·crate 이름

**모듈과 파일**

- `snake_case`이고 단어는 `_`로 나눈다(`runtime_metrics`). 붙여 쓰지 않는다(C-CASE의 `btree_map` 예). 개념 명사 하나를 기본으로 한다.
- 파일은 `foo.rs` + `foo/`다. `mod.rs`와 `#[path]`는 쓰지 않는다. 시험 코드도 같다.
- 한 파일의 주 타입은 파일 이름의 CamelCase와 같게 한다(`target_policy.rs` → `TargetPolicy`). 모듈 이름과 똑같은 이름은 clippy 기본 설정에서 허용된다.
- 모듈 이름에 역할 단어를 쓰는 것은 `ports`와 어댑터 crate뿐이다. `utils` `common` `helpers` `models` `interfaces` `impl` `types`는 쓰지 않는다.
- 부모와 같은 이름의 하위 모듈을 만들지 않는다(`module_inception`).
- 어댑터 모듈 이름은 기술 이름 또는 `<개념>_<기술>`이다(`tls`, `accounts_file`, `management_http`).

**crate**

| 종류 | 이름 | 예 |
| --- | --- | --- |
| 도메인 | `<제품>-domain` | `narumq-domain`, `vnc-relay-domain` |
| 애플리케이션 | `<제품>-application` | `narumq-application`, `vnc-relay-application`(D9) |
| 어댑터 하나로 묶음 | `<제품>-adapters` | `vnc-relay-adapters` |
| 어댑터를 기술별로 나눔 | `<제품>-adapter-<기술>` | `narumq-adapter-tls` |
| 실행 파일 | `<제품>` | `narumq`, `vnc-relay` |
| 보조 UI·도구 | `<제품>-<이름>`, 개발 도구는 `xtask` | `vnc-relay-tray`, `xtask` |
| 시험 하네스 | `<제품>-sim` 또는 `<제품>-testkit` | `narumq-sim` |

crate 이름은 `kebab-case`이고 `-rs`·`-rust`를 붙이지 않는다(C-CASE). 단수 `adapter-<기술>`과 복수 `adapters`를 섞어 쓰지 않는다. 한 crate에 어댑터를 모으면 `adapters`, 기술마다 crate를 나누면 `adapter-<기술>`이다.

### 2.6 함수 이름

- getter는 `get_`를 쓰지 않는다(C-GETTER). `get`·`get_mut`처럼 컬렉션의 단일 대상 접근자는 예외다.
- 변환은 C-CONV를 따른다. `as_`(비용 없음) `to_`(비용 있음) `into_`(소유권 이동).
- 경계 변환은 `From`·`TryFrom` 구현을 먼저 고려한다. 도메인 값은 `new`와 `parse`, `from_<출처>`로 만든다.
- 표본 점검: 두 저장소의 `as_` 이름은 `as_str` `as_bytes`(narumq-v2)와 `as_str` `as_bytes` `as_i64` `as_name`(vnc-relay)뿐이다. vnc-relay의 `into_` 6개(`into_inner` `into_parts` 등)와 `to_` 1개(`to_json_string`, `adapters/src/ops_log.rs:343`)도 규칙과 맞는다. `get_`은 `get_ref`(`application/src/ports.rs:36`), `get_mut`(`domain/src/admin_session.rs:221`), HTTP 경로 함수 `get_session`(`adapters/src/management_http/session_api.rs:112`)이다. 위반 의심은 없다.

### 2.7 Rust 관행과 클린 아키텍처 suffix가 부딪히는 지점

각 행의 ID(X1~X16)는 다른 절에서 가리키는 이름이다.

| # | 부딪히는 지점 | Rust 관행 | 클린 아키텍처 관행 | 규칙 |
| --- | --- | --- | --- | --- |
| X1 | 모듈 경로 중복 `repository::OrderRepository` | `io::Error` 식으로 경로가 맥락을 준다. `module_name_repetitions`(restriction)가 반복을 경고한다 | 역할 suffix로 이름만 보고 역할을 안다 | 역할 타입은 S(suffix 포함, 경로 반복 허용), 개념 타입은 P. 역할 단어를 경로와 suffix에 둘 다 쓰지 않는다. 이 lint는 켜지 않는다 |
| X2 | trait에 `Port`·`I` 붙이기 | trait은 능력 이름(`Read` `Write`). `I` 접두는 관행이 아니다 | `IRepository`, `OrderPort` | 금지. 역할 어휘로 끝낸다(`AuditSink`). 두 저장소가 모두 이미 `I`·`RepositoryTrait` 표식을 금지한다 |
| X3 | `Impl` suffix | 구현체 이름은 기술·성격을 말한다(`BufReader`, `File`) | `OrderServiceImpl` | 금지. `<기술><포트 이름>`을 쓴다. 기술 이름이 없으면 포트를 나누는 이유를 다시 본다 |
| X4 | `Service` 남용 | 해당 관행 없음 | `Service`, `Manager`, `Handler`로 모든 로직을 모음 | `Service`는 유스케이스 입구에만 쓴다. 규칙은 domain 타입의 메서드, 판정 규칙은 `Policy`. 한 `Service` 안의 메서드가 사용 사례 하나다. `Manager` `Helper` `Util` `Handler`는 금지(vnc-relay 금지 목록과 같다) |
| X5 | `get_` 접두사 | C-GETTER: 쓰지 않는다 | getter 관행 | 쓰지 않는다. 포트 메서드도 같다(`source.value()`, `store.load()`) |
| X6 | 변환 `as_` `to_` `into_` `from_` | C-CONV 비용·소유권 규칙 | DTO↔도메인 변환 함수 | 경계 변환은 `From`·`TryFrom`. 이름을 붙일 때는 C-CONV. `to_view()`처럼 비용이 있으면 `to_` |
| X7 | 오류 타입 이름 | `io::Error`처럼 단독 이름은 경로로 한정한다. `ParseIntError`처럼 `<동사목적어>Error`도 많다(C-WORD-ORDER). `std::error::Error`와 겹치면 별칭이 필요하다 | 계층마다 오류가 여러 개 섞인다 | 항상 `<개념>Error`(D5). 표준 trait은 `impl std::error::Error for TlsError`로 쓰면 별칭이 필요 없다 |
| X8 | `Result` suffix | `fmt::Result` `io::Result`는 type alias다 | 결과 묶음 이름 | alias가 아닌 타입에는 `Result` suffix를 쓰지 않는다. 분류에는 `Outcome`, 묶음에는 개념 이름(`VerificationReport`)을 쓴다(D10) |
| X9 | `Config`와 `Settings` | 생태계는 `Config`를 많이 쓴다 | 설정 모델 | `Config`는 생성 옵션, `Settings`는 검증을 마친 설정 모델이다. 한 단어로 합치지 않는다(D6) |
| X10 | `Dto` | 해당 관행 없음 | DTO | 역할이 방향별로 갈리므로 `Command` `Query` `View` `Request` `Response`로 나눈다(D7) |
| X11 | 같은 이름 충돌(`Listener`) | Book 7.4: 부모 모듈 import 또는 `as` | 계층마다 같은 개념 | 별칭보다 이름을 구분한다. 예: 도메인 enum `Listener`는 `ListenerKind`처럼 바꾸고 어댑터 trait은 `Listener`를 유지한다 |
| X12 | 빌더와 `Default` | `new`, 빌더, `Default` | 생성 옵션 | 필수 값이 적으면 `new(config)`. 선택 값이 많으면 `<대상>Builder`. 구현이 하나뿐인데 `Default<포트>`를 만들지 않는다(narumq-v2 L45와 같다) |
| X13 | `Wire` 같은 어댑터 내부 형식 | 비공개로 둔다 | adapter 경계에서 외부 형식을 변환(narumq-v2 `AGENTS.md:23`) | `Wire`는 어댑터 밖으로 내보내지 않는다. 공개하면 `View`·`Response`로 바꾼다 |
| X14 | re-export와 prelude | Book 7.4: `pub use`로 사용자가 생각하는 구조를 노출한다 | 계층 crate 경계 | 어댑터 crate는 평면 `pub use`를 허용한다(narumq-v2 현황). `prelude` 모듈은 만들지 않는다(두 저장소 모두 0개) |
| X15 | `use` 스타일 | 타입은 전체 경로 import, 함수는 부모 모듈 import | 계층을 넘는 import가 많다 | 타입은 직접 import한다. 개념 타입의 동명 충돌(`tenancy::Error` 식)만 부모 모듈 import로 푼다. 함수는 부모 모듈을 import한다(`use crate::toml_text;` → `toml_text::error_position(…)`). 현재 vnc-relay에는 함수를 직접 import하는 곳이 있다(`adapters/src/accounts_file.rs:20`, `adapters/src/admission.rs:20`) |
| X16 | 단어 수 | 해당 관행 없음 | 긴 이름 | 5단어 미만을 유지한다. 개념 prefix + 역할 suffix가 4단어를 넘으면 개념 이름을 다시 정한다. 약어로 자르지 않는다(C-CASE: 약어는 한 단어). 예: `ClientCertificateAuthenticationProvider`(`application/src/ports/authentication.rs:96`)는 4단어이므로 허용 범위의 끝이다 |

## 3. 예제

```text
좋음: application::ports::AuditSink 와 adapters::FileAuditSink, application::AuditService
나쁨: application::ports::audit::Port, FileAuditSinkImpl, audit::AuditUseCaseManager
이유: 포트·구현체·유스케이스는 이름이 개념 + 역할로 끝난다. Port·Impl·Manager 같은 역할 없는 단어는 쓰지 않고, 같은 trait과 구현체의 이름이 겹치지 않는다.
```

```text
좋음: domain::session::Lifecycle, domain::allow_list::AllowList
나쁨: domain::session::SessionLifecycleEntity, domain::models::AllowListModel
이유: 도메인 개념은 모듈 경로가 맥락을 준다. 역할 suffix를 붙이지 않고 models 같은 역할 없는 모듈을 만들지 않는다.
```

```text
좋음: TlsError, SessionError (crate 밖에서도 직접 import)
나쁨: 공개 단독 Error 13개, use std::error::Error as StdError
이유: 동명 충돌(조건 2)과 표준 Error trait 별칭을 없앤다.
```

아래 비교는 사용하는 쪽 코드가 어떻게 읽히는지 보여 주는 예시다. 시그니처는 실제 코드가 아니다.

### 3.1 포트 trait과 구현체 (감사 발행)

```rust
// A. 경로로만 표현 (P)
use narumq_application::ports::{audit, clock};
use narumq_adapter_auditlog as auditlog;

struct Service<P: audit::Publisher, C: clock::Clock> { publisher: P, clock: C }
let publisher: Arc<dyn audit::Publisher> = Arc::new(auditlog::Publisher::new(/* … */));
```

```rust
// B. suffix로만 표현 (S) — 표준
use narumq_application::ports::{AuditSink, Clock};
use narumq_adapter_auditlog::FileAuditSink;

struct AuditService<P: AuditSink, C: Clock> { publisher: P, clock: C }
let publisher: Arc<dyn AuditSink> = Arc::new(FileAuditSink::new(/* … */));
```

```rust
// C. 경로와 suffix 모두 (PS) — narumq-v2 현재 포트 경로
use narumq_application::ports::audit::AuditPublisher; // adapters/auditlog/src/publisher.rs:10
use narumq_application::ports::clock::Clock;
```

| 기준 | A. 경로만 | B. suffix만 | C. 둘 다 |
| --- | --- | --- | --- |
| 이름 충돌 | 직접 import하면 `Publisher`가 `audit`·`metrics`에서 겹친다. 모듈 import로 피하지만 조립 코드마다 `audit::` `metrics::`를 쓴다 | 이름이 서로 달라 직접 import해도 충돌이 없다 | B와 같다. 경로가 길다 |
| grep | `grep Publisher`가 모든 포트를 찾지만 어느 개념인지 모른다. `audit::Publisher`로 찾아야 한다 | `grep AuditSink`가 개념과 역할을 한 번에 찾는다. `grep Sink`로 역할 전체도 찾는다 | B와 같다 |
| IDE 자동 import | 후보 이름이 겹쳐 경로로 고른다 [추정, 미검증] | 후보가 유일하다 [추정, 미검증] | 후보가 유일하다 |
| 프로젝트 간 일관성 | 프로젝트마다 모듈 구성이 다르면 경로가 달라진다 | 이름이 모듈 구성과 무관하게 같다 | B와 같다. 경로·이름 중복으로 줄이 길다 |
| Rust 관행과 거리 | 가장 가깝다(`io::Error`) | 중간. 표준 라이브러리는 능력 이름(`Read`)을 쓴다 | `module_name_repetitions`가 지적한다 |

### 3.2 오류 타입

```rust
// A. 단독 이름 + 경로 — narumq-v2 domain/src/security.rs:12, :435 와 같은 형태
use crate::tenancy::{self, SessionKey};
fn relation(r: Result<bool, tenancy::Error>) -> Result<bool, Error> { /* … */ }
impl std::error::Error for Error {}        // std Error를 import하지 않으면 별칭이 필요 없다
// 현재 코드는 별칭을 쓴다: adapters/auditlog/src/error.rs:1  use std::error::Error as StdError;
```

```rust
// B. <개념>Error — vnc-relay adapters/src/accounts_file.rs:13 와 같은 형태 (표준)
use vnc_relay_app::accounts::{AccountsError, LoadedAccounts};
fn load() -> Result<LoadedAccounts, AccountsError> { /* … */ }
```

| 기준 | A. 단독 `Error` | B. `<개념>Error` |
| --- | --- | --- |
| 이름 충돌 | `std::error::Error`, 다른 모듈의 `Error`와 겹친다(narumq-v2에 13개) | 겹치지 않는다 |
| grep | `Error`가 너무 많아 개념별로 찾기 어렵다 | `grep TlsError`로 바로 찾는다 |
| IDE 자동 import | 같은 이름이 13개라 후보를 고른다 [추정, 미검증] | 후보가 유일하다 [추정, 미검증] |
| 일관성 | crate마다 다른 경로로 쓴다 | crate와 무관하게 같다 |
| Rust 관행 | `io::Error`·`fmt::Error`와 같다 | `ParseIntError`·`RecvTimeoutError`와 같다(C-WORD-ORDER) |

### 3.3 도메인 개념 타입

```rust
// P — 표준 (개념 타입)
use narumq_domain::session;
let lifecycle = session::Lifecycle::open(/* … */);

// PS — 같은 개념을 역할 suffix로 (나쁜 예)
use narumq_domain::session::SessionLifecycleEntity;
```

도메인 개념은 경로가 이미 맥락을 주므로 P가 읽기 쉽고 경고도 없다. 같은 개념을 여러 계층이 부르면(예: 서비스·포트·어댑터) 그때 역할 suffix가 필요하다.

### 3.4 계층 우선과 기능 우선 배치, suffix 필요성

| 항목 | 계층 우선(`domain/`, `application/`, `adapters/` 아래에 기능) | 기능 우선(`order/`, `session/` 아래에 계층) |
| --- | --- | --- |
| 경로 모양 | `application::ports::AuditSink`, `adapters::FileAuditSink` | `audit::ports::Sink`, `audit::adapters::File` |
| suffix 필요성 | 높다. 같은 개념이 여러 계층에 나타나서 이름만으로 계층·역할을 구분해야 한다 | 낮다. 경로가 개념과 계층을 이미 말한다. 단 다른 기능의 같은 이름(`audit::ports::Sink`, `metrics::ports::Sink`)을 함께 import하면 조건 2로 suffix가 필요하다 |
| 의존 방향 강제 | crate 경계로 강제한다. narumq-v2는 Cargo `[dependencies]`가 컴파일 때 막고(ADR-0016 결정 4), vnc-relay는 추가로 `cargo xtask layers`가 허용 의존을 검사한다 | 한 crate 안이면 모듈 규칙을 도구로 검사해야 한다 |
| 두 저장소 현재 | 둘 다 crate 경계가 계층이다. 계층 안은 기능 우선(narumq-v2) 또는 기능 파일 평면(vnc-relay) | |
| 표준 | **계층 우선(crate 경계) + 계층 안 기능 모듈**. suffix 필요성이 높은 쪽이므로 역할 타입에 S 방식을 쓴다 | |

## 4. 예외와 자동 생성 산출물

- 외부 계약이 정한 이름은 계약을 따른다. 예: MQTT 패킷 이름(`Publish`, `PubAck`. `domain/src/mqtt/publish.rs:9`, `domain/src/mqtt/ack.rs:13`), 계약의 필드·오류 코드 이름. 이 이름을 담는 Rust 필드는 계약 이름과 같게 쓴다.
- 문서에 이유와 함께 예외로 등록한 이름은 suffix 규칙보다 우선한다. 예: vnc-relay `RelayStream`(`app/src/ports.rs:167`. 예외 등록은 vnc-relay `rust.md:25`).
- 개발 도구(`xtask`)와 시험 하네스(`narumq-sim`)에는 계층별 suffix 제한을 적용하지 않는다. 대소문자·모듈·파일·crate 규칙과 금지 이름은 적용한다. 도구 API의 역할명(예: AST `Visitor`)과 일반 자료 이름(`State` `Kind` `Record` `Entry`)은 문맥에 맞으면 유지할 수 있다. 두 저장소 문서가 비슷한 예외를 이미 둔다(narumq-v2 `rust-coding-conventions.md:94`, vnc-relay `rust.md:94`).
- `#[cfg(test)]` 모듈과 `tests/`에만 있는 타입은 suffix 규칙을 적용하지 않는다. 시험 대역은 `Fake` `Stub` `Spy` 중 하나로 시작한다.
- 자동 생성 Rust 코드는 두 저장소에 없다. 생기면 생성 위치와 명령을 이 절에 적는다.
- 예외는 프로젝트 컨벤션의 이름 짓기 목록에 이유와 함께 등록한다. 등록 suffix([7절](#7-이름-짓기-목록))는 D2 결정으로 닫힌 목록 밖에서 채택한 suffix다. 새 예외는 표준을 개정할 때 7절에 추가하거나 프로젝트 컨벤션에 이유와 함께 등록한다. 표준을 개정할 때 예외를 다시 검토한다.

## 5. 검증 명령과 통과 기준

이 표준에는 자동 검증 명령이 아직 없다. 리뷰어가 [2절](#2-규칙)의 표와 [7절](#7-이름-짓기-목록)의 목록으로 판정한다.

- 실행 환경: 해당 없음
- 실행 위치: 해당 없음
- 성공 조건: 새 타입·trait·모듈·crate 이름이 2절과 7절에 맞고, 예외는 4절에 따라 등록되어 있다.

이 표준은 lint 설정을 바꾸지 않는다. 이 표준은 `clippy::module_name_repetitions`를 켜지 않는 것을 전제로 한다(2.3절 D, 부록 D의 D8). 자동 검사가 필요하면 각 프로젝트가 `module_inception`과 `mod_module_files`(`mod.rs` 금지)를 켜는 안을 따로 정할 수 있다.

## 6. 기존 코드 마이그레이션

이 표준은 두 저장소의 코드를 바꾸지 않는다. 아래 수는 부록 D에서 채택한 결정(D1(c) D2(c) D3(a) D4(a) D5(a) D6(a) D7(b) D9(a) D10(a))을 두 저장소에 적용할 때 이름이 바뀌는 선언의 대략 개수다. 반영 여부와 시점은 각 저장소의 별도 작업에서 정한다.

| 저장소 | 규칙 | 바뀌는 이름(대략) | 위치 |
| --- | --- | --- | --- |
| narumq-v2 | D7 `Dto` 제거 | 22개 이름. 참조 약 324곳, 32개 파일 | `application/src/ports/*.rs`, `application/src/audit*.rs` |
| narumq-v2 | D5 단독 `Error` → `<개념>Error` | 13개(`sim` 제외) | domain 5, application 2, adapters 6 |
| narumq-v2 | D6 단독 `Config` → `<대상>Config` | 5개 | `adapters/tls`, `adapters/auditlog`, `adapters/authentication`, `adapters/runtimemetrics`, `adapters/websocket` |
| narumq-v2 | D4·조건 2 단독 `Service` | 2개 | `application/src/audit.rs:113`, `application/src/tenantlimit.rs:186` |
| narumq-v2 | 조건 2 단독 `Listener` | 2개 | `adapters/tls/src/listener.rs:17`, `adapters/websocket/src/listener.rs:89` |
| narumq-v2 | D2 `Recorder`·`Publisher`·`Reader` → `Sink`·`Source` | 10개: trait 8, 구현 2(`FileAuditPublisher`, `DataPlaneRecorder`) | `application/src/ports/audit.rs`, `ports/metrics.rs`, `application/src/tenantlimit.rs` |
| narumq-v2 | 구현체 이름 규칙 | 1개: `JwtVerifier` | `adapters/authentication/src/jwt.rs:69` |
| narumq-v2 | D10 `Result` struct | 3개 | `adapters/auditlog/src/verify.rs:10`, `domain/src/session.rs:117`, `:128` |
| narumq-v2 | 2.5절 붙여 쓴 이름 | crate 4개 + 모듈 1개: `auditlog` `diskspace` `runtimemetrics` `walframe` `tenantlimit` | `crates/adapters/*`, `application/src/tenantlimit.rs` |
| narumq-v2 합계 | | 이름 약 58개 + crate 4개 + 모듈 1개 | |
| vnc-relay | D5 `Failure` → `<개념>Error` 또는 `Reason` | 8개 | `app/src/accounts.rs:118`, `adapters/src/listener.rs:28`, `adapters/src/net.rs:203` 등 |
| vnc-relay | D10 `Result` → `Outcome`·개념 이름 | 3개 | `app/src/reload.rs:33`, `app/src/events.rs:356`, `app/src/management/config.rs:55` |
| vnc-relay | 조건 2 동명 충돌 | 2개: 도메인 `Listener`, 한쪽 `TargetHost` | `domain/src/entry_point.rs:7`, `domain/src/handshake.rs:54` |
| vnc-relay | `Control`이 포트가 아닌 타입 | 2개 | `adapters/src/request_deadline.rs:49`, `adapters/src/windows_service.rs:198` |
| vnc-relay | D9 crate `vnc-relay-app` → `vnc-relay-application` | crate 1개. import 338줄, 68개 파일 | 모든 `vnc_relay_app` 사용처 |
| vnc-relay 합계 | | 이름 약 15개 + crate 1개 | |
| vnc-relay | 문서만 고침(코드 rename 없음) | `Policy`·`Snapshot` 어댑터 사용(M6), 목록 밖 suffix 등록(`Failure` 제외 `Reason` `Guard` 등), `#[path]` 시험 5곳 | `docs/conventions/rust.md` |

- 공개 경로 변경이 더 있을 수 있다. 평면 `pub use`를 도입하면 import 경로가 바뀌지만 이름은 같다. 이 수에 넣지 않았다.
- 이 수에는 시험 코드의 참조 수정이 들어 있지 않다(narumq-v2 `Dto`만 참조 수를 적었다).
- 집계 기준은 [부록 A](#부록-a-조사-범위와-집계-기준)다.

## 7. 이름 짓기 목록

규칙은 [이름 짓기 기본 규칙](AGENTS.md#이름-짓기-기본-규칙)을 따른다. 도메인 prefix 목록은 프로젝트마다 용어집의 코드 영어 이름으로 정하므로 이 표준에서는 정하지 않는다.

suffix 목록(닫힌 목록):

| suffix | 계층 | 뜻 | 흡수·대체하는 기존 이름 |
| --- | --- | --- | --- |
| `Service` | application | 유스케이스 입구. 메서드 하나가 사용 사례 하나다 | narumq-v2 `Service` `UseCase`, vnc-relay `Service` |
| `Command` | application·ports | 상태를 바꾸는 입력 | narumq-v2 `RecordCommandDto` `QueryCommandDto` |
| `Query` | application·ports | 조회 조건 입력 | narumq-v2 `AuditRecordQueryDto`, vnc-relay `LogsFilter`(필요 시) |
| `View` | application·ports | 읽기 전용 출력 모델 | 두 저장소 공통 |
| `Snapshot` | application | 한 시점에 고정한 상태 | vnc-relay `Snapshot`, narumq-v2 `MetricsSnapshotDto` |
| `Event` | application·ports | 일어난 사건 | vnc-relay `Event` |
| `Settings` | application | 검증을 마친 설정 모델 | vnc-relay `Settings` |
| `Config` | application·adapters | 생성 시 받는 옵션 | narumq-v2 `Config` |
| `Source` | ports·adapters | 외부·환경에서 값을 읽어 온다 | 두 저장소의 `Source`, narumq-v2 `Reader` |
| `Sink` | ports·adapters | 사건·기록을 밖으로 내보낸다 | vnc-relay `Sink`, narumq-v2 `Publisher` `Recorder` |
| `Store` | ports·adapters | 메모리 밖 상태를 읽고 저장한다 | vnc-relay `Store`. `Repository` 대체 |
| `Gateway` | ports·adapters | 위 넷에 맞지 않는 외부 시스템 호출 | vnc-relay `Gateway` |
| `Control` | ports | 실행 중인 런타임에 명령을 넣는다 | vnc-relay `Control` |
| `Clock` | ports·adapters | 시각. 포트 이름 자체로도 쓴다 | 두 저장소 공통 |
| `Error` | 모든 계층 | 오류 타입. 항상 `<개념>Error` | narumq-v2 단독 `Error`, vnc-relay `Failure` |
| `Outcome` | 모든 계층 | 동작 결과를 분류하는 enum 또는 struct | narumq-v2 `Outcome`·`Result`, vnc-relay `Result`·`Outcome` |
| `Reason` | 모든 계층 | 사유를 분류하는 enum | vnc-relay `Reason`, narumq-v2 `DisconnectReason` |
| `Wire` | adapters | 어댑터 안에서만 쓰는 저장·전송 형식 | narumq-v2 `Wire` |
| `Request` `Response` | adapters | 어댑터가 소유한 API 요청·응답 모델 | vnc-relay `Response` |

등록 suffix(D2 결정으로 채택). 닫힌 목록 밖이며, 두 저장소 중 하나가 이미 쓰고 이 목록으로 대체하기 어려운 역할이다. 새 suffix는 이유와 함께 이 표에 추가한다.

| suffix | 계층 | 뜻 | 쓰는 곳 | 등록 이유 |
| --- | --- | --- | --- | --- |
| `Provider` | ports·adapters | 인증 신원을 얻어 반환한다 | narumq-v2 인증(`ports/authentication.rs:82`) | 뜻이 `Gateway`보다 정확하다 |
| `Verifier` | ports·adapters | 토큰·인증서 같은 값을 검증한다 | narumq-v2 `TokenVerifier`(`ports/authentication.rs:110`) | 검증이라는 동작이 이름에 남는다. vnc-relay 문서는 금지하지만 이 표준이 등록한다 |
| `Renderer`, `Visitor` | ports | `Renderer`는 출력 형식으로 바꾼다. `Visitor`는 순회 callback을 받는다 | narumq-v2 지표(`ports/metrics.rs:147`, `:310`) | `Sink`와 능력이 다르다 |
| `Scheduler`, `Timer` | ports·adapters | 작업을 예약하거나 대기한다 | narumq-v2 시계(`ports/clock.rs:12` `Timer`, `:21` `Scheduler`) | 예약·취소라는 능력이 `Clock`과 다르다 |

## 부록 A. 조사 범위와 집계 기준

- 읽은 문서: 두 저장소의 `AGENTS.md`, `GLOSSARY.md`, `docs/conventions/*`, `docs/architectures/*`(코드 지도), `clippy.toml`, 루트 `Cargo.toml`.
- 읽은 코드: `crates/`, `src/`, `xtask/`의 `.rs` 파일 381개.
- 집계 방법: 정규식으로 `struct`·`enum`·`trait`·`type`·`union` 선언을 뽑고, 이름의 마지막 CamelCase 단어를 역할로 봤다. 경로가 `tests/`, `tests.rs`, `test_support.rs`, `*_tests.rs`인 파일과 `#[cfg(test)] mod` 안의 선언은 뺐다. `xtask`는 역할 집계에서 뺐다.
- 한계: 매크로가 만드는 타입은 센 적이 없다. 중괄호 추적은 근사다. 숫자는 "대략"으로 읽는다. 줄 번호는 조사일의 작업 트리 기준이며, 다른 에이전트가 파일을 고치면 어긋날 수 있다.
- 경로 표기: narumq-v2의 코드 경로는 `crates/` 아래 상대 경로, vnc-relay도 `crates/` 아래 상대 경로다. 두 저장소 루트의 `src/`, `xtask/`는 따로 적는다.
- clippy는 실행하지 않았다. 부록 B.3의 `module_name_repetitions` 해당 수는 소스 규칙을 손으로 적용한 추정이다.

## 부록 B. 현황

### B.1 문서 규칙 비교

| 항목 | narumq-v2 (`docs/conventions/rust-coding-conventions.md`) | vnc-relay (`docs/conventions/rust.md`) |
| --- | --- | --- |
| suffix 목록 | §3.3 표 15행, suffix 28개(L74-90). 목록은 열려 있고 새 suffix는 이유를 적고 추가한다(L92) | §7 표 21개(L137-161). 닫힌 목록. 맞는 것이 없을 때만 추가한다 |
| trait 이름 | 역할 agent noun: `Publisher` `Recorder` `Reader` `Provider` `Verifier` `Reporter` `Limiter` `Visitor` `Source` `Scheduler` `Timer` `Renderer`(L79-86). 시계는 관용 이름 `Clock`(L84) | 모든 trait은 `Service` `Store` `Source` `Sink` `Gateway` `Control` `Clock` `Listener` 중 하나로 끝낸다. 예외는 `RelayStream` 하나(L25) |
| 구현체 이름 | `[구현 성격][port 이름]`(L47). 구현 성격 prefix 목록(L63-70) | `<기술><trait 이름>` 권장. 업무 값이 포트를 겸하면 업무 이름을 유지(L25) |
| 경계 데이터 | `Dto`(application·ports·adapter), `View`(ports·adapter)(L76-77) | `Dto` 금지(L182). `View`(app), `Response`(adapters)(L148, L160) |
| 오류 | `Error`, 모든 계층(L88). `Result` type alias는 만들지 않는다(L92) | 모듈마다 `Error` suffix enum(L31). 이름은 `<개념>Error` |
| 설정 | `Config`(L87) | `Settings`(app, 검증을 마친 값. L147) |
| 저장 포트 | 없음. `IRepository`·`RepositoryTrait`·`Impl` 표식 금지(L107) | `Store`. `Repository` 금지(L151, L181) |
| 유스케이스 | `Service`, `UseCase`(L78) | `Service`의 메서드. `UseCase`·`Interactor` 금지(L150, L180) |
| 금지 이름 | `Manager`, `CommonRepository` 같은 넓은 이름 피하기(L45) | `Manager` `Helper` `Util` `Handler` `UseCase` `Interactor` `Repository` `Dto` `Impl` `Connector` `Hasher` `Verifier` `Loader`(L177-184) |
| 모듈 경로와 이름 중복 | 되풀이하지 않는다. `session::SessionOwner` 대신 `session::Owner`(L17) | 모듈 이름이 prefix를 담으면 반복하지 않아도 된다(`matching::WaitingTable`. L113). 선택 사항 |
| 모듈 파일 | `foo.rs`와 `foo/`. `mod.rs`·`#[path]` 금지(L128) | 같은 방식. clippy `mod_module_files = deny`(`Cargo.toml:89`) |
| 이름 길이 | 5단어 미만 권장(L59) | 5단어 이상으로 짓지 않기를 권장([이름 짓기 기본 규칙](AGENTS.md#이름-짓기-기본-규칙)) |
| 이름 lint | 없음(`Cargo.toml:29-34`: `unsafe_code`, `missing_docs`, `unwrap_used`만) | 없음(`Cargo.toml:86-94`의 clippy 9개 중 이름 관련은 `mod_module_files`뿐) |

### B.2 역할별 비교표

"문서"는 컨벤션 문서의 규칙, "코드"는 실제 선언이다. 횟수는 부록 A의 집계 기준이다.

| 역할 | narumq-v2 문서 | narumq-v2 코드 | vnc-relay 문서 | vnc-relay 코드 |
| --- | --- | --- | --- | --- |
| 유스케이스 입구 | `Service`, `UseCase`(L78) | `Service` 3개. 단독 이름 2개(`application/src/audit.rs:113`, `application/src/tenantlimit.rs:186`), `QueryService`(`application/src/audit/query.rs:55`). `UseCase` 0개 | `Service`. `UseCase`·`Interactor` 금지 | `Service` 11개, 모두 `<개념>Service`. `AdminService`(`app/src/accounts.rs:198`), `RelayService`(`app/src/relay.rs:209`), `ReloadService`(`app/src/reload/service.rs:51`). trait `ConfigService`(`app/src/management/config.rs:116`) |
| 포트 trait | 역할 agent noun + `Clock` | trait 20개. `Recorder` 6(`application/src/ports/audit.rs:226`, `ports/metrics.rs:80`·`101`·`114`·`123`, `application/src/tenantlimit.rs:180`), `Source` 4(`ports/metrics.rs:332`, `ports/random.rs:9`, `adapters/metrics/src/registry.rs:37`, `adapters/runtimemetrics/src/source.rs:18`), `Provider` 2(`ports/authentication.rs:82`·`96`), `Publisher` 1(`ports/audit.rs:232`), `Reader` 1(`ports/audit.rs:325`), `Verifier` 1(`ports/authentication.rs:110`), `Renderer` 1(`ports/metrics.rs:147`), `Visitor` 1(`ports/metrics.rs:310`), `Clock`·`Timer`·`Scheduler`(`ports/clock.rs:6`·`12`·`21`) | trait은 `Service` `Store` `Source` `Sink` `Gateway` `Control` `Clock` `Listener` 중 하나 | app trait 26개가 규칙과 모두 일치. `Gateway` 7(`app/src/ports.rs:41`·`67`·`97`·`284`·`388`·`393`, `app/src/management/config.rs:104`), `Source` 7(`app/src/ports.rs:103`·`300`·`306`·`331`·`375`·`381`, `app/src/reload/ports.rs:11`), `Store` 5(`app/src/ports.rs:24`, `app/src/management/config.rs:94`, `app/src/reload/ports.rs:15`·`23`·`48`), `Control` 2(`app/src/ports.rs:311`·`326`), `Clock` 2(`app/src/ports.rs:88`, `app/src/reload/ports.rs:56`), `Sink` 1(`app/src/ports.rs:19`), `RelayStream`(`app/src/ports.rs:167`). adapters·bin에도 `Listener`(`adapters/src/listener.rs:11`), `WatchGateway`(`adapters/src/file_watch/feed.rs:213`), `ServiceInstallGateway`(`src/service.rs:112`) |
| 어댑터 구현체 | `[구현 성격][port 이름]` | 일치: `FileAuditPublisher`(`adapters/auditlog/src/publisher.rs:41`), `JwksAuthenticationProvider`(`adapters/authentication/src/jwks.rs:52`), `SystemClock`(`adapters/clock/src/lib.rs:15`), `SystemRandomSource`(`adapters/random/src/lib.rs:9`), `TokioScheduler`(`adapters/clock/src/lib.rs:27`). 불일치: `JwtVerifier`가 `TokenVerifier`를 구현(`adapters/authentication/src/jwt.rs:69`, `:214`). `Registry`가 port 3개를 구현(`adapters/metrics/src/registry.rs:144`, `snapshot.rs:11`, `text.rs:15`). 문서는 후자를 허용 | `<기술><trait 이름>` 권장. 업무 값이 겸하면 업무 이름 | 일치: `TcpTargetGateway`(`adapters/src/target.rs:17`), `MmdbFileGateway`(`adapters/src/ip_db.rs:56`), `JsonLinesEventSink`(`adapters/src/ops_log.rs:41`), `Argon2PasswordHashGateway`(`adapters/src/accounts_file/hasher.rs:16`), `SystemClock`(`adapters/src/clock.rs:12`). 예외: `AccountsFile`이 `AccountStore`를 구현(`adapters/src/accounts_file/store.rs:22`, `:85`) |
| 저장 포트(Repository 자리) | 없음 | 0개. 계획만 있다: `ports/{controlstore,sessionstore,deliverylog}`(`docs/architectures/project-structure.md:153-154`) | `Store`. `Repository` 금지 | `Store` 10개. trait 5(위 표) + 구현 5: `FsConfigFileStore`(`adapters/src/config_edit.rs:20`), `FsReloadFileStore`(`adapters/src/file_watch.rs:72`), `RustlsTlsStore`(`adapters/src/tls.rs:183`), `TicketJwksStore`(`adapters/src/ticket/jwks.rs:18`) 등 |
| 오류 | `Error`, 모든 계층 | 오류 타입 18개. 단독 `Error` 14개(`domain/src/session.rs:23`, `domain/src/routing.rs:30`, `domain/src/security.rs:20`, `domain/src/tenancy.rs:20`, `domain/src/mqtt/error.rs:8`, `application/src/audit.rs:38`, `application/src/tenantlimit.rs:64`, 어댑터 6개 `error.rs`: `adapters/tls/src/error.rs:11` 등. `sim/src/lib.rs:30` 포함). 접두 이름 4개: `AdapterError`(`application/src/ports.rs:27`), `UnknownResultError`(`ports/audit.rs:18`), `RecordsUnavailableError`(`ports/audit.rs:250`), `ValidationError`(`application/src/audit/validate.rs:14`). 표준 `Error`와 겹쳐 `as StdError` 별칭이 3곳 | 모듈마다 `Error` suffix enum | `Error` 42개 모두 `<개념>Error`: `TlsError`(`adapters/src/tls.rs:35`), `SessionError`(`domain/src/session.rs:200`), `ConfigError`(`app/src/settings.rs:506`). 문서 목록에 없는 `Failure` 8개가 오류 역할을 한다: `ListenerFailure`(`adapters/src/listener.rs:28`), `BindFailure`(`adapters/src/net.rs:203`), `AdminFailure`(`app/src/accounts.rs:118`), `IpDbFailure`(`app/src/ip_info.rs:50`), `WatchFailure`(`app/src/reload.rs:102`) |
| 설정 | `Config` | 단독 `Config` 5개(`adapters/tls/src/config.rs:152`, `adapters/auditlog/src/publisher.rs:20`, `adapters/authentication/src/jwks.rs:38`, `adapters/runtimemetrics/src/sampler.rs:14`, `adapters/websocket/src/listener.rs:43`). 접두 이름 4개: `ServiceConfig`(`application/src/audit.rs:82`), `VerifierConfig`(`adapters/authentication/src/jwt.rs:25`), `FetchConfig`(`…/keyset.rs:26`), `DataPlaneRecorderConfig`(`application/src/audit/dataplane.rs:28`) | `Settings`(app) | `Settings` 27개(`app/src/settings.rs:20`·`40`·`64`·…). 어댑터 내부 `AcceptedSettings`(`adapters/src/net.rs:379`), `UpgradedSettings`(`adapters/src/websocket.rs:321`). 문서 밖 `Config` 2개: `RelayRuntimeConfig`(`app/src/relay.rs:455`), `Config`(`tray/src/worker.rs:156`) |
| 경계 데이터(DTO·출력 모델) | `Dto`, `View` | `Dto` 22개(`application/src/ports/audit.rs:85`·`140`, `ports/authentication.rs:17`, `application/src/audit.rs:95`, `audit/query.rs:20`·`45`). `View` 6개(`ports/metrics.rs:270`·`280`, `adapters/auditlog/src/wire.rs:140`·`157`·`164`·`173`) | `Dto` 금지. `View`(app), `Response`(adapters) | `Dto` 0개. `View` 16개(`app/src/management.rs:56`, `app/src/management/query.rs:20`). `Response` 2개(`adapters/src/ops_log.rs:314`·`333`) |
| 저장·전송 형식(어댑터 내부) | 목록에 없음 | `Wire` 5개(`adapters/auditlog/src/chain.rs:43`, `wire.rs:16`·`37`·`45`·`57`) | `Response`로 흡수(L160) | `Wire`는 비공개 소켓 enum으로 뜻이 다르다(`adapters/src/websocket.rs:118`) |
| 이벤트·명령·요청 | 목록에 없음 | 감사 이벤트는 `AuditEventDto`(`ports/audit.rs:140`), 명령은 `RecordCommandDto`(`application/src/audit.rs:95`). MQTT 패킷은 규격 표기(`Publish` `domain/src/mqtt/publish.rs:9`, `PubAck` `domain/src/mqtt/ack.rs:13`) | `Event`는 업무 prefix 표에만 있다(L133) | `Event`(`app/src/events.rs:378`), `SourceEvent`(`app/src/sources.rs:51`), `SessionCommand`(`app/src/relay/session.rs:33`), `HandshakeRequest`(`domain/src/handshake.rs:113`), `Reason` enum 7개(`app/src/events.rs:94`·`184`·`271`·…) |
| 결과 분류 | `Outcome`(enum), `Result`(묶음 struct)(L90, L92) | `Outcome` 1개(`ports/audit.rs:30`). `Result` struct 3개(`adapters/auditlog/src/verify.rs:10`, `domain/src/session.rs:117`·`128`) | 목록에 없음 | `Result` 3개: enum `ReloadResult`(`app/src/reload.rs:33`), enum `AdminTerminateResult`(`app/src/events.rs:356`), struct `ConfigEditResult`(`app/src/management/config.rs:55`). `Outcome` 2개(`app/src/reload/service.rs:32`, `domain/src/matching.rs:91`) |
| 진입점·리스너 | 목록에 없음(Target `adapters/tcp`) | 단독 `Listener` 2개(`adapters/tls/src/listener.rs:17`, `adapters/websocket/src/listener.rs:89`) | `EntryPoint`, `Listener`(L157-158) | `TcpEntryPoint`(`adapters/src/net.rs:357`), `WsEntryPoint`(`adapters/src/websocket.rs:49`), `ManagementListener`(`adapters/src/management_http.rs:73`), `StatusListener`(`adapters/src/status_http.rs:33`). 도메인 enum `Listener`(`domain/src/entry_point.rs:7`)가 trait `Listener`와 이름이 같다 |
| 상태·스냅샷 | `State`(L89) | `State` 7개. 단독 이름은 모두 비공개·`pub(crate)`(`adapters/auditlog/src/chain.rs:64`, `…/publisher.rs:48`, `adapters/authentication/src/keyset.rs:70`, `adapters/websocket/src/listener.rs:68`). `Snapshot` 1개(비공개 `adapters/auditlog/src/reader.rs:19`). 지표 스냅샷은 `MetricsSnapshotDto`(`ports/metrics.rs:300`) | `Snapshot`(app)(L149) | `Snapshot` 9개: `RelaySnapshot`(`app/src/relay.rs:493`), `ReloadSnapshot`(`app/src/reload/snapshot.rs:7`), `MetricsSnapshot`(`app/src/metrics/snapshot.rs:84`). 어댑터에도 있다: `QuerySnapshot`(`adapters/src/ops_log.rs:383`). `State` 10개(`domain/src/session.rs:136`, `app/src/ip_info.rs:169`) |
| 정책·규칙 | 목록에 없음 | `Policy`(`domain/src/security.rs:318`), `Rule`(`domain/src/security.rs:154`) | `Policy`(domain)(L143) | `Policy` 8개. domain 6개(`domain/src/target_policy.rs:111`, `domain/src/handshake.rs:37`, …), adapters 2개(`adapters/src/net.rs:251`, `adapters/src/websocket.rs:35`) |
| 식별자·키 | `Key`(L89) | `TenantId`(`domain/src/tenancy.rs:52`), `SessionId`(`domain/src/routing.rs:70`), `SessionKey`(`domain/src/tenancy.rs:83`) | `Id`, `Key`(L141-142) | `RelayId`(`domain/src/relay_id.rs:8`), `SessionId`(`domain/src/session.rs:14`), `ConnId`(`domain/src/conn_id.rs:15`), `TicketKey`(`domain/src/ticket.rs:201`) |
| 메모리 모음 | 목록에 없음 | `Table`(`domain/src/routing.rs:145`), `Blocklist`(`domain/src/security/blocklist.rs:186`) | `Table`, `List`(L144-145) | `WaitingTable`(`domain/src/matching.rs:161`), `AllowList`(`domain/src/allow_list.rs:135`) |
| 빌더 | 없음 | `Builder` 타입은 비공개 `PageBuilder` 1개(`adapters/auditlog/src/reader.rs:143`). `with_*` 메서드 3개(`adapters/runtimemetrics/src/sampler.rs:78`) | 없음 | `Builder` 0개. `with_*` 메서드 10개(`adapters/src/net.rs:48`) |
| 확장 trait(`Ext`) | 없음 | 0개 | 없음 | 0개 |
| 클라이언트·핸들 | 없음 | `Client`·`Handle` 0개 | 없음 | `Client`·`Handle` 0개. 자원 소유 타입은 업무 이름: `Scm`(`adapters/src/windows_service.rs:83`), `RelayRuntime`(`app/src/relay.rs:460`). RAII는 `Guard` 4개(`app/src/relay.rs:1111`·`1192`, `app/src/relay/matching.rs:100`) |
| 시험 대역 | `Fake` `Stub` `Spy` `Mock`(L70) | `FakeClock`(`sim/src/lib.rs:53`), `FakeNetwork`, `FakeDisk`(`sim/src/lib.rs:21-22`) | `Fake`·`Test` 권장(L26) | `FakeStore`(`app/src/accounts.rs:399`), `FakeDenyControl`(`app/src/reload/service.rs:1865`) |
| 금지 이름(`Impl` `Manager` `Helper` `Util` `Controller` `Interactor` `UseCase` `Repository` `Port`) | `IRepository`·`RepositoryTrait`·`Impl` 표식 금지(L107). 나머지는 문서에 없다 | 9개 모두 0개. 시험 하네스 `Handler` 1개만 있다(`sim/src/network.rs:12`) | `Manager` `Helper` `Util` `Handler` `UseCase` `Interactor` `Repository` `Dto` `Impl` `Connector` `Hasher` `Verifier` `Loader` 금지(L177-184) | 금지 이름 사용 0개. `Impl` 금지와 구현체 이름 규칙은 코드와 충돌하지 않는다 |
| 문서에만 있음 / 코드에만 있음인 suffix | §3.3 표에 `Reporter` `Limiter` `Encoder` `Decoder`가 있다 | `Reporter` 0개(문서에만 있음). `Limiter` 1개(`PerSecondLimiter` `application/src/audit.rs:223`), `Encoder` 1개(`domain/src/mqtt/encoder.rs:6`), `Decoder` 1개(`domain/src/mqtt/decoder.rs:13`). 문서에 없고 코드에만 있음: `Wire` 5, `Binding` 3, `Registry`, `Sampler`(M3) | `Reporter` `Limiter` `Decoder`는 21개 목록에 없다 | 코드에만 있음: `Reporter` 2개(`adapters/src/windows_service.rs:288`·`290`), `Limiter` 1개(`LoginLimiter` `domain/src/login_limit.rs:111`), `Decoder` 1개(`FrameDecoder` `adapters/src/websocket_stream/frame.rs:198`), `Failure` 8, `Reason` 7 |

### B.3 모듈 배치와 import 스타일

| 항목 | narumq-v2 | vnc-relay |
| --- | --- | --- |
| 계층 경계 | crate 경계. `crates/domain`, `crates/application`, `crates/adapters/<기술>` 10개, 조립 `crates/narumq`(Target)(`docs/architectures/project-structure.md:35-38`) | crate 경계. `crates/domain`, `crates/app`, `crates/adapters`, `crates/tray`, 루트 `src/`가 실행파일(`docs/architectures/code-map.md:14`) |
| 계층 안 배치 | **기능 우선 모듈**. domain은 `mqtt` `routing` `security` `session` `tenancy`(`domain/src/lib.rs:6-10`), application은 `audit` `ports` `tenantlimit`(`application/src/lib.rs:5-7`), 포트는 `ports/<기능>.rs`(`application/src/ports.rs:10-14`) | **기능 파일 평면**. domain 19개(`domain/src/lib.rs:6-24`), app 11개(`app/src/lib.rs:4-14`), adapters 약 28개(`adapters/src/lib.rs:4-33`). 포트는 `ports.rs` 한 파일에 18개(`app/src/ports.rs:19-393`) |
| 공개 경로 | domain·application은 `pub mod`. 어댑터는 비공개 `mod` + crate 루트 `pub use`(`adapters/tls/src/lib.rs:10-20`) | 거의 모두 `pub mod`. 비공개는 `admission` `request_deadline` `tls_listener` `toml_text` `upgrade`뿐(`adapters/src/lib.rs`) |
| 소비자 쪽 경로 | `narumq_domain::session::Lifecycle`, `narumq_application::ports::audit::AuditPublisher`, `narumq_adapter_tls::Config` | `vnc_relay_domain::account::Account`, `vnc_relay_app::accounts::AccountsError`(`adapters/src/accounts_file.rs:13-17`) |
| `pub use` | 38줄(어댑터 `lib.rs`, `domain/src/mqtt.rs:32-42`, `application/src/audit.rs:29-31`) | 운영 코드 7줄(`adapters/src/accounts_file.rs:21-22`, `adapters/src/ticket.rs:8`, `app/src/metrics.rs:28`, …) |
| prelude 모듈 | 0개 | 0개 |
| `use` 스타일(운영 코드, 내부 경로) | 타입 직접 import 243, 모듈 import 61(`adapters/auditlog/src/publisher.rs:9-13`) | 타입 직접 import 983, 모듈·함수 import 116, `super::*` 62(시험 모듈) |
| 모듈 이름 형태 | 한 단어, 붙여 씀: `walframe` `runtimemetrics` `diskspace` `auditlog` `tenantlimit` | `snake_case` 다단어: `admin_session` `allow_list` `ip_db` `process_stats` `management_http` |
| 같은 이름의 공개 타입 | 5개 이름, 선언 24개: `Error` 13, `Config` 5, `State` 2, `Listener` 2, `Service` 2 | 2개 이름, 선언 4개: `Listener` 2(`domain/src/entry_point.rs:7`, `adapters/src/listener.rs:11`), `TargetHost` 2(`domain/src/handshake.rs:54`, `domain/src/target_policy.rs:11`) |
| 충돌 회피 별칭 | `Error as StdError` 3곳(`adapters/auditlog/src/error.rs:1`, `adapters/runtimemetrics/src/error.rs:1`, `application/src/audit.rs:8`) | `Listener as SocketListener`(`adapters/src/websocket.rs:25`) |
| `module_name_repetitions` 해당(추정) | 공개 모듈의 `pub` 타입 87개 중 17개(전부 `application/src/ports/*`) | 328개 중 106개(`app/src/settings.rs`의 `*Settings` 24개, `app/src/management/config.rs`의 `Config*` 13개 등). 모듈 이름과 똑같은 타입 15개는 기본 설정에서 허용 |
| `module_inception` | 해당 없음 | 해당 없음(운영 코드 기준. 시험 파일 2개가 같은 이름을 쓰나 crate 루트다) |
| `mod.rs` / `#[path]` | 0개 / 0개 | 0개 / 시험 파일 5개(`adapters/tests/ip_db.rs:14`, `adapters/tests/websocket_ticket.rs:38`, `src/serve.rs:1370`, `tests/dashboard_browser.rs:13`, `tests/dashboard_live.rs:7`) |

## 부록 C. 충돌 목록

### C.1 저장소 사이: 같은 역할에 다른 이름

| ID | 역할 | narumq-v2 | vnc-relay | 비고 |
| --- | --- | --- | --- | --- |
| C1 | 경계 데이터 | `Dto` 22개 | `Dto` 금지. `View` 16개 + `Response` 2개 | 같은 역할에 서로 반대 규칙. narumq-v2 `Dto`는 입력과 출력을 모두 덮는다 |
| C2 | 출력·기록 포트 | `Publisher`(1) `Recorder`(6) | `Sink`(1) | 같은 "사건·기록을 내보낸다" 역할. vnc-relay에도 `MetricsSink`(`app/src/metrics.rs:304`), narumq-v2에도 `MetricsRecorder`(`ports/metrics.rs:123`)가 있다 |
| C3 | 읽기·검증 포트 | `Reader` `Provider` `Verifier` `Renderer` | `Source` `Gateway`. `Verifier`·`Loader`는 금지 | narumq-v2 `AuthenticationProvider`와 vnc-relay `PasswordVerifyGateway`(`app/src/ports.rs:97`)는 같은 성격의 외부 능력이다 |
| C4 | 저장 포트 | 어휘 없음 | `Store` | narumq-v2가 합의 저장소 포트(Target)를 만들 때 이름을 새로 정해야 한다 |
| C5 | 오류 이름 | 단독 `Error` 14 | `<개념>Error` 42 | narumq-v2는 `std::error::Error`와 겹쳐 `StdError` 별칭 3곳 |
| C6 | 설정 | `Config` | `Settings` 27 | vnc-relay는 `Config*`를 "설정 파일" 개념(`ConfigView` `ConfigPatch` 등)에 쓰기도 한다 |
| C7 | 유스케이스 이름 | `Service` 단독 2개 + `QueryService` | `<개념>Service` 11개 | 단독 `Service` 2개는 `application` crate 안에서 같은 이름이다 |
| C8 | 결과 분류 | `Result` 3(struct) + `Outcome` 1(enum) | `Result` 3(enum 2 + struct 1) + `Outcome` 2 | vnc-relay `ReloadResult`는 enum이라 narumq-v2 규칙(L92)으로는 `Outcome`이다 |
| C9 | crate 이름 | `narumq-application`, `narumq-adapter-<기술>` | `vnc-relay-app`, `vnc-relay-adapters` | 계층 이름과 단수·복수가 다르다 |
| C10 | 모듈 이름 | 붙여 쓴 한 단어 | `snake_case` 다단어 | [Rust API Guidelines C-CASE](https://rust-lang.github.io/api-guidelines/naming.html)는 모듈에 `snake_case`를 요구하고 `btree_map`을 예로 든다 |
| C11 | 시험 파일 `#[path]` | 금지 | 시험 5곳에서 사용 | |
| C12 | 모듈 경로와 이름 중복 | 문서는 금지하나 코드는 `ports/*`에서 반복 | 문서는 선택, 코드는 혼재 | 부록 C.2의 M1·M2·M9 |

### C.2 문서와 코드 사이

각 행의 ID는 M1~M10이다.

| ID | 저장소 | 문서 | 코드 | 위치 |
| --- | --- | --- | --- | --- |
| M1 | narumq-v2 | 모듈 경로에 있는 이름은 되풀이하지 않는다(L17) | `ports/audit.rs`의 `AuditPublisher` `AuditRecordReader` `AuditEventDto`, `ports/authentication.rs`의 `AuthenticationProvider`, `ports/metrics.rs`의 `MetricsRecorder`, `ports/random.rs`의 `RandomSource` 등 17개가 모듈 이름으로 시작한다(추정) | `application/src/ports/*.rs` |
| M2 | narumq-v2 | 같은 문서의 §3.1은 `[구현 성격][port 이름]`을 쓰라고 한다(L47). 어댑터 crate 안에서는 `adapters/auditlog`의 `FileAuditPublisher`처럼 crate 이름을 되풀이한다 | 같은 crate의 `Config` `Error` `Listener`는 단독 이름이다. 한 문서 안에서 되풀이와 생략이 섞인다 | `adapters/auditlog/src/publisher.rs:20`, `:41` |
| M3 | narumq-v2 | suffix 표에 `Wire`·`Binding`이 없다 | `Wire` 5개(`adapters/auditlog/src/chain.rs:43`, `wire.rs:16`), `Binding` 3개(`adapters/authentication/src/jwks.rs:21`, `mtls.rs:69`, `password.rs:130`), `Registry`(`adapters/metrics/src/registry.rs:48`), `Sampler`(`adapters/runtimemetrics/src/sampler.rs:25`) | 컨벤션 L92는 목록에 이유를 쓰고 추가하라고 한다 |
| M4 | narumq-v2 | 어댑터는 `[구현 성격][port 이름]` | `JwtVerifier`는 `TokenVerifier` 구현이지만 `JwtTokenVerifier`가 아니다 | `adapters/authentication/src/jwt.rs:69`, `:214` |
| M5 | narumq-v2 | 코드 지도가 `NewService`를 적는다. Rust 코드에는 `Service::new`가 있다 | Go 시절 이름이 남아 있다 | `docs/architectures/project-structure.md:209` |
| M6 | vnc-relay | `Policy`는 domain, `Snapshot`은 app | 어댑터에도 있다: `AcceptPolicy`(`adapters/src/net.rs:251`), `WsPolicy`(`adapters/src/websocket.rs:35`), `QuerySnapshot`(`adapters/src/ops_log.rs:383`) | `docs/conventions/rust.md:143`, `:149` |
| M7 | vnc-relay | `Control`은 실행 중 중계에 명령을 넣는 포트(L155) | 포트가 아닌 타입에도 쓴다: `IdleControl`(`adapters/src/request_deadline.rs:49`), `StopControl`(`adapters/src/windows_service.rs:198`) | |
| M8 | vnc-relay | 목록에 `Failure` `Reason` `Guard` `Reply` `Pair` `Item` `Plan`이 없다. 문서는 "데이터 모양을 설명하는 말은 업무 이름의 일부"라고 한다(L113) | `Failure` 8개는 `Error`와 같은 역할이다(`app/src/accounts.rs:118`, `adapters/src/listener.rs:28` 등) | |
| M9 | vnc-relay | 문서는 모듈 이름 반복을 선택으로 둔다(L113) | 같은 계층 안에서 섞인다: `matching::WaitingTable`(반복 없음), `ticket::TicketPolicy`(`domain/src/ticket.rs:115`, 반복), `account::AccountName`(`domain/src/account.rs:9`, 반복) | |
| M10 | vnc-relay | 같은 이름 방지는 규칙이 없다 | 도메인 enum `Listener`와 어댑터 trait `Listener`가 같은 이름이다. `TargetHost`가 한 crate에 2개다 | `domain/src/entry_point.rs:7`, `adapters/src/listener.rs:11`, `domain/src/handshake.rs:54`, `domain/src/target_policy.rs:11` |

## 부록 D. 결정 기록

### 결정 배경

- 가장 큰 충돌 다섯 가지: (1) 경계 데이터 `Dto` 22개 대 `View`·`Response`(C1), (2) 포트 역할 단어 `Publisher`·`Recorder` 대 `Sink`·`Store`·`Gateway`(C2·C3·C4), (3) 오류 이름 단독 `Error` 14개 대 `<개념>Error` 42개(C5), (4) 설정 `Config` 대 `Settings` 27개(C6), (5) 모듈 경로 반복 정책과 crate 이름(C9·C12).
- 문서와 코드가 어긋나는 곳은 narumq-v2에 `ports/*`의 17개 이름과 `Wire`·`Binding` 등 문서 밖 suffix, vnc-relay에 `Failure` 8개와 포트가 아닌 `Control` 2개가 있다(부록 C.2).

### 결정 항목

주인(kayden.kim)이 2026-11-24에 아래 10개 항목을 채택했다. D1은 주인이 (c)를 직접 정했고, D2~D10은 작성 단계에서 제시한 추천안을 그대로 채택했다.

| # | 결정 | 채택안 | 검토한 선택지 | 이유 | 결정일 | 결정자 |
| --- | --- | --- | --- | --- | --- | --- |
| D1 | 이름에 역할을 나타내는 방식 | (c) 개념 타입은 경로(P), 역할 타입은 suffix(S)(2.2절) | (a) 모든 타입을 경로로(P, std식). (b) 모든 타입에 suffix(S) | std 관행과 clippy 모범(경로)을 개념 타입에 따르고, 직접 import되는 역할 타입의 충돌과 grep 문제를 suffix로 푼다. (a)는 조립 코드에서 동명 충돌이 크고(narumq-v2 `Error` 13개), (b)는 도메인 이름에 `…Entity`가 붙어 어색하다 | 2026-11-24 | 주인 |
| D2 | 포트 역할 어휘 | (c) 닫힌 6개 `Source` `Sink` `Store` `Gateway` `Control` `Clock` + 등록 suffix `Provider` `Verifier` `Renderer` `Visitor` `Scheduler` `Timer`(7절) | (a) vnc-relay식 닫힌 6개만. (b) narumq-v2식 열린 agent noun | 닫힌 어휘는 이름 고르기를 쉽게 한다. 단 `Verifier`처럼 동작이 이름에 남아야 하는 포트는 이유를 적고 등록한다. (a)만 쓰면 narumq-v2 인증 포트가 `Gateway`로 뭉개진다 | 2026-11-24 | 주인 |
| D3 | `Repository` 대 `Store` | (a) `Store` | (b) `Repository`. (c) 둘 다 허용 | `Store`는 집합체 단위 저장이라는 DDD 의미를 요구하지 않고 `Source`·`Sink`와 어휘가 맞는다. 두 저장소 중 vnc-relay만 쓴다. (c)는 같은 역할 두 단어라 원칙 4에 어긋난다 | 2026-11-24 | 주인 |
| D4 | 유스케이스 | (a) `<개념>Service`, 메서드가 사용 사례 | (b) 사용 사례마다 타입(`PlaceOrder`). (c) `UseCase` suffix | 두 저장소가 이미 `Service`를 쓴다. (b)는 타입이 늘어난다. X4의 남용 방지 규칙을 같이 둔다 | 2026-11-24 | 주인 |
| D5 | 오류 이름 | (a) 항상 `<개념>Error` | (b) 모듈·crate마다 단독 `Error`. (c) crate 루트의 오류만 단독 `Error` | 동명 충돌(조건 2)과 `std::error::Error` 별칭을 모두 없앤다. 두 저장소 합계는 `<개념>Error`가 42개로 단독 14개보다 많다 | 2026-11-24 | 주인 |
| D6 | `Config`와 `Settings` | (a) 둘을 구분: `Config`는 생성 옵션, `Settings`는 검증을 마친 설정 모델 | (b) `Config`로 통일. (c) `Settings`로 통일 | vnc-relay는 `Config*`를 "설정 파일" 개념에도 쓰므로(`ConfigView` `ConfigPatch`) 두 단어가 의미를 나눈다. narumq-v2의 `Config`(생성 옵션)와도 겹치지 않는다. (b)는 vnc-relay `Settings` 27개, (c)는 narumq-v2 `Config` 9개를 바꾼다 | 2026-11-24 | 주인 |
| D7 | 경계 데이터 | (b) `Dto` 금지. 방향별로 `Command` `Query` `View` `Request` `Response` | (a) `Dto` 유지. (c) 개념 이름만(suffix 없음) | `Dto`는 입력과 출력을 모두 덮어 역할을 말하지 않는다. vnc-relay는 이미 (b)를 쓴다. 바뀌는 쪽은 narumq-v2 22개다 | 2026-11-24 | 주인 |
| D8 | `module_name_repetitions` | (a) 켜지 않는다(경로 반복 허용) | (b) 켠다(모든 타입을 P로). (c) 도메인 crate에서만 켠다 | S 방식 역할 타입이 모듈 이름과 겹치는 것을 허용하기 때문이다. 이 lint는 `restriction`이라 기본으로 꺼져 있다. lint 설정 변경은 이 표준의 범위 밖이다 | 2026-11-24 | 주인 |
| D9 | crate 이름 | (a) `application`, 어댑터는 `adapters`(한 crate) 또는 `adapter-<기술>`(기술별) | (b) `app`. (c) 프로젝트마다 자유 | `app`은 실행 파일(앱)과 헷갈린다. 단수·복수는 구성에 따라 고정해 2.5절에 쓴다. 바뀌는 쪽은 vnc-relay `app` crate 하나다(import 338줄, 68파일) | 2026-11-24 | 주인 |
| D10 | `Result` suffix | (a) 금지. 분류는 `Outcome`, 묶음은 개념 이름 | (b) narumq-v2식 허용: struct 묶음만 `Result`. (c) 모두 허용 | `fmt::Result`·`io::Result`는 alias라 독자가 `XResult`를 alias로 읽는다. 두 저장소의 `Result` 6개가 enum과 struct로 섞여 있어(C8) 기준이 이미 갈린다 | 2026-11-24 | 주인 |

위치 결정: 이 문서의 최종 위치는 `docs/conventions/rust-naming-standard.md`다(결정일 2026-11-24, 결정자 주인. 부록 E.1).

## 부록 E. 문서 위치와 범위

### E.1 위치

이 문서는 `docs/conventions/rust-naming-standard.md`에 둔다. 주인이 2026-11-24에 이 위치를 최종 위치로 확정했다. [변경에 따른 문서 선택](../README.md#변경에-따른-문서-선택)이 "이름 짓기·서식·코드 작성 방식 변경"을 컨벤션 문서로 정한다. [템플릿 규칙](../templates/AGENTS.md#편집과-복제)이 정한 `coding-convention.template.md`의 머리 표와 1~7절 구조에 맞춰 썼고 [컨벤션 목록](README.md)에 `Active`로 등록했다.

- 이 저장소의 `docs/`는 프로젝트 문서의 틀이다. 틀을 복제한 프로젝트에 이 문서도 따라갈 수 있다. 이 점은 위치를 정할 때 위험으로 보고했고, 주인이 이 위치로 확정했다.
- 각 프로젝트의 Rust 컨벤션은 이 표준을 링크하고 프로젝트 고유 항목(도메인 prefix 목록, 등록한 suffix 예외)만 적는다. [컨벤션 작성 규칙](AGENTS.md#작성-규칙)이 "대상에 이미 있는 규칙 문서를 옮겨 쓰지 않고 링크한다"고 정한 방식과 같다.

템플릿 8절(테스트와 스펙 시나리오 연결)은 해당하지 않아 지웠다. 현황·충돌·결정 기록은 템플릿에 없는 내용이라 부록으로 붙였다.

### E.2 범위 밖

이 표준은 다음을 하지 않는다.

- narumq-v2와 vnc-relay의 코드 수정과 rename
- 두 저장소의 `AGENTS.md`와 컨벤션 문서에 표준 반영(각 저장소의 별도 작업이다)
- lint 설정 변경
