# Feature name

* Proposal: [SE-NNNN](NNNN-swiftpm-test-only-dependencies.md)
* Authors: [thislooksfun](https://github.com/thislooksfun)
* Review Manager: TBD
* Status: **Awaiting review**

## Introduction

This proposal seeks to add a natively supported way to use some external packages for use only when running tests through Swift Package Manager (SwiftPM)

Swift-evolution thread: [\[swift-evolution\] \[SwiftPM\] Proposal: Add support for test-only	dependencies](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20161226/029720.html)

## Motivation

Currently, using any external package only when testing is technically possible, but it is roundabout and not very clean or intuitive, especially when running tests manually, whether through the CLI or XCode.

The current method in use by many projects ([1][test-example-1], [2][test-example-2], [3][test-example-3], [4][test-example-4]) is to have an entirely separate `Package.swift`, commonly titled `.Package.test.swift`, which is then swapped out to replace `Package.swift` during testing. This is both cumbersome and can easily lead to confusion and failing tests if dependencies fail to be added to both files, or if the two are not swapped before, or after, running tests.

Further motivation: this was originally intended to be in SwiftPM - [it was even implemented, but was removed after it stopped working and nobody fixed it.](https://github.com/apple/swift-package-manager/commit/34b7826cb586b0769ea5f60a7718d7de599ce27f)

## Proposed solution

The proposed solution is to add a new, optional, section to `Package.swift`, entitled `testDependencies`, which will only be read when running `swift test`.

## Detailed design

The actual implementation of `testDependencies` will be nearly identical to the existing `dependencies` section. All the same subtypes can be used in either, and the syntax is exactly the same for both. The sole difference is that when running `swift build`, the `testDependencies` is ignored completely, but when running `swift test`, all the packages listed in `testDependencies` are effectively added to the `dependencies` list before compiling and testing occurs.

`Package.swift` example with `testDependencies`:
```swift
import PackageDescription
let package = Package(
    name: "ExamplePackage",
    dependencies: [
      .Package(url: "https://github.com/user1/package1.git", majorVersion: 2),
      .Package(url: "https://github.com/user2/package2.git", majorVersion: 1, minor: 4),
    ],
    testDependencies: [
      .Package(url: "https://github.com/user3/test-package.git", majorVersion: 3),
    ]
)
```

In the above example example, running `swift build` would build the package `ExamplePackage`, including `user1/package1` and `user2/package2`, but **not** `user3/test-package`.

Running `swift test`, on the other hand, would build and test the package `ExamplePackage`, while including `user1/package1` and `user2/package2`, _and_ `user3/test-package`.

Effectively, `swift build` sees this:
```swift
import PackageDescription
let package = Package(
    name: "ExamplePackage",
    dependencies: [
      .Package(url: "https://github.com/user1/package1.git", majorVersion: 2),
      .Package(url: "https://github.com/user2/package2.git", majorVersion: 1, minor: 4),
    ]
)
```

While `swift test` sees:
```swift
import PackageDescription
let package = Package(
    name: "ExamplePackage",
    dependencies: [
      .Package(url: "https://github.com/user1/package1.git", majorVersion: 2),
      .Package(url: "https://github.com/user2/package2.git", majorVersion: 1, minor: 4),
      .Package(url: "https://github.com/user3/test-package.git", majorVersion: 3),
    ]
)
```

## Source compatibility

As this is an entirely additive, and optional, proposal, there would be zero source breakage.

## Effect on ABI stability

As mentioned [above](#source-compatibility), this is an entirely additive change that would only affect running tests, and thus would have no effect on ABI stability now or in the future.

## Effect on API resilience

Once added, this feature cannot be removed without breaking testing workflows, however, since this is a vital feature for moving forward with SwiftPM and standalone testing, it is unlikely to be removed once added.

## Alternatives considered

The names `devDependencies` and `localDependencies` were suggested as an alternative to `testDependencies`, however `testDependencies` was selected due to its un-ambiguous naming.


[test-example-1]: https://github.com/Quick/Quick/tree/500a6bbb171d9062a921770bb99ffc8ebe17c3cf
[test-example-2]: https://github.com/Carthage/Commandant/tree/0cd56446bdb8cb94acc26093c8322c47d9544fbe
[test-example-3]: https://github.com/Swinject/Swinject/tree/819ac12620bcbc94152e1b04433e64d968e55909
[test-example-4]: https://github.com/ReactiveCocoa/ReactiveSwift/tree/0fb7a16e8a1717432024e60d8a1a7c601bb0bd56