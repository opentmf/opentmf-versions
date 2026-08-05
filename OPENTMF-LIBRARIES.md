# OPENTMF-LIBRARIES.md — capability quick reference

What the shared `opentmf` libraries (`~/prj/opentmf`) provide and how to wire them, so you can pick the
right one without reading their source. **Rule:** whenever possible reuse these instead of hand-rolling
HTTP, persistence, locking, auth, serialization, or query handling.

All artifacts are version-managed by the **opentmf-versions BOM** — import it once and omit `<version>`
on individual deps. Source of truth is the code on each repo's `develop` branch; this file is a summary.

> **Last verified against BOM `opentmf-versions` 2.1.16 (2026-08-05).** Put the BOM
> version you checked against here on every refresh — it is the one fact that tells a
> reader whether this file is current, and it is checkable in one glance against the
> BOM's own version.
>
> **⚠ Do not restate library VERSION NUMBERS in the prose below.** The BOM is their
> single source of truth; a number copied here is a cache with no invalidation and it
> will drift. Version numbers appear here only where they mark *when a behaviour
> changed* ("since 2.1.6…", "2.2.0 fixed…") — that is history, which the BOM does not
> carry. What this file is FOR is the behaviour, the wiring and the **gotchas** — none
> of which live anywhere else.
>
> **Refreshing this doc.** It's a hand-built summary, not generated — regenerate it whenever the
> libraries change (new module, new config key, new bean, removed capability). Each folder under
> `~/prj/opentmf` is its own git repo; skip `retired/` and non-git folders (`code-generators/`,
> `opentmf-compose/`). Procedure: for each library repo, ensure it's on `develop` and `git pull`,
> then re-inspect `pom.xml`, `META-INF/spring/...AutoConfiguration.imports`, `@ConfigurationProperties`
> classes (prefixes + default values), exported public beans/annotations/utils, and `README.md`.
> Keep every entry in the fixed template (**What · Use it for · Key API · Config · Snippet · Gotchas ·
> Maven**) and verify names against the actual source — don't guess. An AI agent can do this by
> fanning out one inspection task per library group and reassembling the entries.

## Pick by task

| I need to… | Reach for | Entry point |
|---|---|---|
| Pin all opentmf versions | opentmf-versions | BOM `import` |
| Jackson 3 mapper / JSON utils / safe-text validation | opentmf-commons | `JacksonUtil`, `@SafeId/@SafeText` |
| RFC 6902/7396 patch on Jackson 3 | opentmf-json-patch | `JsonPatch`, `JsonMergePatch` |
| Standard error codes + RFC 9457 / TMF error bodies | opentmf-errors | `CommonErrorCode`, `ProblemDetailRenderer` / `TmfErrorRenderer` |
| TMF v4 type contract (interfaces) | opentmf-v4-api | `I*` interfaces |
| TMF v4 concrete DTOs | opentmf-v4-models / dnext-v4-models | `ServiceOrder`, `Tmf641JacksonConfig` |
| Read characteristics / related parties off a model | opentmf-v4-utils | `CharacteristicUtil`, `RelatedPartyUtil` |
| Call another TMF service (CRUD/patch/paging), **token auto-attached** | opentmf-api-clients | `GenericTmfClient` bean |
| Raw outbound HTTP transport (**you attach the token** via `<id>TokenService`) | opentmf-http-clients | `<id>RestClient`, `SyncClientUtil` |
| TMF-630 filter/sort/paging/fields on a read endpoint | tmf630-toolkit | `@QuerydslPredicate`, `@Tmf630Response` |
| Secure a service with JWT + RBAC by config | openid-rbac-security | `opentmf.security.*` |
| `@CreatedBy`/`@CreatedDate` auditing | auditor-aware | `auditorAware` beans |
| Cluster-wide lock (deploy/sync once) | opentmf-db-lock-service | `@UsingClusterLock`, `LockContext` |
| Run a workflow engine server | opentmf-camunda7 | deployable image |
| Deploy bundled BPMN on boot | camunda7-bpmn-sync-service | `opentmf.bpmn-sync.*` |
| Log workflow incidents | camunda7-incident-logger | drop-in plugin |
| Integration-test a BPMN flow | camunda7-test-framework | `BaseBpmIT` |
| Sync bundled catalogs on boot | dnext-catalog-sync-service | `opentmf.catalog-sync.*` |
| Mock TMF backends + OIDC in tests | opentmf-mockserver | `Dynamic*Callback` |
| Wire a mock TMF backend + OIDC into a JUnit 5 / Spring Boot IT | opentmf-mockserver-test-support | `MockServerSupport.shared()` |
| Kick one HTTP endpoint from a CronJob | http-endpoint-kicker | Docker image |

---

## Core & serialization

### opentmf-versions
**What:** A pom-only Maven BOM pinning compatible versions for the whole `org.opentmf.*` family.
**Use it for:** Any project consuming opentmf libraries — import once, never set opentmf versions on deps.
**Key API:** `<dependencyManagement>` import; no code.
**Config:** none.
**Snippet:**
```xml
<dependency><groupId>org.opentmf</groupId><artifactId>opentmf-versions</artifactId>
  <version>2.1.x</version><type>pom</type><scope>import</scope></dependency>
```
**Gotchas:** Pin a fixed version. 2.x = Spring Boot 4 + Jackson 3; 1.x = Boot 3.5 + Jackson 2. Overriding a BOM-managed version defeats the purpose (do it only for a CVE, with a comment).
**Maven:** `org.opentmf:opentmf-versions` (the BOM).

### opentmf-commons
**What:** Dependency-light utilities: a preconfigured Jackson 3 mapper, on-demand validation, and security-hardening field annotations.
**Use it for:** JSON (de)serialize/merge/convert, ad-hoc validation without a Spring context, hardening TMF DTO fields against injection.
**Key API:** `JacksonUtil` (`defaultMapperBuilder`, `getDefaultJsonMapper`, `setDefaultJsonMapper`, `objectToJson`/`jsonToObject`, `objectToTree`/`treeToObject`, `convertValue`, `merge`, `objectToMap`); `ValidationUtil.validate/ensureValid`; `ListUtil.safe`; annotations `@Required`, `@SafeText`, `@SafeId`, `@SafeJsonPath`, `@SafeUrl`.
**Config:** none — no autoconfig. Wire the mapper yourself in a `@Configuration`.
**Snippet:**
```java
var mapper = JacksonUtil.defaultMapperBuilder().build();
JacksonUtil.setDefaultJsonMapper(mapper);   // sync static helpers to your mapper
```
**Gotchas:** `defaultMapperBuilder()` ignores `spring.jackson.*` by design. Expose the bean as `JsonMapper` (not `ObjectMapper`) so Boot 4 doesn't add a second mapper. `@SafeQuery` is deprecated → `@SafeUrl`. Jackson 3 = `tools.jackson.*`.
**Maven:** `org.opentmf.commons:opentmf-commons` (BOM-managed).

