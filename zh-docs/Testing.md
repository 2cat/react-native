---
id: testing
title: 测试
layout: docs
category: Guides
permalink: docs/testing.html
next:  embedded-app
---

## 运行测试用例和贡献

React Native 的仓库中有几个测试用例，你可以运行它们来确保你的 PR 不会引入导致项目恶化的内容。这些测试连续运行在 [Travis](http://docs.travis-ci.com/) 集成系统上面，并且会自动提交运行结果到你的 PR 。你也可以按下 cmd+U 在本地的 XCode 运行 IntegrationTest 和 UIExplorer 应用中的测试。你可以通过 `npm test` 命令运行 jest 测试。我们的测试至今未覆盖完全，所以，这么多的改变仍然需要有意义的人工验证，但是我们欢迎你帮助我们增加测试覆盖度！

## Jest 测试

[Jest](http://facebook.github.io/jest/) 是仅针对于 JS 的测试工具，运行在基于 node 的命令行上。测试代码位于 `__tests__` 目录，与要测试的代码文件位于同一个目录下面，
[Jest](http://facebook.github.io/jest/) tests are JS-only tests run on the command line with node.  The tests themselves live in the `__tests__` directories of the files they test, and there is a large emphasis on aggressively mocking out functionality that is not under test for failure isolation and maximum speed.  You can run the existing React Native jest tests with `npm test` from the react-native root, and we encourage you to add your own tests for any components you want to contribute to.  See [`getImageSource-test.js`](https://github.com/facebook/react-native/blob/master/Examples/Movies/__tests__/getImageSource-test.js) for a basic example.

## Integration Tests.

React Native provides facilities to make it easier to test integrated components that require both native and JS components to communicate across the bridge.  The two main components are `RCTTestRunner` and `RCTTestModule`.  `RCTTestRunner` sets up the ReactNative environment and provides facilities to run the tests as `XCTestCase`s in Xcode (`runTest:module` is the simplest method).  `RCTTestModule` is exported to JS via `NativeModules` as `TestModule`.  The tests themselves are written in JS, and must call `TestModule.markTestCompleted()` when they are done, otherwise the test will timeout and fail.  Test failures are primarily indicated by throwing an exception.  It is also possible to test error conditions with `runTest:module:initialProps:expectErrorRegex:` or `runTest:module:initialProps:expectErrorBlock:` which will expect an error to be thrown and verify the error matches the provided criteria.  See [`IntegrationTestHarnessTest.js`](https://github.com/facebook/react-native/blob/master/IntegrationTests/IntegrationTestHarnessTest.js) and [`IntegrationTestsTests.m`](https://github.com/facebook/react-native/blob/master/IntegrationTests/IntegrationTestsTests/IntegrationTestsTests.m) for example usage.

## Snapshot Tests

A common type of integration test is the snapshot test.  These tests render a component, and verify snapshots of the screen against reference images using `TestModule.verifySnapshot()`, using the [`FBSnapshotTestCase`](https://github.com/facebook/ios-snapshot-test-case) library behind the scenes.  Reference images are recorded by setting `recordMode = YES` on the `RCTTestRunner`, then running the tests.  Snapshots will differ slightly between 32 and 64 bit, and various OS versions, so it's recommended that you enforce tests are run with the correct configuration.  It's also highly recommended that all network data be mocked out, along with other potentially troublesome dependencies.  See [`SimpleSnapshotTest`](https://github.com/facebook/react-native/blob/master/IntegrationTests/SimpleSnapshotTest.js) for a basic example.
