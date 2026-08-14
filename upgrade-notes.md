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