### opentmf-json-patch
**What:** Jackson 3-native RFC 6902 (JSON Patch) + RFC 7396 (JSON Merge Patch) — replaces the Jackson-2-bound `com.github.fge:json-patch`.
**Use it for:** Building/parsing/applying patch bodies in Boot 4 / Jackson 3 code (e.g. api-clients `patch(...)`).
**Key API:** `JsonPatch.builder().add/remove/replace/move/copy/test(...).build()`, `fromJson`, `apply(JsonNode)` (returns new node); `JsonMergePatch.fromJson/apply`; `JsonPatchException`.
**Config:** none.
**Snippet:**
```java
JsonPatch p = JsonPatch.builder().replace("/lifecycleStatus", "Retired").build();
JsonNode out = p.apply(doc);   // atomic; original untouched
```
**Gotchas:** Application is atomic — any failing op throws and leaves the original unchanged. Merge-patch replaces arrays wholesale; a `null` field means remove. Jackson 3 only; package `org.opentmf.commons.patch`.
**Maven:** `org.opentmf.commons:opentmf-json-patch` (BOM-managed).

### opentmf-errors
**What:** Framework-agnostic error-code registry (`ErrorCode` contract + 59-entry `CommonErrorCode` seed catalog, wire-identical to the dnext "Back End Module Error Codes") with optional Spring renderers.
**Use it for:** Emitting stable, machine-readable error codes on the wire — as an RFC 9457 `ProblemDetail` or a TMF-style `Error` body — instead of string-building codes in business code.
**Key API:** core — `ErrorCode` (`code()`, `httpStatus()`, `reasonTemplate()`, `remedy()`, default `reason(Object...)` / `reason(Map<String,?>)`), `CommonErrorCode` enum (+ `findByCode(String)`), `ErrorTemplate.format(template, args)`; resources `org/opentmf/errors/error-codes.json` and `org/opentmf/errors/opentmf-errors-oas31.yaml` (`#/components/schemas/Error`, `#/components/schemas/ProblemDetail`). spring — `ProblemDetailRenderer.render(code, reasonArgs…)` / `renderWithDetail(code, detail, reasonArgs…)` (+ `TYPE_PREFIX`), `TmfErrorRenderer.render(code, reasonArgs…)` (+ `TYPE`), DTO `TmfError` (`code`, `reason`, `message`, `status`, `referenceError`, `errorDetail`, `@baseType`/`@schemaLocation`/`@type`).
**Config:** none — no properties, no beans, no autoconfig; renderers are static.
**Snippet:**
```java
ProblemDetail p = ProblemDetailRenderer.renderWithDetail(CommonErrorCode.NOT_FOUND, "No customer with id 42");
TmfError e = TmfErrorRenderer.render(CommonErrorCode.BUSINESS_VALIDATION, "BVR_042", "quota exceeded");
```
**Gotchas:** `opentmf-errors-core` is zero-dep (test-scope Jackson only); `opentmf-errors-spring` pulls `spring-web` + `jackson-annotations` only — no Boot autoconfiguration, you own the single `@RestControllerAdvice` mapping exceptions→`ErrorCode` (sample: `SampleGlobalExceptionMapper` in the spring test tree). Catalog is **append-only**: a published code never changes meaning or HTTP status, corrections mint a new variant. Code format is `NNN.NNN-VV` but `httpStatus()` is authoritative, not the leading segment — `500.522-98/-99` serve HTTP 400 and `200.200-99` (`DRY_RUN_SUCCESS`) is a success code in the error catalog. `ProblemDetail.type` is the dnext-generic URN `urn:dnext:error:<code>`; `TmfError` puts the same single code value on both `code` and `status`.
**Maven:** `org.opentmf.errors:opentmf-errors-core` + optional `org.opentmf.errors:opentmf-errors-spring` (BOM-managed since opentmf-versions 2.1.14, currently 1.0.0).

## TMF v4 models, API & utils

### opentmf-v4-api
**What:** Getter-only Java interfaces (`I*`, e.g. `IServiceOrder`) generated from TM Forum v4 OpenAPI specs — the contract layer, zero external deps.
**Use it for:** Depending on a lightweight TMF type contract (shared utils/clients) without pulling concrete models.
**Key API:** Interfaces only, per TMF (`org.opentmf.tmf641.model.IServiceOrder`, …). No beans/annotations/runtime behavior.
**Config:** none.
**Snippet:** `List<? extends IServiceOrderErrorMessage> errs = order.getErrorMessages();`
**Gotchas:** These are **getter-only model interfaces**, not Spring MVC controllers (the BOM README mislabels them). Depend on the specific per-TMF module(s) you need.
**Maven:** `org.opentmf.model:opentmf-{NNN|group}-v4-api` (BOM-managed).

### opentmf-v4-models
**What:** Concrete TM Forum v4 DTO classes (Lombok + Jackson polymorphism + validation), each implementing the matching `I*` interface.
**Use it for:** The actual request/response DTOs in a TMF service (e.g. `ServiceOrder`, `CancelServiceOrder`).
**Key API:** Model classes (`org.opentmf.tmf641.model.ServiceOrder`, …) + one `Tmf<NNN>JacksonConfig.registerExtensions(JsonMapper.Builder)` per module that registers its polymorphic subtypes.
**Config:** none — wire a `JsonMapper` bean and call `registerExtensions` yourself.
**Snippet:**
```java
@Bean @Primary JsonMapper jsonMapper() {
  var b = JacksonUtil.defaultMapperBuilder();
  Tmf641JacksonConfig.registerExtensions(b);     // TMF-641 + Service subtypes
  var m = b.build(); JacksonUtil.setDefaultJsonMapper(m); return m;
}
```
**Gotchas:** Polymorphic deserialization fails unless you call `registerExtensions` for **each** TMF module you use directly — subtypes aren't auto-registered.
**Maven:** `org.opentmf.model:opentmf-{NNN|group}-v4-model` (BOM-managed).

