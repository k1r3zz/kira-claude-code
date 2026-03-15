# Scan Rules — Java/Kotlin (JVM)

File priority rules for JVM projects (tag: `jvm`).

| Priority | File patterns |
|----------|--------------|
| P0 | `**/service/**`, `**/domain/**`, `**/usecase/**`, `**/handler/**` |
| P1 | `**/model/**`, `**/entity/**`, `**/repository/**`, `**/controller/**`, `**/dto/**` |
| P2 | `**/util/**` / `**/helper/**` / `**/config/**` / `**/constant/**` / `**/enums/**` with business vocabulary |
| Skip | `target/`, `build/`, `out/`, `**/test/**`, `*Test.java`, `*Test.kt`, `*Spec.kt`, `*.class` |
