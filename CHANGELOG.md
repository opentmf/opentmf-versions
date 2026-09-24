# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.30] - 2026-09-24

Only one managed version moves (no BOM structure change), so this is a patch of the BOM as an
artifact.

| Library | From → To | Wire-visible effect |
|---|---|---|
| `tmf630-toolkit` | 3.3.0 → **3.4.0** | **None for a service that has not enabled regex** (`opentmf.tmf630.attribute-filtering.regex.enabled` still defaults to `false`, and that gate runs first on every backend). With regex enabled: `.regex`/`.regexi`/Part 6 `=~` on a JPA `@Entity` root answers `200` with the same rows as Mongo/JSONB for the `LIKE`-expressible subset, and `400` naming the subset for anything else (was `400` for every pattern, or a `500` under the compat flag for `^`, `[...]`, `\d`); JSONB `filter=` with `=~` answers rows instead of a `500`. |

### Updated
- Updated `tmf630-toolkit` to **3.4.0** (from 3.3.0). **Regex on JPA roots renders the
  `LIKE`-expressible subset and rejects the rest.** A new translator in `PredicateFactory` (used by
  attribute-side `.regex`/`.regexi`, the Part 1 `%3D~` form and the Part 6 `=~` literal alike)
  renders literal characters (with `%`, `_` and the escape character escaped so they match
  themselves), `\`-escaped metacharacters, `.` → `_`, `.*` and `.*?` → `%`, a leading `^` and a
  trailing `$` as anchors, and the `i` flag as `lower(column) LIKE lower(pattern)`; an unanchored
  side is padded with `%`, so `name.regex=p` means *contains* and `/^p$/` means *equals*, as the
  regex does. Anything outside the subset (`+`, `?` other than in `.*?`, `*` not after `.`, `[ ]`,
  `( )`, `{ }`, `|`, `\d`/`\w`/`\s`/back-references, `^`/`$` away from the ends) is a `400` whose
  message names the supported subset and the JSONB/Mongo escape. Mongo and JSONB keep real regex;
  inside the subset all three backends return identical rows. **Adopter note:** the bump changes
  nothing for a service that has not set `regex.enabled: true` — a test pinning `.regex` →
  `400 "Regex operator is disabled."` stays green; a test that pinned the pre-3.4.0 JPA refusal is
  re-pinned in the same change that enables the operator. **Deprecated:**
  `opentmf.tmf630.attribute-filtering.regex.allow-jpa-like-semantics` is inert (still binds, logs
  one warning asking for its removal; removal is a 4.0.0 item). **Fixed:** `=~` inside a JSONB
  `filter=` now lowers to Postgres `like_regex` (plus `flag "i"`) instead of failing with a
  `BadSqlGrammarException` `500`, gated by `regex.enabled` and accepting only the `i` flag
  (`JsonbJsonPathTranslator` gained a third constructor argument `regexEnabled`; the two-argument
  constructor keeps regex disabled). **Fixed:** under the compat flag (3.0.0 – 3.3.0), a pattern
  with `^`, `[`, `]` or a class escape reached querydsl-jpa's `regexToLike` at JPQL serialization
  and threw an unmapped `QueryException` — a `500` by default; every such pattern is now a `400` at
  parse time, and `%`/`_` typed by the caller match literally on every backend instead of acting
  as SQL wildcards on JPA. **Fixed:** `tmf630-toolkit-attribute-filtering-autoconfigure` ships
  `spring-configuration-metadata.json` again (no release up to 3.3.0 carried metadata for the 20
  `opentmf.tmf630.attribute-filtering.*` properties).

## [2.1.29] - 2026-09-21

Only managed versions move (no BOM structure change), so this is a patch of the BOM as an
artifact, as every release since 2.1.14 — including the ones that carried a library major.

| Library | From → To | Wire-visible effect |
|---|---|---|
| `tmf630-toolkit` | 3.2.2 → **3.3.0** | The paging `Link` header is **omitted** over a size budget (any query-parameter value > 256 chars, or an assembled header > 2048 chars); adopters and their clients must treat `Link` as optional and page by `offset`/`limit` against `X-Total-Count`. A query-parameter value > 2048 chars answers **400** and a query string > 4096 chars answers **414**, before any handler, as TMF `ErrorMessage` bodies. |
| `opentmf-http-clients` | 2.1.8 → **2.2.0** | A bearer token mint is retried once on a transport failure; a mint that still fails throws `BearerTokenTransportException` (was the raw `ResourceAccessException`/reactor error); new counter `opentmf.client.token.fetch{client, outcome}` and one `INFO` per mint. |
| `openid-rbac-security` | 3.2.1 → **3.3.0** | 3.2.2: the typed `503` carries `Retry-After` exactly once, whichever resolver renders it (servlet). 3.3.0: opt-in `jwks` health contributor per issuer (`opentmf.security.jwks.readiness: true`, default off) that joins the `readiness` group so a pod whose keys are unavailable goes NotReady instead of answering 503s, and a NotReady probe triggers one paced background JWKS retry; always-on `opentmf.security.jwks.*` meters when Micrometer is present; the boot line names the JWKS host and route (proxy/direct). |
| `opentmf-outbox-service` | 1.2.0 → **1.2.1** | A consumer **without `spring-kafka`** on the classpath now starts (1.0.0–1.2.0 failed at context start with `NoClassDefFoundError: org/springframework/kafka/core/KafkaTemplate`) and gets the HTTP publisher only; a web-less consumer starts with the Kafka one only. No behaviour change for a consumer that has both; the publisher configurations are not scan candidates, so a consumer whose scan root covers `org.opentmf.outbox` is safe. |

### Updated
- Updated `tmf630-toolkit` to **3.3.0** (from 3.2.2). Fixes a `500` with an empty body and
  `Connection: close` on any paged endpoint given one long query parameter (≥ ~2100 chars): the
  pagination `Link` header echoed the request's full query string in each of its four links, past
  Tomcat's 8 KB response-header buffer, and the application's catch-all handler then failed the
  same way (found by a DAST active scan on 2026-09-17; every adopter with a paged list endpoint was
  exposed). Three layers close it. **Bounded `Link` header:** when any single query-parameter value
  is longer than `opentmf.tmf630.paging.link.max-param-value-length` (default 256), or the
  assembled header would exceed `opentmf.tmf630.paging.link.max-length` (default 2048), the header
  is omitted — never truncated, because a `next` that drops a filter walks a different result set;
  `X-Total-Count`, `X-Result-Count`, `Content-Range` and the status are unchanged, and clients must
  treat `Link` as the SHOULD it is in TMF-630 Part 1 §4.5.1 (new `Tmf630LinkHeaderSettings` record,
  `Tmf630Util.applyLinkHeader(...)` overload; existing overload and `tmfPage(...)` use the
  defaults). **Query-parameter guard** (`opentmf.tmf630.query-limits.*`, `enabled=true`,
  independent of `paging.enabled`): `Tmf630QueryLimitsInterceptor` on every mapping answers **400**
  for a single value longer than `max-param-value-length` (2048) and **414 URI Too Long** for a
  query string longer than `max-query-string-length` (4096), as TMF `ErrorMessage` bodies via
  `Tmf630QueryLimitExceptionHandler`; the defaults reject nothing the filtering module accepts
  today. **`HeadersTooLargeException` recovery:** `Tmf630HeadersTooLargeRecoveryResolver`
  recognises Tomcat's overflow, logs which header overflowed, resets the uncommitted response and
  answers one chunked `500` TMF error body (covers adopter-added headers and raised budgets).
- Updated `opentmf-http-clients` to **2.2.0** (from 2.1.8). Bearer token mints — sync and reactive —
  retry exactly once, immediately, with one `WARN`, on a transport-level failure under the token
  `POST` (a keep-alive connection reused after the peer closed it, a reset, a premature EOF, an I/O
  timeout at the headers or body stage); the JDK HttpClient's own stale-connection retry covers
  `GET`/`HEAD` only. Status errors from the token endpoint and open circuit breakers are not
  retried. **Changed:** a mint that fails after the retry throws `BearerTokenTransportException`
  (`org.opentmf.client.bearer.exception`: `tokenUrl`, `attempts`, cause) instead of the raw
  `RestClientException`/`ResourceAccessException` (sync) or reactor error — not a response
  exception, so the retry utilities never retry it; map it to 503. One `INFO` line per successful
  mint (issuer URL, scope, attempt, duration) and, with a `MeterRegistry` bean, the counter
  `opentmf.client.token.fetch{client=<id>, outcome=ok|retried|failed}` (Prometheus
  `opentmf_client_token_fetch_total`); `TokenFetchListener` / `TokenFetchMeters` for hand-wired
  token clients; `RestClientRegistrar.createTokenService(String, RestClient, ClientProperties)`
  overload (the 2-arg form opts out of the counter). README corrected: only the reactive token
  client retries retryable statuses per `num-retries`. Spring Boot 4.1.1; build tooling bumps.
- Updated `openid-rbac-security` to **3.3.0** (from 3.2.1 — 3.2.2 and 3.3.0 in one step; both
  additive, nothing changes without a decision). **3.3.0 — `jwks` health, behind an opt-in.**
  `opentmf.security.jwks.readiness: true` (default `false`) registers a `jwks` health contributor
  with one component per issuer, read from the same key source the wire answers from: `UP` when a
  set is loaded and fresh; `UP` with `stale: true`, the age and the last failure when refreshes fail
  but the cached set still serves (readiness must not flip while tokens still validate); `DOWN`
  with the issuer, the last failure and since when, exactly when every bearer request from that
  issuer answers `503` (never loaded, or older than the outage TTL). It joins the `readiness` health
  group when Kubernetes probes are enabled (Boot's default), so the pod goes NotReady instead of
  serving `503`s, without touching `management.endpoint.health.group.readiness.include`; Boot's
  `management.health.jwks.enabled` still switches it off. A readiness probe that finds the keys
  unavailable also retries the load in the background (one attempt in flight, paced by the refresh
  interval), so a pod heals while NotReady; the probe never waits on the network. **Metrics, always
  on when Micrometer is present**, per issuer (tag `issuer`): `opentmf.security.jwks.keys`,
  `opentmf.security.jwks.keys.age` (seconds, `NaN` before the first load),
  `opentmf.security.jwks.fetch.failures` (Prometheus `opentmf_security_jwks_keys`,
  `opentmf_security_jwks_keys_age_seconds`, `opentmf_security_jwks_fetch_failures_total`). The boot
  line names the JWKS host and route — `… loaded (2 keys) from idp.example (via proxy
  10.0.0.1:3128)` / `… could not be loaded from … (direct): …` — host only, never a path or query,
  and a URL inside a Nimbus message is reduced to its host in the log and the health details; the
  `503` body still names the issuer alone. `spring-boot-health` and `micrometer-core` are optional
  dependencies; each half of `JwksObservabilityAutoConfiguration` is conditional on its own classes.
  **3.2.2:** the typed `503` shipped `Retry-After`
  twice (`30, 30`) in 3.2.1 when nothing in the application handled `ErrorResponseException` and it
  fell to Spring's default resolver, which copies the exception's headers with `addHeader` and then
  `sendError`s (after which the 3.2.1 collapse-after-rendering never ran). The resolvers now see a
  response on which re-adding a value the header already carries is a no-op, so the wire carries
  each value exactly once whoever renders — a mapper that rebuilds the response without headers,
  Spring's default resolver, the bare fallback, or a resolver that sets its own different value
  (kept). Servlet only.
- Updated `opentmf-outbox-service` to **1.2.1** (from 1.2.0). **Fixed:** a consumer without
  `spring-kafka` on the classpath could not start. `OutboxAutoConfiguration` named `KafkaTemplate` in
  a bean-method signature; the method-level `@ConditionalOnClass` did skip the bean, but Spring
  still introspects the auto-configuration class reflectively to resolve its other factory
  methods, and the missing type threw `NoClassDefFoundError: org/springframework/kafka/core/KafkaTemplate`
  before any context came up (found on the first Kafka-less consumer; consumers that carry Kafka
  were never affected, which is why 1.0.0–1.2.0 never hit it). The Kafka publisher now lives in a
  nested, name-guarded member class (`KafkaPublisherConfiguration`), so a Kafka-less consumer
  starts with the HTTP publisher only and nothing Kafka-typed is linked. **Changed, same reason:**
  the HTTP publisher's guard is name-based and nested too (`HttpPublisherConfiguration`,
  `@ConditionalOnClass(name = "org.springframework.web.client.RestClient")`), so a web-less
  consumer (a pure Kafka relay) also starts; no behaviour change for any consumer that has
  `spring-web`. Neither nested class is `@Configuration` — a stereotype would make it a
  component-scan candidate, and a consumer whose scan root covers `org.opentmf.outbox` would
  register it ahead of the auto-configuration order, where `@ConditionalOnBean(KafkaTemplate)`
  evaluates before `KafkaAutoConfiguration` exists and the publisher silently vanishes; lite
  member classes are processed only through the outer auto-configuration. Compiled against
  `tmf630-toolkit-all` 3.3.0 (was 3.1.1), the line this BOM pins; the dependency stays optional.
  No schema, relay or failure-policy change — 1.2.0's surface stands. `KafkaLessStartupTests`
  pins all of it (a child-first classloader that defines the library classes without the hidden
  package, a signature scan of the outer class, the no-stereotype rule on the nested ones).
  - **Consumer action:** a Kafka-less consumer that worked around this by adding `spring-kafka` and
    excluding `KafkaAutoConfiguration` can drop both at its next touch. Consumers with Kafka pick
    1.2.1 up through the BOM with no behaviour change.

## [2.1.28] - 2026-09-16

### Fixed
- Updated `openid-rbac-security` to **3.2.1** (from 3.2.0 — fix only, no configuration or API
  change). On the servlet stack the typed `503`'s `Retry-After` reached the wire only when the
  application's exception mapper carried the exception's headers through: the library handed
  `JwkSetUnavailableException` to the application's `HandlerExceptionResolver`s first and wrote the
  headers only on the bare fallback path, so a mapper that rebuilds the answer as
  `ResponseEntity.status(body.getStatus()).body(body)` sent the `503` without `Retry-After`. The
  renderer now writes the exception's headers before delegating (a `ResponseEntity` adds its headers
  without resetting those already set) and once more after rendering, so a resolver that copies them
  itself leaves one value. Reactive was already correct.

## [2.1.27] - 2026-09-16

### Updated
- Updated `openid-rbac-security` to **3.2.0** (from 3.1.0 — adopter checklist: the BOM bump and
  nothing else; no new configuration is required and the 3.1.0 status matrix is unchanged). The
  signing keys of every trusted issuer are now fetched **cache-first, off the request path**: the
  JWK set is loaded at startup by a background warm-up (logged by the issuer's configured name,
  never its URL), refreshed in the background ahead of the cache expiry, served stale while
  refreshes fail for up to `outage-ttl`, and refreshed on an unknown key id at most twice per
  interval — after the first successful load no request thread ever performs a fetch again (one
  cache, one refresh thread per issuer, shared by both stacks and both ports). An issuer whose keys
  have never been obtained answers a **typed `503`** with `Retry-After` and a problem body of type
  `urn:opentmf:security:problem:signing-keys-unavailable`, rendered through the application's own
  error rendering like the 3.1.0 404/405 — previously the lazy first-request fetch failed with
  `AuthenticationServiceException`, i.e. the container's 500. A bad or expired token is still 401,
  an unknown issuer is still 401 before any key is looked at, anonymous callers and whitelisted
  paths are unaffected; `JwtService.decodeJwt` throws the same typed `JwkSetUnavailableException`.
  Any service that added an interim filter of its own to render a 503 for this case should remove
  it with the bump, or two responders compete. New `opentmf.security.jwks.*` tuning, every default
  equal to the previous behaviour for a reachable issuer: `cache-ttl` (5m), `outage-ttl` (24h),
  `refresh-interval` (30s), `connect-timeout` / `read-timeout` (JVM defaults, then 30s),
  `on-startup-failure` (`warn` default — boot and serve 503 for that issuer until a refresh
  succeeds; `fail` stops the application when *no* issuer's keys could be loaded). **Proxy honour
  — the one deliberate change for a reachable issuer:** the fetch goes through the per-issuer
  `issuers[].proxy` (or `jwks.proxy` in single-issuer mode), else the JVM proxy properties, else
  the process environment `HTTPS_PROXY`/`HTTP_PROXY` with `NO_PROXY` exclusions (which the JDK
  never reads on its own), else direct. A deployment that exports `HTTPS_PROXY` without listing the
  identity provider's host in `NO_PROXY` fetched the keys *directly* before and goes through the
  proxy from 3.2.0 — set `NO_PROXY` or the per-issuer `proxy` if that is not what you want. The
  reactive fetch now honours the JVM proxy properties and gains a read timeout, and key lookup runs
  on `boundedElastic`, never an event-loop thread. A `classpath:` JWK set inside a fat jar now
  loads (its `jar:` URL used to fail every bearer request with a 500). Source-level:
  `ServletJwtSupport` / `ReactiveJwtSupport` take a `TrustedIssuerKeys` second constructor argument
  (provided by the new `JwksAutoConfiguration`); `ServletResourceRetriever`,
  `ReactiveResourceRetriever` and `ResourceRetrieverSupport` are removed.

## [2.1.26] - 2026-09-15

### Updated
- Updated `openid-rbac-security` to **3.1.0** (from 3.0.0 — **breaking for any consumer relying on
  401/403 for unknown paths, on the `Allow` header of a 405, or still setting
  `unmatched-method-response`**). Every request is now answered by one fixed HTTP-status matrix,
  evaluated before authentication and before the access rules, in this order: **404** when no
  handler serves the path — anonymous *and* authenticated (such a path used to fall to
  `other-endpoints` and answer 401/403; that non-disclosure posture is dropped, and a whitelisted
  prefix with nothing behind it or a path variable carrying a `/` is a 404 too); **405 with no
  `Allow` header** when the path exists but the method is not implemented on it, for every caller
  (3.0.0 answered 401 anonymously and 405 + `Allow` authenticated) and for unknown method names
  such as `PROPFIND`/`BREW` (previously a firewall 400 with Boot's error JSON — the library now
  registers a strict firewall that lets any method name through; a consumer's own firewall bean
  takes precedence); **401** for no or an invalid token; **403** for a valid token without the role.
  The 404/405 bodies are the application's own error rendering: the library raises the exceptions
  Spring raises natively (`NoHandlerFoundException`, `HttpRequestMethodNotSupportedException`;
  `ResponseStatusException`, `MethodNotAllowedException` on reactive) through the application's
  resolvers, so a `@ControllerAdvice`/`ProblemDetail`/TMF-`Error` renderer answers them like
  everything else; without one the application gets Spring's default resolver → the container's
  error page, which in a Boot application is Boot's default error JSON (the same body a permitted
  path gets natively). A handler lookup that fails is left to the access rules, never turned into a
  404 the mappings did not claim. Blacklisted paths are subject to the matrix first (unmapped → 404,
  unimplemented method → 405), so they no longer answer a uniform 403. "The path is served" now
  counts every `HandlerMapping` in dispatcher order — functional routes, resource handlers (a
  static-resource handler claims a path only when the resource resolves, and implements `GET`/`HEAD`
  only) and consumer-registered mappings — not just annotation-based controllers; a request the
  container dispatches to a non-MVC servlet bypasses the matrix. The management port follows the
  same matrix from its own handler mappings: an unexposed actuator path is 404, an unserved method
  405 — alerts keyed on 401/403 from that port need 404/405 added. Plain `OPTIONS` is unchanged in
  effect (401 anonymous; 200 + `Allow` with a valid token). **Removed:**
  `opentmf.security.unmatched-method-response` and its management twin — the matrix is not
  configurable, and a configuration that still sets either **fails at startup** naming the property
  (remove it); the public types `UnmatchedMethodResponse`, `MethodNotAllowedAccessDeniedHandler`,
  `MethodNotAllowedServerAccessDeniedHandler`, `BlacklistDecision`, `ReactiveBlacklistDenial`, and
  `EndpointRules.allowedFor` (replaced by `EndpointRules.answerFor`). Contract tests that pin
  `Allow` on a 405, 401 for an anonymous unimplemented method, or 401/403 for an unknown path must
  be rewritten. The `GET`→`HEAD` coverage and the upper-case method restriction of 3.0.0 are
  unchanged. Build-only tooling bumps; no dependency change in the published artifact.

## [2.1.25] - 2026-09-12

### Updated
- Updated `tmf630-toolkit` to **3.2.2** (from 3.1.1 — 3.2.0 to 3.2.2 in one step; additive API,
  plus one behavioural tightening on sorting). 3.2.0 adds `@Tmf630PassThrough({"version"})`: a
  handler names exact query parameters that are not entity properties (a mandatory `version`
  selector, a derived `state`), and both filter terminals (`@QuerydslPredicate`,
  `@Tmf630JsonbFilter`) leave them alone instead of answering 400 under
  `on-unknown-field: REJECT`. It is honoured on an API interface method as well as on the
  implementation, matches exact names only (`version.eq` is still a filter key) and has no global
  switch. **Behaviour change — a correct request can now fail:** on a handler that binds a filter
  root (`@QuerydslPredicate(root = X)` or `@Tmf630JsonbFilter(root = X)`), a plain `sort=` key must
  name a declared field of `X`, resolved exactly like a filter key, so **sorting on a getter-only
  (derived) property that Spring Data accepted now answers 400** with no typo involved. Unknown or
  mistyped keys answer **400** too (*"Unknown sort property"*), where JPA used to fail with a 500
  through a service's catch-all and Mongo/JSONB silently ignored them. Check which sort keys your
  clients send before rolling out, or declare a `TmfSortKeyValidator` bean of your own
  (e.g. `TmfSortKeyValidator.NONE`) to opt out. Handlers without a filter root, and correlated
  sort terms (`field[key=value].leaf`, JsonPath), are unchanged. No source break: the new
  `Tmf630FilterParser` / `Tmf630JsonbClauseBuilder` overloads and resolver constructors are
  additive. 3.2.1 removes regular-expression backtracking from JSONB `filter=` wrapper recognition
  (a crafted `$[?(...)]` / `length() == N` value with a long whitespace run could burn CPU on the
  request thread, bounded by the container's URL-length limit; the accepted grammar is unchanged),
  and a positional `[N]` in a correlated sort term containing a line break is now rejected with a
  message naming the real problem (still 400). 3.2.2 repairs three errors in the published
  Javadoc (a heading out of sequence, an unescaped `<…>`, a dead link); no code changed.
- Updated `opentmf-cadenzaflow` to **1.3.0** (from 1.2.3 — **breaking for any deployment that
  activates a Spring profile**). The image no longer supplies security defaults to a deployment:
  the whole `opentmf.security` block (issuer, `user-claim`, both ACLs) and the role-bearing
  actuator settings now live only under a `standalone` profile, activated through
  `spring.profiles.default: standalone`. Set `SPRING_PROFILES_ACTIVE` to anything and the image
  contributes **nothing** to those properties — supply your own. With neither
  `opentmf.security.jwk-set-uri` nor `opentmf.security.issuers` present the service refuses to
  start: the safe direction, but a **boot-time** failure that no render-time gate sees, so bring up
  one instance before trusting a green pipeline. Through 1.2.3 a deployment that mounted no
  security block silently inherited the image's ACL, written in role names that meant nothing to
  it, which could leave `/actuator`, `/actuator/metrics` and `/actuator/loggers` answering
  **200 anonymously** on the management port. A bare `docker run` keeps its defaults with one
  exception: **anonymous log-level writes are closed**. A tokenless
  `POST /actuator/loggers/<logger>` used to answer 204 and change the level; reading a level now
  needs a valid token and changing one needs `admin`, so scripts that change log levels on the
  management port without credentials stop working. `/actuator/env` values stay masked for a
  deployment (Spring Boot's `show-values: never` until it opts in), and `/actuator/info` left the
  standalone whitelist.

## [2.1.24] - 2026-09-08

### Updated
- Updated `opentmf-api-clients` to **3.0.0** (major — one breaking signature on the reactive
  surface). Adds an **entity view** on both surfaces: `client.entity()` returns a `TmfEntityClient` /
  `ReactiveTmfEntityClient` whose verbs answer `ResponseEntity<...>` / `Mono<ResponseEntity<...>>`,
  so `Location`, `ETag` and custom `X-*` headers are readable on the success path; `sub(...)`
  composes with it, and untyped `delete` now returns `ResponseEntity<Void>` instead of discarding
  the status. **Breaking:** reactive `listPaged*` returns `Mono<TmfPage<List<R>>>` instead of
  `Mono<TmfPage<Flux<R>>>` — a body `Flux` inside a page was a single-subscription live stream that
  left the connection undrained when only metadata was read; migrate `flatMapMany(TmfPage::getContent)`
  to `flatMapIterable(TmfPage::getContent)` (see the library's MIGRATION.md). **Fixed:** an
  `AuthType.NONE` client threw `Authorization token must not be empty.` before any request reached
  the network; it now sends no `Authorization` header. Conversely a BEARER/BASIC client whose token
  service returns a blank token now fails locally instead of collecting a remote 401.
- Updated `opentmf-cadenzaflow` to **1.2.3** (from 1.1.5 — 1.2.0, 1.2.1, 1.2.2 and 1.2.3 in one
  step; there was no 1.1.6). **Every deployment on 1.2.1 or earlier should move**, for two reasons
  that are not about features. 1.2.2 fixes a `logback-spring.xml` that left the root logger with an
  **empty appender list** on every release from 1.0.0 through 1.2.1 — the service started, served
  traffic and reported healthy while writing **no application logs at all**, unless a file was
  mounted via `LOGGING_CONFIG`; the appender is now chosen by plain variable substitution
  (`LOGGING_APPENDER`, defaulting to console). 1.2.1 moves embedded Tomcat to 11.0.25 via explicit
  `dependencyManagement` overrides, closing three CRITICAL findings including **CVE-2026-65182**
  (a security-constraint bypass) that no Boot GA pins a fix for yet. 1.2.2 and 1.2.3 also repair
  log masking for values reached through a URL-encoded request line — `Basic` and opaque bearer
  credentials leaked in full, and an encoded `+` exposed MSISDNs and card numbers verbatim.
  1.2.0 adds the incident-operations surface under `/engine-rest/extensions/incident*` (a grouped
  report across a root BPMN's whole call tree that counts each failure once, one-call bulk retry of
  a group, and a TMF-630-paged incident list) plus a published `docs/openapi.yaml` unioning the
  engine's API with this service's additions. It also picks up `openid-rbac-security` 3.0.0, whose
  405 behaviour reaches only the actuator endpoints — Jersey-served `/engine-rest/**` paths are
  invisible to the method resolver, so engine 401/403 answers are unchanged — plus CadenzaFlow
  engine 1.2.3 and Spring Boot 4.1.1.

## [2.1.23] - 2026-08-31

### Updated
- Updated `openid-rbac-security` to **3.0.0** (major — read before upgrading). A `GET` access rule
  now also covers `HEAD` on the same path with the same roles, so a `HEAD` probe that previously
  fell through to `other-endpoints` now carries that rule's roles (401/403 where it used to be
  served). A denied request for a method the application does not implement answers **405** with an
  `Allow` header instead of 403, governed by the new `unmatched-method-response` (default
  `method-not-allowed`; set `deny` to restore the old uniform 403); a plain `OPTIONS` on a served
  path answers 200 + `Allow`. **Breaking:** `secure-endpoints[].method` / `allowed-endpoints[].method`
  now accept only `GET`, `POST`, `PUT`, `PATCH`, `DELETE` in upper case — a `HEAD`/`OPTIONS`/`TRACE`
  entry, or a lower-case value, fails startup. Delete `HEAD` entries (a `GET` rule covers them) and
  replace `OPTIONS` entries with real CORS configuration. Spring Boot 4.1.1.

## [2.1.22] - 2026-08-27

### Updated
- Updated `opentmf-outbox-service` to 1.2.0 (additive throughout — a 1.0.0 or 1.1.0 consumer
  upgrades unchanged). Adds a **per-publisher failure policy**: `OutboxPublisher` gains default
  `maxAttempts(event)`, `backoff(event, attempt)` and `onExhausted(event)` → `PARK` (default) or
  `DROP`, so the resolved publisher — not the library-wide setting — books every failure. A
  publisher throws the new `TerminalOutboxException` to reach the exhaustion outcome immediately.
  Parking is now recorded by an explicit `parked_on` stamp, and the relay's claim predicate reads
  that stamp instead of bounding on the attempt count, so per-publisher budgets are honoured by the
  claim itself. Adds a private per-row `reference` (`OutboxAppend.withReference`, filterable on the
  ops list) that is never forwarded to the wire — `headers` is the wire, `reference` is private —
  and promotes `OutboxHeaders` to public API, since the idempotency-key format is a cross-service
  contract. Adds `GET /ops/outbox/state/{state}`, with the `/ops` surface answering 404/409/400
  through Spring's `ErrorResponse` contract. Onboarding a pre-library `outbox` table is now owned
  by the library: `001`/`002` are `MARK_RAN`-guarded when the table or its columns already exist and
  `003-outbox-policy-reference-onboarding` adds every missing column with `add column if not
  exists`, so recorded checksums are unchanged. Ships consumer-conformance ITs.
  - **Fixed:** the HTTP publisher appended a stored header that collided with a relay header
    (`x-event-type` went out twice); it now replaces the colliding value.
  - **Consumer action:** a global exception handler that swallows Spring's `ErrorResponse` turns the
    `/ops` 404/409/400 into a generic 500 — adopt the dnms-template `GlobalExceptionMapper` fix
    (PR #12) in the same release as this upgrade. Test fixtures that seed parked rows must now set
    `parkedOn`; attempts alone no longer park a row.
  - **Upgrade note:** a row parked under 1.1.0 has no `parked_on` stamp, so it becomes claimable
    again on upgrade and parks with the stamp on its next failure.

## [2.1.21] - 2026-08-27

### Updated
- Updated `opentmf-outbox-service` to 1.1.0 (purely additive — a 1.0.0 consumer upgrades unchanged).
  Adds a **scheduled-send hold**: `OutboxWriter.append(OutboxAppend)` accepts a `release_at` instant
  that is frozen at write time, so an effect is not delivered before it and delivery-retry backoff
  never moves it. Adds **cancellation of an unreleased effect** via
  `OutboxMaintenanceService.cancel(id)` and `POST /ops/outbox/{id}/cancel`, recorded as
  `cancelled_on`; if the relay has already claimed the row, the claim wins the race against a
  concurrent cancel. Liquibase changeset `002-outbox-hold-and-cancel` adds the two nullable columns
  and auto-applies on next start.

## [2.1.20] - 2026-08-26

### Added
- Added `opentmf-outbox-service` 1.0.0 (`org.opentmf.util:opentmf-outbox-service`) — transactional
  outbox pattern as a Spring Boot starter, using the client application's JDBC datasource and
  Kafka/HTTP infrastructure. The business transaction writes its state change and one outbox row in
  the same local transaction; an in-service relay then delivers the event at-least-once, so
  "state changed AND the platform heard it" never has a crash window. The library auto-configures
  itself when a JPA `DataSource` is present and owns a single `OUTBOX` table created by its bundled
  Liquibase changelog, which consumers include by reference. Kafka, web, and `tmf630-toolkit-all`
  are optional dependencies, so the delivery transport and the `/ops` query surface are opt-in.

## [2.1.19] - 2026-08-25

### Updated
- Updated `camunda7-test-framework` to 2.1.0 (chaos toolkit, engine clock control and scripted
  task outcomes; Spring Boot 4.1.1 clears 15 transitive HIGH/CRITICAL CVEs. `VariableUtil` is now
  `final` — code that instantiated or subclassed it no longer compiles).

## [2.1.18] - 2026-08-20

### Added
- Added `opentmf-cadenzaflow` 1.1.5 (`org.opentmf.cadenzaflow:opentmf-cadenzaflow`) — a Spring Boot 4
  microservice embedding CadenzaFlow CE, the maintained Camunda 7 fork, with Spin, Keycloak OpenID
  auth, and `openid-rbac-security`. Like `opentmf-camunda7`, this is a **deployable service** rather
  than a library: the normal way to consume it is the published image
  (`ghcr.io/opentmf/opentmf-cadenzaflow:<version>`, plus `-aws` and `-azure` flavours). The BOM
  manages its version so the coordinate is pinnable from one place.

### Updated
- Updated `opentmf-api-clients` to 2.1.0 (sub-resource path support: new `sub(template, vars...)`
  on both `TmfClient` and `ReactiveTmfClient` returns a derived client scoped to a nested path,
  so every existing verb works against it unchanged — e.g.
  `orderClient.sub("/{orderId}/action/{action}/item", orderId, action).get(itemId, Item.class)`.
  Path variables are strictly encoded and arity is validated eagerly. Adds the immutable
  `SubResourcePath` value type plus `UriBuilderUtil` overloads. **Source-compatibility note:**
  direct callers of `UriBuilderUtil.buildUriWithId(server, endpoint, id, null)` passing a null
  literal must now cast it to `(TmfRequestContext)` because of the new overload; binary
  compatibility and the typed clients' public API are unaffected).
- Updated `opentmf-http-clients` to 2.1.8 (build now enforces exact toolchain versions — JDK 17.x
  and Maven 3.9.x — instead of minimums; a consumer-facing no-op, source builds only).

## [2.1.17] - 2026-08-10

### Updated
- Updated `tmf630-toolkit` to 3.1.1 (Adds JSONB URL-binding bridge: `@Tmf630JsonbFilter`, 
  plus, migrates JSONB module to Jackson 3).
- Updated `opentmf-http-clients` to 2.1.7 (W3C trace-context propagation on outbound calls).
- Updated `opentmf-db-lock-service` to 2.2.2 (PostgreSQL: no more ACCESS EXCLUSIVE table lock on every startup)

## [2.1.16] - 2026-08-05

### Updated
- Updated `openid-rbac-security` to 2.3.0 (multi-issuer resource-server support via
  `opentmf.security.issuers`, each entry with its own JWK set / claim mapping / optional
  audience; opt-in — existing single-issuer config unchanged; Spring Boot 4.1.0).
- Updated `opentmf-http-clients` to 2.1.6 (transport-level Apache retry removed — retry
  policy is now solely owned by `executeWithRetry` / `WebClientUtil.retry`, so all three
  backends behave identically; `Retry-After` honoured as a floor, bounded by new
  `max-retry-after`; response headers and `Content-Type` charset carried through to
  error handling).
- Updated `tmf630-toolkit` to 3.0.1 (`.regex` / LIKE-family now accept polymorphic
  `Object` / `Serializable` fields on Mongo and JSONB backends; JPA still rejects and
  points at `@Tmf630JsonbBacked` as the escape).
- Updated `opentmf-mockserver` to 2.1.11 (Docker image coordinate restored to
  `ghcr.io/opentmf/opentmf-mockserver` — after the repo rename to
  `opentmf-mockserver-parent`, the 2.1.10 image published to the `-parent` path;
  Maven artifacts are unchanged from 2.1.10).

## [2.1.15] - 2026-07-30

### Fixed
- Updated `opentmf-http-clients` to 2.1.5 — fixes an `HttpClientRegistry` regression
  shipped in 2.1.4 that broke ApplicationContext startup on any classpath without
  reactor-netty (a synthetic registry method exposed a reactor-netty type, so Spring's
  bean introspection failed with `NoClassDefFoundError`). Sync-only apps that adopted
  2.1.14 should upgrade.

## [2.1.14] - 2026-07-30

### Added
- Added `opentmf-errors` 1.0.0 — shared error-code catalog with `opentmf-errors-core`
  (registry contract + seed catalog + generated OAS 3.1 components) and
  `opentmf-errors-spring` (ProblemDetail and TMF-style renderers).

### Updated
- Updated `tmf630-toolkit` to **3.0.0** (major): JPA positional `[N]` filter with
  `@OrderColumn`, multi-hop correlated sort with `min()`/`max()` aggregators, and
  entity versioning per TMF-630 Part 4 §2. Repo split adds new sub-modules
  `tmf630-toolkit-jpa-correlated-sort`, `tmf630-toolkit-jsonb`, and
  `tmf630-toolkit-mongo-split-collection`.
- Updated `openid-rbac-security` to 2.2.0 (pluggable 401/403 response handlers via
  bean-presence; fixes ineffective servlet management-port security shipped in 2.1.0).
- Updated `opentmf-http-clients` to 2.1.4 (new `HttpClientRegistry` for programmatic
  runtime lifecycle; optional resilience4j circuit-breaker / bulkhead / time-limiter
  integration; Apache pool micrometer gauges).
- Updated `opentmf-mockserver` to 2.1.10 (repo is now multi-module under
  `opentmf-mockserver-parent`; adds the new `opentmf-mockserver-test-support` module
  — fluent JUnit 5 harness with TMF/OIDC/stub builders and JVM-shared MockServer mode).
- Updated `camunda7-bpmn-sync-service` to 2.1.1 (single configurable
  `resource-location` for BPMN + DMN files; fixes 2.1.0 startup failure when
  `classpath:dmn/` is absent).

## [2.1.13] - 2026-07-24

### Updated
- Updated `tmf630-toolkit` to 2.1.5 (several further TMF630 compliancy improvements).
- Updated `opentmf-mockserver` to 2.1.8 (fixes: Rare concurrent cache update issues under heavy load).
- Updated `opentmf-camunda7` to 24.0.6 (fixes a GraalJS polyglot-context memory leak in
  JavaScript script tasks — contexts are now closed per evaluation; Spring Boot 3.5.16,
  GraalJS 25.1.3, Docker images on Eclipse Temurin JRE 25).
- Updated `camunda7-bpmn-sync-service` to 2.1.0 (DMN deployment support, and Multi tenancy deployment support)

## [2.1.12] - 2026-06-26

### Updated
- Updated `opentmf-db-lock-library` to 2.2.1 (dynamically removes stale locks).
- Updated `opentmf-mockserver` to 2.1.6 (external token validation enhancements, + idempotency-key handling)

## [2.1.11] - 2026-06-10

### Updated
- Updated `tmf630-toolkit` to 2.1.3 (JSONPath sort terms now accept coercion (num() / str() / date()) and aggregator (min() / max()) wrappers).

## [2.1.10] - 2026-06-03

### Updated
- Updated `tmf630-toolkit` to 2.1.2 (Fixes multi-term Mongo correlated sort failing with "parallel arrays").

## [2.1.9] - 2026-06-01

### Updated
- Updated `camunda7-incident-logger` to 2.0.1 (Migrates to cibseven 2.2.0).
- Updated `opentmf-mockserver` to 2.1.4 (Adds dynamic PUT callback).
- Updated `camunda7-test-framework` to 2.0.2 (Migrates to cibseven 2.2.0 and boot-4 starters, removing the previous bridge classes).
- Updated `opentmf-api-clients` to 2.0.9 (Adds PUT methods to TmfClient implementations).

## [2.1.8] - 2026-05-18

### Fixed
- Updated `opentmf-api-clients` to 2.0.8.
  - Fix double percent-encoding of resource path segments when UriBuilderUtil.withContext(...) or UriBuilderUtil.withPagination(...) is applied to a URI that already contains an encoded {id} (typically TMF composite keys like Spec:(version=1))

## [2.1.7] - 2026-05-16

### Fixed
- Updated `tmf630-toolkit` to 2.1.1.
  - Mongo correlated-sort executor now places rows with null / missing sort keys last regardless of direction.

## [2.1.6] - 2026-05-11

### Updated
- Updated `tmf630-toolkit` to 2.1.0.
  - Comma-separated value lists are now accepted for the multi-value attribute-filter operators .in, .nin, and .between.
  - JSON Path filter grammar now supports the unary negation form `[?(!@.field)]`
  - Simple-rich correlated sort now accepts the outer coercion wrapper form
  - New setting `opentmf.tmf630.attribute-filtering.on-unknown-json-path-field` (default IGNORE).

## [2.1.5] - 2026-05-11

### Updated
- Updated `opentmf-v4-api` to 4.1.1 (regeneration with the new version of openapi-multi-generator, and per-TMF api submodules from `*.10` to `*.11`).
- Updated `opentmf-v4-models` to 4.1.1 (regeneration with the new version of openapi-multi-generator, and per-TMF model submodules from `*.10` to `*.11`).
- Updated `dnext-v4-models` to 2.12.1 (Adds Snapshot API on product/service/resource inventories, `pointOfNoReturnIFOC`/`pointOfNoChange` on orders, `completionCallback` on cancel-orders, `extensions` on resource shapes, and with the release of the new openapi-multi-generator, fixes the datatypes that were set as String because the generator didn't follow `oneOf` previously).
- Updated `opentmf-api-clients` to 2.0.7 (supports multiple-add-objects through `patchCollections` methods).
- Updated `opentmf-commons` to 2.2.0 (starts accepting unicode alphabetical characters in `@SafeText`, makes `@SafeQuery` deprecated, introduces `@SafeUrl` and optimizations).
- Updated `opentmf-mockserver` to 2.1.3 (adds multi-jsonPatch add statements).

## [2.1.4] - 2026-05-06

### Fixed
- Updated tmf630-toolkit to 2.0.2 (Fixes filter coexistence with rich sort variations)

## [2.1.3] - 2026-05-05

### Added
- Added the new library tmf630-toolkit-mongo-aggregation version 2.0.1.

### Changed
- Updated tmf630-toolkit to 2.0.1 (Correlated sort for MongoDB + TMF630 syntax shorthand)

## [2.1.2] - 2026-04-27

### Added
- Added the new library auditor-aware version 3.0.0. It replaces now obsoleted auditor-aware-jpa 2.0.0 library.

## [2.1.1] - 2026-04-25

### Updated
- Updated openid-rbac-security version to 2.1.0. (Adds management server security configuration as well).

## [2.1.0] - 2026-04-23

### Updated
- Updated opentmf-db-lock-service version to 2.2.0. (Adds multi-db support).

## [2.0.9] - 2026-04-20

### Fixed
- Updated dnext-catalog-sync-service to 2.0.4. (Adds `Accept` headers).

### Updated
- Updated opentmf-db-lock-service to 2.1.0 (Enhances `@UsingClusterLock` so that the success condition can be specified)

## [2.0.8] - 2026-04-16

### Fixed
- Updated opentmf-http-clients to 2.1.3 (Fixes Content-Type header during token retrieval when Content-Type already exists in the fixed-headers).
- Updated opentmf-api-clients to 2.0.6 (Started setting `Accept: application/json` if the `Accept` header is not set via fixed-headers or in TmfRequestContext)

## [2.0.7] - 2026-04-14

### Fixed
- Updated opentmf-api-clients to 2.0.5 (re-adds the missed fixed-headers from the old library)

## [2.0.6] - 2026-04-14

### Fixed
- Updated camunda7-test-framework to 2.0.1 (fixes JerseyApplicationPath registration)

## [2.0.5] - 2026-04-06

### Fixed
- opentmf-http-clients updated to 2.1.2 (Fixes logbook configuration timing issue)

### Changed
- opentmf-api-clients updated to 2.0.3 (starts using opentmf-http-clients 2.1.2)

## [2.0.4] - 2026-04-06

### Fixed

- opentmf-api-clients updated to 2.0.3 (All four auto-configuration classes refactored from constructor-based bean registration to static BeanDefinitionRegistryPostProcessor pattern. This fixes the "Ghost Configuration" problem in Spring Boot 4 where auto-configuration classes that only register beans in their constructor are never instantiated if no @Bean method output is requested by the dependency graph.)

## [2.0.3] - 2026-04-06

### Changed

- opentmf-api-clients updated to 2.0.2 (Ensures autoconfiguration to run after opentmf-http-clients)

## [2.0.2] - 2026-03-30

### Changed

- opentmf-http-clients updated to 2.0.1 (Fixes scope:pom in the started artifacts).
- opentmf-api-clients updated to 2.0.1 (Starts using the newly released opentmf-http-clients)

## [2.0.1] - 2026-03-30

### Changed

- dnext-catalog-sync-service updated to 2.0.3 (Starts synchronizing product catalog endpoints too).

## [2.0.0] - 2026-03-28

**Backward Incompatible** — targets Spring Boot 4.x and Jackson 3.x.

### Added

- Added opentmf-json-patch for JSON Patch (RFC 6902) support
- Added opentmf-v4-api with generated model interfaces for 80+ TMF v4 specifications
- Added opentmf-http-clients with REST and reactive HTTP client support and cached bearer-token handling
- Added opentmf-api-clients with REST and reactive TMF-630 compliant API clients
- Expanded opentmf-v4-models from 25 to 85 per-TMF module versions
- Added dnext-v4-models, a complete set of models generated from DNext swaggers covering every available DNext microservice

### Changed

- All libraries upgraded to their 2.x versions for Spring Boot 4.x and Jackson 3.x compatibility

### Removed

- Removed opentmf-web-clients (replaced by opentmf-http-clients)
- Removed opentmf-clients-base (replaced by opentmf-api-clients)
- Removed opentmf-v4-clients (no direct replacement, but easy client bean generation via opentmf-api-clients)
- Removed dnext-opentmf-v4-models (replaced by dnext-v4-models)

## [1.4.9] - 2026-03-20

### Changed

- Updated `opentmf-clients-base` to 1.1.8 (Does not try to set Content-Type on post operations, if the header already exists).
- Updated `opentmf-v4-clients` and its submodules to 1.1.8 (Starts using the new opentmf-clients-base).

## [1.4.8] - 2026-03-20

### Changed

- Updated `dnext-catalog-sync-service` to 1.1.1 (Context-aware resourceSpecification endpoints)
- Updated `tmf630-toolkit` and its submodules to 1.0.7 (Better error messages for invalid dates and reduces noise in the logs).
- Updated `opentmf-clients-base` to 1.1.7 (Does not try to set Content-Type on patch operations, if the header already exists).
- Updated `opentmf-v4-clients` and its submodules to 1.1.7 (Starts using the new opentmf-clients-base).

## [1.4.7] - 2026-03-11

### Changed

- Updated tmf630-toolkit to 1.0.6 (new `@Tmf630Result` annotation)
- Updated opentmf-camunda7 to 24.0.3 (Separate docker image that supports AWS IAM)
- Updated opentmf-clients-base to 1.1.6 (Fix: Enforces jakarta validations for all client configurations)
- Updated opentmf-v4-clients to 1.1.6 (starts using 1.1.6 of opentmf-clients-base)

## [1.4.6] - 2026-03-03

### Fixed

- Updated tmf630-toolkit to 1.0.5. It adds offset, limit, and fields to the reserved parameter set to prevent 400 Bad Request when `onUnknownField=REJECT` and request contains offset, limit, and/or fields. 

## [1.4.5] - 2026-03-03

### Changed

- Updated tmf630-toolkit to 1.0.4 (enhances enum handling)
- Updated camunda7-test-framework to 1.0.8 (adds withCount/withExecutionConsumer builder methods)

## [1.4.4]

### Fixed

- Updated tmf630-toolkit to 1.0.3 (fixes a bug in the `sort=` parameter handling)

## [1.4.3]

### Fixed

- Updated tmf630-toolkit to 1.0.2 (fixes deep nested key parsing and enables default .eq match)

## [1.4.2]

### Added

- Updated tmf630-toolkit to 1.0.1 (adds `filter=` JsonPath support both for JPA and MongoDB)

## [1.4.1]

### Added

- Added tmf630-toolkit artifacts

## [1.4.0]

### Changed

- Updated camunda7-test-framework to 1.0.7 (enhances support for more task types that can receive a correlation message)

## [1.3.9]

### Changed

- Updated openid-rbac-security to 1.1.1 (includes fallback user claims support)
- Updated auditor-aware-jpa to 1.0.5 (fixes potential NPE in auditor-aware providers when authentication.getName() returns null)

## [1.3.8]

### Fixed

- Updated opentmf-commons to 1.0.7 (fixes reading from the classpath by using the current thread's classloader)

## [1.3.7]

### Changed

- Updated opentmf-mockserver to 1.1.1
- Updated opentmf-camunda7 to 24.0.2
- Updated opentmf-clients-base to 1.1.5
- Updated opentmf-v4-clients to 1.1.5

## [1.3.6]

### Changed

- Updated camunda-incident-logger to 1.0.4 (camunda 7.24 upgrade)
- Updated camunda7-test-framework to 1.0.6 (camunda 7.24 upgrade)
- Updated opentmf-camunda7 to 24.0.0 (camunda 7.24 upgrade)
- Updated opentmf-mockserver to 1.0.7 (fixes encoded path parsing)

## [1.3.5]

### Changed

- Updated opentmf-clients-base to 1.1.3 (fixes default error handling)
- Updated opentmf-v4-clients to 1.1.3 (uses the new opentmf-clients-base)
- Updated opentmf-db-lock-service to 1.0.9 (adds a new method isLocked(LockType))
- Updated auditor-aware-jpa to 1.0.4 (marks createdOn and createdBy fields as not updatable)

## [1.3.4]

### Fixed

- Updated opentmf-clients-base to 1.1.3 (fixes default error handling)
- Updated opentmf-v4-clients to 1.1.3 (uses the new opentmf-clients-base)

## [1.3.3]

### Fixed

- Updated camunda7-bpmn-sync-service from 1.1.1 to 1.1.3 (fixes resource name read issue)
- Updated opentmf-mockserver from 1.0.2 to 1.0.6 (fixes versioned entity handling)

### Changed

- Updated opentmf-camunda7 from 23.0.0 to 23.0.2 (single sign-on support added for OpenID auth)

## [1.3.2]

### Changed

- Updated opentmf-commons to 1.0.6 (numeric values allowed in OffsetDateTime deserialization)
- Updated opentmf-clients-base to 1.1.2 (better error handling for non-json content or no content)

## [1.3.1]

### Changed

- Updated dnext-catalog-sync-service to 1.1.0

## [1.3.0]

### Changed

- Initial open-source version, replacing pia with opentmf

## [1.2.7]

**Backward Incompatible**

### Changed

- Updated tmf-clients-base to 1.1.0
- Updated tmf-v4-clients to 1.1.1

## [1.2.6]

### Changed

- Updated tmf-clients-base to 1.0.5
- Updated tmf-v4-clients to 1.1.0

## [1.2.5]

### Added

- Updated dnext-tmf-v4-models to 1.0.9 (adds DNextProductOffering with the extended field "rules")

## [1.2.4]

### Added

- Updated camunda-7-test-framework to 1.0.3 (adds the ability to use set listeners for each and every task, not only receive tasks)

## [1.2.3]

### Added

- Updated pia-web-clients to 1.0.9 (adds the ability to use mutual TLS authentication)

## [1.2.2]

### Added

- Updated tmf-v4-utils to 1.0.4 (adds validateOrder method to ServiceOrderUtil)

## [1.2.1]

### Changed

- Updated dnext-tmf-v4-models to 1.0.8

## [1.2.0]

### Changed

- Updated dnext-tmf-v4-models to 1.0.7

## [1.1.9]

### Changed

- Updated auditor-aware-jpa to 1.0.2
- Updated pia-commons to 1.0.2

## [1.1.8]

### Changed

- Updated pia-security to 1.0.9

## [1.1.7]

### Changed

- Updated pia-commons to 1.0.1
- Updated dnext-tmf-v4-models to 1.0.6

## [1.1.6]

### Changed

- Updated pia-bpmn-sync-service to 1.1.0
- Updated pia-catalog-sync-service to 1.0.8

## [1.1.5]

### Changed

- Updated pia-security to 1.0.8

## [1.1.4]

### Changed

- Updated pia-web-clients to 1.0.8 (fewer dependencies for the reactive WebClient)
- Updated pia-bpmn-sync-service, pia-catalog-sync-service, pia-web-clients, tmf-clients-base, tmf-v4-clients
- Corrected the link from tmf-clients-base to pia-web-clients in pia-libraries.png

## [1.1.3]

### Changed

- Updated pia-bpmn-sync-service to 1.0.8

## [1.1.2]

### Changed

- Updated dynamic-mock-expectations to 1.0.1
- Updated pia-db-lock-service to 1.0.7
- Updated pia-bpmn-sync-service to 1.0.7
- Updated pia-catalog-sync-service to 1.0.6

## [1.1.1]

### Added

- Introduced auditor-aware-jpa
- Added dnext-tmf-633-model to dnext-tmf-v4-models

### Changed

- Updated pia-security to 1.0.7
- Updated pia-camunda-7 to 22.0.6

## [1.1.0]

### Changed

- Updated many library versions to their latest

### Fixed

- Fixed quoteClientProvider bean name

## [1.0.9]

### Added

- Added tmf-681 v4 model and tmf-client

### Changed

- Updated dnext tmf v4 models and tmf v4 utils

## [1.0.8]

### Changed

- Updated pia-security from 1.0.3 to 1.0.5
- Updated pia-db-lock-service from 1.0.4 to 1.0.5
- Updated pia-bpmn-sync-service from 1.0.4 to 1.0.5
- Updated pia-catalog-sync-service from 1.0.3 to 1.0.4

## [1.0.7]

### Changed

- Updated dnext-tmf-v4-models versions

## [1.0.6]

### Changed

- Updated pia-web-clients to 1.0.5
- Updated pia-db-lock-service to 1.0.3
- Updated pia-bpmn-sync-service to 1.0.3
- Updated pia-catalog-sync-service to 1.0.2
- Updated tmf-clients-base to 1.0.2
- Updated tmf-v4-clients to 1.0.4

## [1.0.5]

### Changed

- Updated pia-security to 1.0.3
- Updated pia-camunda-7 to 22.0.2 (to use pia-security 1.0.3)

## [1.0.0 - 1.0.4]

### Added

- Initial releases