### dnext-v4-models
**What:** Concrete TMF v4 POJOs in the DNext flavor (`org.opentmf.dnext.tmfNNN.model.*`), each implementing the `opentmf-v4-api` `I*` interface.
**Use it for:** Typed DTOs for a DNext-integrated service; depend on the specific `dnext-NNN-v4-model` module(s) used.
**Key API:** Model classes (`org.opentmf.dnext.tmf622.model.ProductOrder`, …); `TmfNNNJacksonConfig.registerExtensions(JsonMapper.Builder)` for subtype registration.
**Config:** none.
**Snippet:** `var b = JacksonUtil.defaultMapperBuilder(); Tmf622JacksonConfig.registerExtensions(b);`
**Gotchas:** Maven **groupId is `org.opentmf.model`** (not `…dnext`) despite the `org.opentmf.dnext.*` package. Same per-module `registerExtensions` requirement as opentmf-v4-models.
**Maven:** `org.opentmf.model:dnext-NNN-v4-model` (BOM-managed).

### opentmf-v4-utils
**What:** Stateless helpers for any TMF v4 model implementing the `opentmf-v4-api` interfaces (characteristics, related parties, notes, product/service orders).
**Use it for:** Looking up/typing characteristics by name, finding related parties by role, reading TMF-622/641 fields without boilerplate.
**Key API:** `CharacteristicUtil` (`findCharacteristicByName`, `getMandatoryCharacteristicStringValue/BooleanValue/…`, `getOptionalCharacteristic*Value`, `toNameStringMap`); `RelatedPartyUtil` (`findRelatedPartyByRole`, `findCustomerParty`, …); `NoteUtil`; `ProductOrderUtil` (622), `ServiceOrderUtil` (641).
**Config:** none.
**Snippet:** `String imei = CharacteristicUtil.getMandatoryCharacteristicStringValue(p.getCharacteristic(), "IMEI");`
**Gotchas:** Pick the right module per scope: `opentmf-common-v4-util` / `opentmf-622-v4-util` / `opentmf-641-v4-util`. Your model must implement the `opentmf-v4-api` interfaces.
**Maven:** `org.opentmf.util:opentmf-{common|622|641}-v4-util` (BOM-managed).

## Outbound HTTP

### opentmf-api-clients
**What:** High-level TMF API client layer over opentmf-http-clients — one generic client bean per configured endpoint (CRUD + JSON/merge patch + pagination), no per-API code.
**Use it for:** Calling another TMF service's resources by config alone, instead of hand-rolling URLs over a raw RestClient.
**Key API:** Bean `GenericTmfClient` (sync; `-rest`) / `GenericReactiveTmfClient` (`-reactive`); typed clients via `TmfClientFactory.create(serverConfig, "endpoint", R.class)`; pub/sub `TmfHubClient`; pagination `TmfOffsetRequest`/`TmfPage`; errors `OpenTmfClientResponseException`/`OpenTmfClientNotFoundException`. **Bean name = `<server>.<endpoint>TmfClient`** (relaxed-bound keys, e.g. qualifier `catalogManagement.productOfferingTmfClient`).
**Config:** prefix `opentmf.api-clients.<server>.*`: `client-ref` (→ an `opentmf.http-clients` entry), `base-url`, `context-path` (default `""`), `fixed-headers`, `endpoints.<name>.{path, fixed-headers, scopes}`. Autoconfig `TmfApiClientsAutoConfiguration`.
**Snippet:**
```java
@Qualifier("catalogManagement.productOfferingTmfClient") GenericTmfClient client;
ProductOffering po = client.get("123", ProductOffering.class);
```
**Gotchas:** Unlike the raw http-clients `RestClient`, `GenericTmfClient` **auto-attaches** the bearer token (via `HeaderUtil` + the `<clientRef>TokenService`) and validates it's present — so reach for api-clients (not a hand-used `RestClient`) whenever you're calling a TMF resource and want auth handled. Endpoints whose path ends `/hub` get only a `TmfHubClient`, no generic bean. REST and reactive are separate modules — don't pull WebFlux into a servlet app. Endpoint `fixed-headers` override server-level on collision.
**Maven:** `org.opentmf.client:opentmf-api-clients-rest` (or `-reactive`/`-hub`/`-common`) (BOM-managed).

