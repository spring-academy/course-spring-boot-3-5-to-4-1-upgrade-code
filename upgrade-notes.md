# 4.1 Upgrade Notes

## Initial baseline

- Code compiled successfully
  - No errors
  - Deprecation warnings
    - src/main/java/example/cashcard/SecurityConfig.java uses or overrides a deprecated API.
- found disabled/broken test - CashCardApplicationTests#shouldReturnACashCardWhenDataIsSaved
  - fixed broken test and re-enabled

_Result:_ code compiles without errors or warnings and all tests pass

## Major Release Considerations

- Spring Boot 4.0 pairs with Spring Security 7.0, which **removes** `AntPathRequestMatcher` and `MvcRequestMatcher` outright in favor of `PathPatternRequestMatcher`
- Our current Spring Boot 3.5.16 baseline (Spring Security 6.5.11) already flags this:
  ```
  [WARNING] .../src/main/java/example/cashcard/SecurityConfig.java: org.springframework.security.web.util.matcher.AntPathRequestMatcher in org.springframework.security.web.util.matcher has been deprecated and marked for removal
  ```
  - Reference: https://docs.spring.io/spring-security/reference/6.5/migration-7/web.html
- Since this is a deprecation we can already see and fix *before* the major version jump, we're doing it now rather than discovering a hard compile error later
- Replace `AntPathRequestMatcher` with `PathPatternRequestMatcher.withDefaults().matcher(...)`
- The H2 console registers its own servlet (`/h2-console/*`) alongside the app's `DispatcherServlet` (`/`). `PathPatternRequestMatcher` needs to know which servlet a pattern belongs to, so the H2 console matcher needs `PathPatternRequestMatcher.withDefaults().basePath("/h2-console").matcher("/**")` instead of a plain pattern - otherwise Spring Security can't tell which servlet's path the pattern is relative to

_Result:_ code compiles without errors or warnings and all tests pass

## Upgrade Spring Boot Parent Version

- Updated `spring-boot-starter-parent` from `3.5.16` to `4.1.0`
- Code did **not** compile cleanly:
  ```
  [ERROR] .../src/main/java/example/cashcard/CashCardApplication.java:[5,32] cannot find symbol
    symbol:   class ConfigurableBootstrapContext
    location: package org.springframework.boot
  [ERROR] .../src/main/java/example/cashcard/CashCardApplication.java:[6,32] cannot find symbol
    symbol:   class DefaultBootstrapContext
    location: package org.springframework.boot
  ```
- Spring Boot 4 moved `ConfigurableBootstrapContext` and `DefaultBootstrapContext` from `org.springframework.boot` to `org.springframework.boot.bootstrap` as part of its broader package modularization
- Updated the two imports to the new package; no other code changes were needed
- We are intentionally skipping test execution for now (`./mvnw clean compile` only) - testing gets its own dedicated set of lessons and labs later in this course

_Result:_ code compiles without errors or warnings (tests not yet run)

## Remove hard-coded Spring/Spring Boot Dependencies

- remove hardcoded `spring-data-jdbc` version, letting `spring-boot-starter-parent` manage it
- `./mvnw dependency:tree | grep spring-data-jdbc` now shows `4.1.0` (parent-managed) instead of the hardcoded `3.4.5`
- Code Compiles, No Errors or Warnings

_Result:_ code compiles without errors or warnings (tests not yet run)

## Update Non Spring/Spring Boot Managed Dependencies

- remove hardcoded `lombok` version, letting the parent manage it transitively
- Code Compiles, No Errors or Warnings
- moved hardcoded `itextpdf` version to `<properties>` - Spring doesn't manage this dependency for us, so we can't just delete the version, but we can stop hard-coding it inline

_Result:_ code compiles without errors or warnings (tests not yet run)

## Update Spring Boot Starter Names

- Spring Boot 4 modularized the old monolithic `spring-boot-autoconfigure` jar into per-technology modules, and Spring Initializr now generates dependency coordinates that match: `spring-boot-starter-web` -> `spring-boot-starter-webmvc`, `spring-boot-starter-test` -> `spring-boot-starter-webmvc-test`
- The classic names (`spring-boot-starter-web`, `spring-boot-starter-test`) still exist in Boot 4 as migration aids and would keep compiling, but we're updating to match what a fresh Spring Initializr project would generate today
- Code Compiles, No Errors or Warnings

_Result:_ code compiles without errors or warnings (tests not yet run)

## Test - Initial Test Execution

- Ran `./mvnw clean test` for the first time since the version bump
- Test *compilation* failed:
  ```
  [ERROR] .../src/test/java/example/cashcard/CashCardApplicationTests.java:[9,48] package org.springframework.boot.test.web.client does not exist
  [ERROR]   symbol:   class TestRestTemplate
  ```
  - Spring Boot 4 moved `TestRestTemplate` out of `spring-boot-test` entirely, into a new `spring-boot-resttestclient` module at `org.springframework.boot.resttestclient.TestRestTemplate`
  - Added `spring-boot-resttestclient` and `spring-boot-starter-restclient` (test scope), updated the import, and added `@AutoConfigureTestRestTemplate` to `CashCardApplicationTests` - Boot 4 no longer auto-registers a `TestRestTemplate` bean under `@SpringBootTest(RANDOM_PORT)` the way earlier versions did
- With that fixed, tests *compiled* but 15 of them *failed at runtime*:
  ```
  Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'example.cashcard.CashCardRepository' available
  ```
  - This is the payoff of the Boot 4 auto-configuration modularization: the raw `org.springframework.data:spring-data-jdbc` artifact we've been carrying since `01-kgs` compiled cleanly the whole way through, but it never brought Spring Data JDBC's repository auto-configuration with it in Boot 4 - that now lives behind the `spring-boot-starter-data-jdbc` starter
  - Swapped `org.springframework.data:spring-data-jdbc` for `org.springframework.boot:spring-boot-starter-data-jdbc`
  - **Lesson: a clean compile tells you nothing about auto-configuration. Any raw, non-Boot-starter dependency is worth double-checking after a major Boot upgrade.**

_Result:_ code compiles without errors or warnings and all tests pass

## Test Dependency Housekeeping

- Removed the explicit `assertj-core` test dependency
- Code Compiles, No Errors or Warnings, all tests still pass
  - It's managed by `spring-boot-starter-webmvc-test`

_Result:_ code compiles without errors or warnings and all tests pass
