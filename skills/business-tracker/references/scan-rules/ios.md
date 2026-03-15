# Scan Rules — iOS (Swift & Objective-C)

## Swift (tag: `ios-swift`)

| Priority | File patterns |
|----------|--------------|
| P0 | `**/Services/**/*.swift`, `**/Domain/**/*.swift`, `**/UseCases/**/*.swift`, `**/Managers/**/*.swift` (business managers), `**/Interactors/**/*.swift` |
| P1 | `**/Models/**/*.swift`, `**/ViewModels/**/*.swift`, `**/Repositories/**/*.swift`, `**/Controllers/**/*.swift` (non-pure-UI), `**/Coordinators/**/*.swift`, `**/Networking/**/*.swift` (API definitions) |
| P2 | `**/Helpers/**/*.swift` / `**/Utils/**/*.swift` / `**/Constants/**/*.swift` / `**/Enums/**/*.swift` with business vocabulary |
| Skip | `Pods/`, `*.xcassets`, `*.storyboard`, `*.xib`, `*Tests.swift`, `*Spec.swift`, `DerivedData/`, `Build/`, `*.generated.swift`, `Packages/` |

## Objective-C (tag: `ios-objc`)

| Priority | File patterns |
|----------|--------------|
| P0 | `**/Services/**/*.m`, `**/Managers/**/*.m`, `**/Domain/**/*.m` |
| P1 | `**/Models/**/*.m`, `**/Controllers/**/*.m` (non-pure-UI), `**/Networking/**/*.m` |
| P2 | `**/Utils/**/*.m` / `**/Helpers/**/*.m` / `**/Constants.h` with business vocabulary |
| Skip | `Pods/`, `*.xcassets`, `*.storyboard`, `*.xib`, `*Tests.m`, `DerivedData/`, `Build/` |