### opentmf-http-clients
**What:** Spring Boot HTTP client factory — builds configured `RestClient`/`RestTemplate`/`WebClient` beans per YAML entry, with OAuth2 bearer/basic auth, Caffeine token caching, mTLS, fixed headers, Logbook wiring.
**Use it for:** The transport under api-clients, or any standalone outbound client needing token auth from config. `client-ref` points here.
**Key API:** Per-`<id>` beans `<id>RestClient`, `<id>RestTemplate`, `<id>WebClient`, `<id>TokenService`. Utils `SyncClientUtil` (`executeWithRetry`, `emptyOn404`), `WebClientUtil`, `HttpClientUtil`. Ordering marker bean `opentmfHttpClientsStarter` (`@DependsOn`). Since 2.1.4: **`HttpClientRegistry`** bean for runtime/programmatic clients (`getOrCreate(name, ClientType, props)` / `replace` / `evict`, reactive `getOrCreateReactive`; handle exposes `restClient()`/`webClient()`/`tokenService()`); **`ClientPropertiesValidator.validate(ClientType, props)`** — static, side-effect-free, returns ALL findings at once (SRE "test connection" checklists); **`OpenTmfClientResilienceException`** for circuit-open/bulkhead-full rejections (deliberately OUTSIDE the `OpenTmfClientResponseException` hierarchy so retry utils never retry it; map to 503).
**Config:** prefix `opentmf.http-clients.<id>.*`; global `opentmf.client-type` (default `jdk`; `apache`/`netty`). Per-client `base-url`, timeouts, `num-retries` (3); auth block `bearer-auth.{token-url, client-id, client-secret, form-data:{grant_type,…}}` or `basic-auth.{username,password}` (block presence selects auth; neither = no-auth). Since 2.1.4: optional **resilience4j** per client under `<id>.resilience.*` — `enabled` (false), `circuit-breaker.{failure-rate-threshold=50, slow-call-rate-threshold=100, slow-call-duration-threshold=5s, sliding-window-size=50, minimum-number-of-calls=20, wait-duration-in-open-state=30s, permitted-calls-in-half-open=5, record-status-codes=[500,502,503,504]}`, `bulkhead.{max-concurrent-calls=0, max-wait-duration=0s}`, `time-limiter.timeout-duration` (reactive-only). Property javadoc now ships as IDE `spring-configuration-metadata`. Autoconfig `OpentmfHttpClientsAutoConfiguration`.
**Snippet:**
```java
@Qualifier("dsyncRestClient") RestClient client;
Optional<Catalog> c = SyncClientUtil.emptyOn404(() -> client.get().uri("/catalog/123").retrieve().body(Catalog.class));
```
**Gotchas:** The `<id>RestClient` / `<id>WebClient` are the raw transport — neither attaches a bearer token on its own (their only auto-headers are fixed-headers, `Accept`, gzip, Logbook). This is deliberate: attaching auth is the caller's job so a client can forward an existing/customer token (relay / on-behalf-of) instead of always minting a service token. Inject the `<id>TokenService` (sync) / `TokenService` (reactive) bean and stamp `Authorization: <getTokenType()> <getToken()>` yourself (e.g. a `requestInterceptor` via `restClient.mutate()` — which copies the interceptor list, leaving the exposed bean untouched). Only **opentmf-api-clients** (`GenericTmfClient`) attaches the token for you — so a raw client consumer that assumes auto-auth will send unauthenticated requests and get 401s. Beans are registered programmatically → IDEs flag false "cannot autowire"; add `@DependsOn("opentmfHttpClientsStarter")` if a `NoSuchBeanDefinitionException` hits at startup. Does NOT auto-retry calls (opt-in via the utils); only token retrieval retries. ⚠ **Since 2.1.6 this is finally TRUE for Apache too** — `client-type: apache` previously inherited Apache's `DefaultHttpRequestRetryStrategy`, silently retrying 429/503 once even with retries off, and STACKING on caller retries (`num-retries: 3` → up to 8 attempts on Apache vs 4 on JDK/Netty). Removed in 2.1.6: one owner, opted into at the call site via `executeWithRetry`/`WebClientUtil.retry`, all three backends identical. **If you were on `apache` and relying on the implicit retry, you now get none — opt in explicitly.** Logbook is optional and not transitive — add `logbook-spring-boot-autoconfigure` or wire-logging silently no-ops (since 2.1.2 a present starter is picked up via `ObjectProvider`, no spurious warning). Since 2.1.3 `fixed-headers` are **defaults, not overrides** on the REST side — a per-request header wins (also fixes token retrieval when `fixed-headers.Content-Type` is set). Resilience is off by default AND classpath-gated (consumer adds the resilience4j jars); one config decorates every client shape of the id INCLUDING its bearer-token calls (a broken IdP opens the same circuit); order bulkhead → breaker → time limiter, with `executeWithRetry` OUTSIDE; 4xx never trips the breaker (`emptyOn404` unaffected). Per-type semantics: `request-timeout` = connect timeout on JDK/Netty but pool-lease timeout on Apache; `max-connections`/`connection-idle-timeout` are IGNORED by the JDK client (validator flags it — use `apache` or the bulkhead). Apache clients + a `MeterRegistry` expose pool gauges `opentmf.client.pool.{leased,available,pending,max}{client=<name>}`. Registry-retired clients close after a 30s grace (JDK close is real only on Java 21+ runtimes); `replace`/`evict` reset that name's resilience instances; prefer `client-type: apache` for dynamic sources.
**Maven:** `org.opentmf.client:opentmf-http-clients-starter-rest` (or `-starter-reactive`/`-starter`, `<type>pom</type>`) (BOM-managed).

## Query, security, audit, locking

### tmf630-toolkit
**What:** Server-side Spring MVC library turning TMF-630 query params into QueryDSL `Predicate`s, plus paging/sorting/field-selection response helpers.
**Use it for:** Any read endpoint that accepts `field.op=value` filters, signed `sort`, `offset`/`limit`, `fields=` projection, or a JsonPath `filter=`.
**Key API:** `@QuerydslPredicate(root = Entity.class) Predicate` param (repo extends `QuerydslPredicateExecutor`); `@Tmf630Response` for status + `Content-Range`/`X-Total-Count` headers + field selection; `Tmf630Util.tmfPage(Page)`; enum `TmfOperator`. Since 3.0.0 (major), optional modules: `tmf630-toolkit-jpa-correlated-sort` (`Tmf630JpaCorrelatedSortExecutor` — dotted/`[k=v]`-correlated/multi-hop sorts, `min()`/`max()` wrappers, auto-wired on an `EntityManager`), `tmf630-toolkit-jsonb` (Postgres `payload jsonb` backend: `@Tmf630JsonbBacked`, filter/write executors, sub-resource controller — the escape hatch for everything JPA rejects), `tmf630-toolkit-mongo-split-collection` (`@Tmf630MongoSplitBacked`, split-aware translators); plus TMF-630 Part-4 **entity versioning**: `TmfVersionedId` path binding (`/X:(version=1.0)`), `@Tmf630Versioned(versionOrder = LEX|NUMERIC_STRING|SEMVER)`, `Tmf630VersionResolver` (JPA/Mongo/Jsonb impls).
**Operator suffixes (exact):** `eq, ne, eqi, nei, gt, gte, lt, lte, between, in, nin, isnull, isnotnull, like, likei, contains, containsi, startswith, startswithi, endswith, endswithi, regex, regexi`. (`*i` = case-insensitive; `in/nin/between` multi-value; `isnull/isnotnull` valueless.) Sort: `-field` = descending.
**Config:** `opentmf.tmf630.paging.*` (`default-limit=50`, `max-limit=500`, `strict-mode=true`); `…field-selection.default-depth=1`; `…attribute-filtering.*` (`implicit-eq-enabled=true`, `on-unknown-field=REJECT`, `regex.enabled=false`, `json-path-filter.enabled=true`). Autoconfigs `Tmf630WebMvcConfigurer`, `Tmf630AttributeFilteringAutoConfiguration`, `Tmf630FieldSelectionAutoConfiguration`.
**Snippet:**
```java
@GetMapping("/persons") @Tmf630Response
Page<Person> list(@QuerydslPredicate(root = Person.class) Predicate p, Pageable pageable) {
  return repo.findAll(p, pageable);
} // GET /persons?surname.startswith=D&born.gte=1990&sort=-surname&limit=20&fields=id,surname
```
**Gotchas:** Use `tmf630-toolkit-all` to get both filtering autoconfigs. Unknown field/operator → 400 by default (`on-unknown-field=REJECT`). `default-depth=1`: `fields=address` expands only scalar sub-fields — use a dot-path or raise depth for associations. **3.0.0 breaking:** `.regex`/`.regexi` (and JSONPath `=~`) on a JPA `@Entity` now 400 at parse time instead of silently degrading to SQL `LIKE`; deprecated escape hatch `opentmf.tmf630.attribute-filtering.regex.allow-jpa-like-semantics=true` restores old behavior with a one-time WARN (jsonb module supports real regex). New `opentmf.tmf630.paging.nulls-last=true` decorates sorts with `nullsLast()` (synthetic `CASE WHEN` cost on non-native dialects). Sub-resource base controllers now carry `@Tmf630Response` themselves — subclasses get headers/`fields=` without annotating. JPA `filter=` got stronger: array correlation via correlated `EXISTS` on JOIN-mapped collections, positional `[N]` via `INDEX(alias)` (requires `@OrderColumn`); JSON-column collections still 400. Versioning `LEX` order trap: `"1.10" < "1.9"` — use `NUMERIC_STRING`/`SEMVER` (which sort in-JVM, O(N) per logical id). Mongo: OR-of-splits needs `$unionWith` — illegal inside a multi-document transaction. **3.0.1:** `.regex`/`.regexi` and the LIKE family now accept polymorphic (`Object`/`Serializable`) value fields on the **Mongo and JSONB** backends (TMF-620-style `productSpecCharacteristicValue.value`) — no effect on plain JPA entities.
**Maven:** `org.opentmf.query:tmf630-toolkit-all` (+ optional `tmf630-toolkit-{jpa-correlated-sort,jsonb,mongo-split-collection}`) (BOM-managed).

### openid-rbac-security
**What:** Spring Boot autoconfig for OAuth2 Resource Server (Bearer/JWT) RBAC with declarative, property-driven endpoint rules (servlet + reactive).
**Use it for:** Securing a service with JWT validation + per-method/path role rules expressed in config, not a hand-written `SecurityFilterChain`.
**Key API:** Fully autoconfigured — set properties only. Types `OpenTmfSecurityProperties`, `SecureEndpoint`, `OtherEndpoints` enum (`ALLOW`/`DENY`/`AUTHENTICATED`). Since **2.3.0: MULTI-ISSUER** — `opentmf.security.issuers[]` (each entry: `name`, `issuer`, `jwk-set-uri`, optional `audiences`, optional claim overrides); tokens route to the entry matching their `iss`, and `JwtDecoder`/`ReactiveJwtDecoder` route by issuer so `JwtService` and out-of-filter decoding keep working. Since 2.2.0: **custom 401/403 bodies via optional consumer beans** — servlet `AuthenticationEntryPoint` (401) / `AccessDeniedHandler` (403), reactive `ServerAuthenticationEntryPoint` / `ServerAccessDeniedHandler`; bean presence is the switch (no new properties), each independent, applied to BOTH the bearer-token path and the exception-translation path; no bean → RFC 6750 defaults unchanged.
**Config:** prefix `opentmf.security`: `jwk-set-uri` (required), `user-claim` (default `sub`), `authorities-claim` (default `roles`), `secure-endpoints[]`, `whitelist[]` (any method), `blacklist[]`, `other-endpoints` (default `DENY`). Separate `opentmf.security.management.*` mirror for the management port (its `whitelist` defaults to actuator health/info; `other-endpoints` defaults `AUTHENTICATED`). Autoconfigs `ServletSecurityAutoConfiguration`, `ServletJwtAutoConfiguration` (+ reactive).
**Snippet:**
```yaml
opentmf.security:
  jwk-set-uri: http://idp/realms/r/protocol/openid-connect/certs
  whitelist: [/actuator/health, /v3/api-docs/**]
  secure-endpoints:
    - { method: POST, path: /orders/**, roles: [order-writer] }
```
**Gotchas:** Main-port unmatched paths default to `DENY` (403); management-port unmatched default to `AUTHENTICATED` — asymmetric on purpose. Eval order: blacklist → whitelist → allowed-endpoints → secure-endpoints → other-endpoints. Management block applies only when `management.server.port` ≠ `server.port` — **and on servlet it actually works only from 2.2.0** (in 2.1.x Boot exposed the parent filter chain in the management child context, so main-port rules governed the management port; reactive was never affected). Unknown props fail fast. Custom 401/403 beans (2.2.0): apply to the MAIN port only; multiple candidate beans of one type → default kept + WARN naming candidates, disambiguate with `@Primary`; `@PreAuthorize` denials still reach `@RestControllerAdvice` as before. Behavior matrix surprises: a present-but-invalid token on a `permitAll` URL still → 401; an anonymous request on a `denyAll`/blacklisted URL → **401, not 403** (anonymous denials translate to authentication failures). Reactive trap: re-emitting the exception via `Mono.error` into Boot's `ErrorWebExceptionHandler` yields 500, not 401/403 — render the body in the bean (shared-renderer pattern) instead. **Multi-issuer (2.3.0):** configure EXACTLY ONE of `jwk-set-uri` or `issuers` — both, or neither, fails startup by design (as do duplicate `issuer`/`name` values). There is **no fallback issuer**: a token whose `iss` matches nothing, or that carries no `iss`, is 401 — deliberate. Single-issuer mode still does NOT check `iss`, so adding `issuers` is what turns issuer validation on. ⚠ **Set `audiences` when the provider mints tokens for many apps from one tenant (Entra especially)** — otherwise a token issued to a *different application of the same tenant* passes issuer validation. Entra field traps (README): tenant- and version-specific issuer values, prefer **App Roles** over group claims (which truncate), and `oid` is the stable principal while `preferred_username`/`upn` are readable but mutable. Entries inherit top-level `user-claim`/`fallback-user-claims`/`authorities-claim` unless they override — normalize onto ONE role vocabulary so `secure-endpoints` never forks per provider.
**Maven:** `org.opentmf.security:openid-rbac-security` (BOM-managed).

### auditor-aware
**What:** Persistence-agnostic Spring Data `AuditorAware<String>` + `DateTimeProvider` beans wired to Spring Security (servlet + reactive).
**Use it for:** Populating `@CreatedBy`/`@LastModifiedBy` (username) and `@CreatedDate`/`@LastModifiedDate`.
**Key API:** beans `auditorAware` (returns `Authentication.getName()` or `"n/a"`) and `auditorAwareDateTimeProvider`; helpers `OffsetDateTimeProvider`, `ServletAuditorAwareProvider`.
**Config:** none. Autoconfigs `ServletAuditorAwareAutoConfiguration`, `ReactiveAuditorAwareAutoConfiguration`.
**Snippet:** `@EnableJpaAuditing(dateTimeProviderRef = "auditorAwareDateTimeProvider")` + `@CreatedBy String createdBy;`
**Gotchas:** Servlet autoconfig is `@ConditionalOnWebApplication(SERVLET)` — it does **not** fire under `@DataJpaTest`; supply explicit `OffsetDateTimeProvider` + `ServletAuditorAwareProvider` test beans (and `@WithMockUser` for `@CreatedBy`). Both beans are `@ConditionalOnMissingBean`. Replaces the older `auditor-aware-jpa`.
**Maven:** `org.opentmf.util:auditor-aware` (BOM-managed).

### opentmf-db-lock-service
**What:** Cluster-level distributed lock over the app's own JDBC datasource (pure JDBC), with versioned upgrade/downgrade semantics and an auto-release safety timeout.
**Use it for:** Serializing once-per-cluster work (BPMN deployment, catalog sync, adapter version-gating), gated on a requested version.
**Key API:** `@UsingClusterLock(lockType=, requestedVersion=, downgradeAllowedAfter="PT10M", failureMessage=)` on a method; a `LockContext` method param the aspect enriches (`context.setSuccess(false)` to release without recording the version); `DbLockService.acquireLock(LockType, version)`; enum `LockType` (`BPMN`=B, `CATALOG`=C, `LOCK_X/Y/Z`); `DbLockException`.
**Config:** prefix `opentmf.db-lock` (strict): `create-tables=true`, `lock-acquire-timeout=120000`, `lock-hold-timeout=300000`, per-type `duration-overrides`. Autoconfig `DbLockAutoConfiguration` (activates on `spring.datasource.url`). Tables `DB_LOCK`/`DB_LOCK_LATEST`/`DB_LOCK_HISTORY` (`lock_type CHAR(1)`).
**Snippet:**
```java
@UsingClusterLock(lockType = LockType.BPMN, requestedVersion = "1.3", failureMessage = "BPMN deploy failed")
void deploy(LockContext ctx) { /* runs once cluster-wide */ }
```
**Gotchas:** **NEVER pass `null` for the `LockContext` param — pass `new LockContext()`** so the aspect can enrich it. Tables are per-microservice (each service owns its own `db_lock_*`). A held lock auto-releases after `lock-hold-timeout`. Any thrown exception releases with `success=false`. DDL ships for Postgres/MySQL/MariaDB/Oracle/SQL Server/DB2/H2; dialect auto-detected.
**Maven:** `org.opentmf.util:opentmf-db-lock-service` (BOM-managed).

## Workflow (Camunda 7 and compatible implementations)

### opentmf-camunda7
**What:** A deployable microservice embedding a Camunda-7-compatible workflow engine + its Cockpit/Tasklist/Admin web UIs, secured with OpenTMF OIDC/RBAC + Keycloak.
**Use it for:** When you need to **run** a workflow-engine server (REST + web UIs) for dsync engine/adapters to talk to — not a Java dependency.
**Key API (endpoints):** engine REST `/engine-rest`; web apps `/app`; actuator on its own port. OAuth2/SSO via `CamundaSpringSecurityOAuth2AutoConfiguration`. Bundles incident-logger + Spin (workflow vars >4KB).
**Config:** server port `8080`, context-path `/camunda/v7`; management port `16000`; `camunda.bpm.oauth2.sso-logout.*`; Keycloak `plugin.identity.keycloak.*`; RBAC `opentmf.security.*`. Configure via env vars.
**Snippet:** N/A — run the published image; reach `http://host:8080/camunda/v7/engine-rest/...`.
**Gotchas:** Despite `jar` packaging it is a **service**, not a library (fat jar only under `repackage`/`docker` profiles). Not BOM-managed (uses Spring Boot 3.5.x, Java 17). A `-aws` image variant adds RDS/Aurora + MSK IAM.
**Maven:** `org.opentmf.camunda:opentmf-camunda7` — deployable service, not a dependency.

### camunda7-bpmn-sync-service
**What:** A Spring Boot autoconfig **library** that on startup deploys the app's bundled BPMN files to the engine and optionally auto-migrates running instances, under a DB cluster lock.
**Use it for:** A service that owns BPMN under `src/main/resources/bpmn/` and must keep the engine's deployed definitions in sync on boot.
**Key API:** Autoconfig `BpmnSyncAutoConfiguration` exports `BpmnSyncService` and runs `ensureBpmnConsistency()` at startup; `BpmnMigrationService`; `ResourceUtil.getBpmnFiles()`.
**Config:** prefix `opentmf.bpmn-sync`: `enabled` (default `true`), `deployment-name` (required, gates activation), `bpmn-version` (required, **string-compared**), `auto-migrate` (default `false`), `downgrade-allowed-after` (600000 ms), `client-ref` (required); since 2.1.1 `resource-location` (default `classpath:bpmn/`) — ONE folder scanned recursively for both `*.bpmn` AND `*.dmn` (sub-folder structure preserved in resource names; no separate `classpath:dmn/` scan anymore); optional `tenant-id` for tenant-scoped deployments/migration. Plus `camunda.bpm.client.base-url`. Activates only when `deployment-name` is set and a `dbLockService` bean exists.
**Snippet:**
```yaml
opentmf.bpmn-sync: { deployment-name: dsync-adapter, bpmn-version: "1.2", client-ref: engine, auto-migrate: true }
```
**Gotchas:** Despite the name it's a **library**. `bpmn-version` is string-compared, so `"1.9" > "1.10"` — bump it whenever any BPMN **or DMN** changes or sync is skipped; zero-pad. The single version governs the whole BPMN+DMN bundle; DMNs deploy in the same Camunda deployment (`CamundaDeploymentResponse.deployedDecisionDefinitions`) and are never auto-migrated. The `resource-location` folder **must exist** or startup fails with an error naming the property. Needs `spring.data.jdbc.repositories.enabled=false`; disable in ITs via `opentmf.bpmn-sync.enabled=false`.
**Maven:** `org.opentmf.camunda:camunda7-bpmn-sync-service` (BOM-managed downstream).

### camunda7-incident-logger
**What:** A zero-config engine plugin that logs a WARN line whenever the engine raises an incident (retries exhausted, instance hung).
**Use it for:** Surfacing hung process instances in logs with deployment / process-definition / activity / failure-message — drop-in, nothing to call.
**Key API:** `IncidentLoggerPlugin` (auto-registered) installs `IncidentLogger` for failed-job and external-task handlers.
**Config:** none — auto-wires via the autoconfig imports file. Only requirement: `org.opentmf.camunda` log level ≥ WARN.
**Snippet:** N/A — adding the dependency is the entire setup.
**Gotchas:** None known; logging never disrupts the engine (any `Throwable` is caught).
**Maven:** `org.opentmf.camunda:camunda7-incident-logger` (BOM-managed downstream).

### camunda7-test-framework
**What:** A test-scoped library for integration-testing BPMN flows against an embedded engine, scripting each task with lambdas/variables instead of real workers.
**Use it for:** Driving a full BPMN flow in an IT — stub service tasks, satisfy message-catch/receive, set vars, assert ended/waiting/incident.
**Key API:** Base classes `BaseBpmIT` (`startProcessInstance`, `validateDeployment`, `assertProcessEnded`, `assertProcessWaiting`, `assertIncidentCreated`) and `BaseBpmUnitTest`; `@EnableBpmnTaskListenerPlugin`; `CamundaExpectationUtil.registerTaskExecutionListener()/registerMessageCatchExecutionListener()` fluent builders (`withTaskId`, `withVariableMap`, `withRunnable`, `withCorrelationMessage`, `create`); enum `EventType{START,END}`.
**Config:** none — activate via `@EnableBpmnTaskListenerPlugin` or by extending `BaseBpmIT`. README recommends `@ActiveProfiles({"it","camunda"})`.
**Snippet:**
```java
class FlowIT extends BaseBpmIT {
  registerTaskExecutionListener().withTaskId("t1").withVariableMap(vars).create();
  assertProcessEnded(startProcessInstance("dsync_engine_flow", start));
}
```
**Gotchas:** `EventType` has only `START`/`END`. `assertIncidentCreated` polls up to 120s. Embedded engine is test-scoped (no runtime classpath impact).
**Maven:** `org.opentmf.camunda:camunda7-test-framework` (test scope; BOM-managed downstream).

## Boot-time sync starters

### dnext-catalog-sync-service
**What:** A Spring Boot autoconfig **library** that on startup version-checks and pushes a service's bundled resource/service/product catalogs into the DNext catalog backends under a DB cluster lock.
**Use it for:** A DNext-integrated service that ships versioned catalog files and must idempotently sync them on boot. Add as a normal dependency — it self-activates.
**Key API:** Autoconfig `CatalogSyncAutoConfiguration` registers `CatalogSyncService` and runs `ensureCatalogConsistency(LockContext)` at startup. Auto-selects reactive (`WebClient`) vs sync (`RestClient`) by which `<clientRef>…Client` bean exists.
**Config:** prefix `opentmf.catalog-sync`: `enabled` (true), `catalog-version` (required, gates activation), `client-ref` (required → an `opentmf.http-clients` entry), `downgrade-allowed-after` (PT10M), `product-/resource-/service-catalog-url`.
**Snippet:** `opentmf.catalog-sync: { catalog-version: "1.2", client-ref: dnextBackend }`
**Gotchas:** Activates only if a `dbLockService` bean exists **and** `catalog-version` is set. Version compare is `String.compareTo` — zero-pad. Sync runs synchronously during startup (blocks boot). A library, not a standalone service, despite the name.
**Maven:** `org.opentmf.dnext:dnext-catalog-sync-service` (BOM-managed).

## Testing & ops tools

### opentmf-mockserver
**What:** A standalone TMF-630-compliant dynamic HTTP mock server (MockServer fork, Jackson 3) with a built-in Keycloak/OIDC mock that issues real RSA-signed JWTs.
**Use it for:** ITs (or local dev) that need to fake TMF REST backends and/or an OAuth2 token provider without a real Keycloak; in-process usage now goes through **opentmf-mockserver-test-support** (next entry) rather than raw `ClientAndServer` + expectation JSON; or run as a Docker image.
**Key API:** Register expectations pointing at callbacks `org.opentmf.mockserver.callback.Dynamic{Post,Get,GetList,Put,Delete,JsonPatch,MergePatch,JsonPatchCollection}Callback`; OIDC auto-registered. Control plane `PUT /mockserver/expectation`; token `POST /realms/{realm}/protocol/openid-connect/token`; JWKS `/realms/{realm}/…/certs` (default realm `realm1`).
**Config:** env vars — `SERVER_PORT` (1080), `ENFORCE_TOKEN` (false), `TOKEN_ISSUER`, `JWKS_URI`, `MOCKSERVER_INITIALIZATION_JSON_PATH`. Auto-manages `id`/`href`/`createdDate` + TMF paging headers.
**Snippet:** `PUT localhost:1080/mockserver/expectation` with `httpResponseClassCallback.callbackClass = …DynamicPostCallback`.
**Gotchas:** Jackson 3 only; XML/dashboard/proxy/templating stripped out. First GET on an order **auto-transitions** its state field (e.g. `acknowledged`→`completed`) — a read mutates observable state. `ENFORCE_TOKEN=true` enforces the reader/writer/admin role matrix. Pin the Docker image at **2.1.11+**: 2.1.9 never published (broken release Dockerfile), and **2.1.10 published to the WRONG GHCR coordinate** — the repo rename to `opentmf-mockserver-parent` leaked into the image name, so it landed at `ghcr.io/opentmf/opentmf-mockserver-parent:2.1.10` and broke consumers pinning the historical path. 2.1.11 hard-codes `opentmf/opentmf-mockserver`.
**Maven:** `org.opentmf.mockserver:opentmf-mockserver` (BOM-managed; repo is now multi-module under `opentmf-mockserver-parent`, artifact coordinates unchanged). Also a Docker image.

### opentmf-mockserver-test-support
**What:** Fluent JUnit 5 test-support layer over `opentmf-mockserver` (new sibling artifact in the 2.1.9 multi-module split): starts an in-process MockServer on a random free port, registers the server's `Dynamic*Callback`s, mints RSA-signed Keycloak-shaped JWTs, and redirects Spring properties at the mock.
**Use it for:** Replacing hand-rolled `ClientAndServer` + `Dynamic*Callback` + JWKS + `@DynamicPropertySource` IT glue. Works for plain JUnit, Spring Boot ITs, and non-JUnit harnesses (Cucumber/E2E) via `start()`/`stop()`.
**Key API:** Package `org.opentmf.mockserver.testsupport`. `MockServerSupport` (`create()` per-class / `shared()` JVM-singleton; `start/stop`, `port()`, `baseUrl()`, `client()`, `keepExpectationsBetweenTests()`, `expect(HttpRequest,HttpResponse)`, `clear(Registration...)`, `redirectApiClients(registry, ids…)`, `redirectHttpClients(registry, ids…)`, `redirectJwks(registry[, realm])`, `token/tokenFor/bearerHeader`) — implements `BeforeAllCallback`/`AfterAllCallback`/`AfterEachCallback`. Builders: `tmf(apiClientId)` → `TmfMockBuilder.post/get/getList/put/delete/jsonPatch/mergePatch/jsonPatchCollection/crud(path)`; `stub()` → `StubBuilder.get/post/put/delete/method` + `limit/pathParam/queryParam/header/jsonBody` → `respondJson/respondStatus/respondDelayed/respondSequence`; `verify()` → `VerifyBuilder.get/post/put/delete/method` + `withHeader/withQueryParam/withJsonBody` → `times/atLeast/atMost/never/once`; `Registration.id()/ids()/clear()`; `OidcMockSupport.realm/clientId/expiresInSeconds/token/tokenFor/jwksPath/jwksUri/jwksJson/registerJwksEndpoint` (`DEFAULT_REALM="realm1"`, `DEFAULT_CLIENT_ID="opentmf-mockserver"`, `DEFAULT_EXPIRES_IN_SECONDS=3600`).
**Config:** No properties of its own — it *writes* them: `opentmf.api-clients.<id>.base-url` (+ `context-path` → `""`), `opentmf.http-clients.<id>.base-url`, `opentmf.security.jwk-set-uri` → `<baseUrl>/realms/<realm>/protocol/openid-connect/certs`. Server behavior/env vars are `opentmf-mockserver`'s.
**Snippet:**
```java
@SpringBootTest
class DocumentServiceIT {
  @RegisterExtension static MockServerSupport mock = MockServerSupport.shared();

  @DynamicPropertySource
  static void redirect(DynamicPropertyRegistry r) {
    mock.redirectApiClients(r, "onedms");
    mock.redirectJwks(r);
  }

  @Test void archives() {
    mock.tmf("onedms").crud("/document");
    mock.stub().get("/kba/{key}").pathParam("key", "42").respondJson(200, "{\"v\":42}");
    // exercise service, e.g. with header Authorization = mock.bearerHeader("writer")
    mock.verify().post("/document").once();
  }
}
```
**Gotchas:** `afterEach` calls `server.reset()` — it wipes **all** expectations, including the JWKS endpoint `redirectJwks` registers once at property-resolution time and anything registered in `@BeforeAll`; register per `@Test` or call `keepExpectationsBetweenTests()`. The constructor is private: only `create()`/`shared()`; `shared()`'s reset is server-wide, so it is unsafe with test classes running in parallel (use `create()`). `limit(N)` applies to the next `respond*` only and is a no-op with `respondSequence(...)` (which pins one call per element); calling a matcher or `respond*` before a verb-starter throws `IllegalStateException`. `tmf(...).get/put/delete/jsonPatch/mergePatch` append the regex `/[^/]+` to the path, and `crud(path)` omits `jsonPatchCollection`; the `apiClientId` argument is only a label and does not affect the URL. JUnit, `spring-test`, and `spring-boot-test` are `provided`+`optional` — non-Spring consumers get the rest of the module but must not call `redirect*`.
**Maven:** `org.opentmf.mockserver:opentmf-mockserver-test-support` (test scope; BOM-managed since opentmf-versions 2.1.14).

### http-endpoint-kicker
**What:** A tiny Docker image that optionally fetches an OAuth2 token, makes exactly one configurable HTTP request, and exits 0 on 2xx.
**Use it for:** Kicking a single endpoint from a CronJob / CI / init container (e.g. a scheduled `/processing/process`). Not a library, not a Spring app.
**Key API:** OCI image `ghcr.io/opentmf/http-endpoint-kicker`, driven by env vars; built-in token script `/app/get-token-default.sh` (client_credentials + password grants).
**Config:** `TRIGGER_URL` (required), `TRIGGER_METHOD` (POST), `TRIGGER_BODY`/`_FILE`, `TRIGGER_CONTENT_TYPE`; auth via `BASIC_AUTH_*` or `TOKEN_SCRIPT_PATH` + `OIDC_TOKEN_URL`/`OIDC_CLIENT_ID`/`OIDC_CLIENT_SECRET`/`OIDC_GRANT_TYPE`/`OIDC_SCOPE`.
**Snippet:** `docker run --rm -e TRIGGER_URL=https://svc/processing/process -e TOKEN_SCRIPT_PATH=/app/get-token-default.sh -e OIDC_TOKEN_URL=… ghcr.io/opentmf/http-endpoint-kicker:1.0.0`
**Gotchas:** One request per run, no retries; success is strictly 2xx. A custom token script must print only the raw token to stdout. `TRIGGER_BODY_FILE` wins over `TRIGGER_BODY`.
**Maven:** Docker image, not a Maven dependency.
