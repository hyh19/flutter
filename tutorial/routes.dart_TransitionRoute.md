# TransitionRoute 类详解

## 概述

`TransitionRoute` 是 Flutter 路由系统中负责处理路由进入和退出过渡动画的抽象基类。它继承自 `OverlayRoute`，并实现了 `PredictiveBackRoute` 接口，为所有需要动画过渡的路由提供了统一的动画管理机制。

```dart 106:111:packages/flutter/lib/src/widgets/routes.dart
/// A route with entrance and exit transitions.
///
/// See also:
///
///  * [Route], which documents the meaning of the `T` generic type argument.
abstract class TransitionRoute<T> extends OverlayRoute<T> implements PredictiveBackRoute {
```

## 类继承关系

`TransitionRoute` 的继承层次结构如下：

- `Route<T>`：路由基类，定义了路由的基本生命周期和接口
- `OverlayRoute<T>`：管理路由在 `Overlay` 中的显示
- `TransitionRoute<T>`：添加了过渡动画支持
- `PredictiveBackRoute`：支持预测性返回手势（Android 预测性返回功能）

## 核心属性

### 动画完成状态

```dart 115:122:packages/flutter/lib/src/widgets/routes.dart
  /// This future completes only once the transition itself has finished, after
  /// the overlay entries have been removed from the navigator's overlay.
  ///
  /// This future completes once the animation has been dismissed. That will be
  /// after [popped], because [popped] typically completes before the animation
  /// even starts, as soon as the route is popped.
  Future<T?> get completed => _transitionCompleter.future;
  final Completer<T?> _transitionCompleter = Completer<T?>();
```

`completed` 是一个 `Future`，它只在过渡动画完全结束后才完成。这个 `Future` 的完成时机晚于 `popped`，因为 `popped` 通常在路由被弹出时立即完成，而动画可能才刚刚开始。

### 性能模式请求

```dart 124:130:packages/flutter/lib/src/widgets/routes.dart
  /// Handle to the performance mode request.
  ///
  /// When the route is animating, the performance mode is requested. It is then
  /// disposed when the animation ends. Requesting [DartPerformanceMode.latency]
  /// indicates to the engine that the transition is latency sensitive and to delay
  /// non-essential work while this handle is active.
  PerformanceModeRequestHandle? _performanceModeRequestHandle;
```

在路由动画进行时，系统会请求延迟敏感的性能模式，通知引擎延迟非必要工作，确保过渡动画的流畅性。

### 过渡时长

```dart 132:148:packages/flutter/lib/src/widgets/routes.dart
  /// {@template flutter.widgets.TransitionRoute.transitionDuration}
  /// The duration the transition going forwards.
  ///
  /// See also:
  ///
  /// * [reverseTransitionDuration], which controls the duration of the
  /// transition when it is in reverse.
  /// {@endtemplate}
  Duration get transitionDuration;

  /// {@template flutter.widgets.TransitionRoute.reverseTransitionDuration}
  /// The duration the transition going in reverse.
  ///
  /// By default, the reverse transition duration is set to the value of
  /// the forwards [transitionDuration].
  /// {@endtemplate}
  Duration get reverseTransitionDuration => transitionDuration;
```

- `transitionDuration`：路由进入时的动画时长（抽象属性，子类必须实现）
- `reverseTransitionDuration`：路由退出时的动画时长（默认等于 `transitionDuration`）

### 不透明度控制

```dart 150:156:packages/flutter/lib/src/widgets/routes.dart
  /// {@template flutter.widgets.TransitionRoute.opaque}
  /// Whether the route obscures previous routes when the transition is complete.
  ///
  /// When an opaque route's entrance transition is complete, the routes behind
  /// the opaque route will not be built to save resources.
  /// {@endtemplate}
  bool get opaque;
```

`opaque` 决定路由是否完全不透明。当不透明路由的进入动画完成后，后面的路由将不会被构建，以节省资源。

### 快照支持

```dart 158:171:packages/flutter/lib/src/widgets/routes.dart
  /// {@template flutter.widgets.TransitionRoute.allowSnapshotting}
  /// Whether the route transition will prefer to animate a snapshot of the
  /// entering/exiting routes.
  ///
  /// When this value is true, certain route transitions (such as the Android
  /// zoom page transition) will snapshot the entering and exiting routes.
  /// These snapshots are then animated in place of the underlying widgets to
  /// improve performance of the transition.
  ///
  /// Generally this means that animations that occur on the entering/exiting
  /// route while the route animation plays may appear frozen - unless they
  /// are a hero animation or something that is drawn in a separate overlay.
  /// {@endtemplate}
  bool get allowSnapshotting => true;
```

`allowSnapshotting` 允许路由在过渡时使用快照进行动画，以提高性能。当启用时，进入/退出路由会被快照，然后对快照进行动画，而不是直接动画实际的 widget。这意味着在路由动画期间，路由内部的动画可能会看起来冻结（除非是 Hero 动画或在独立 overlay 中绘制的内容）。

### 动画控制器和动画

```dart 182:207:packages/flutter/lib/src/widgets/routes.dart
  /// The animation that drives the route's transition and the previous route's
  /// forward transition.
  Animation<double>? get animation => _animation;
  Animation<double>? _animation;

  /// The animation controller that the route uses to drive the transitions.
  ///
  /// The animation itself is exposed by the [animation] property.
  @protected
  AnimationController? get controller => _controller;
  AnimationController? _controller;

  /// The animation for the route being pushed on top of this route. This
  /// animation lets this route coordinate with the entrance and exit transition
  /// of route pushed on top of this route.
  Animation<double>? get secondaryAnimation => _secondaryAnimation;
  final ProxyAnimation _secondaryAnimation = ProxyAnimation(kAlwaysDismissedAnimation);

  /// Whether to takeover the [controller] created by [createAnimationController].
  ///
  /// If true, this route will call [AnimationController.dispose] when the
  /// controller is no longer needed.
  /// If false, the controller should be disposed by whoever owned it.
  ///
  /// It defaults to `true`.
  bool willDisposeAnimationController = true;
```

- `animation`：驱动路由过渡的主动画，值从 0.0 到 1.0
- `controller`：动画控制器，用于控制动画的播放
- `secondaryAnimation`：辅助动画，用于协调当前路由与上层路由的过渡动画
- `willDisposeAnimationController`：是否在路由销毁时自动释放动画控制器

## 动画创建方法

### 创建动画控制器

```dart 226:242:packages/flutter/lib/src/widgets/routes.dart
  /// Called to create the animation controller that will drive the transitions to
  /// this route from the previous one, and back to the previous route from this
  /// one.
  ///
  /// The returned controller will be disposed by [AnimationController.dispose]
  /// if the [willDisposeAnimationController] is `true`.
  AnimationController createAnimationController() {
    assert(!debugTransitionCompleted(), 'Cannot reuse a $runtimeType after disposing it.');
    final Duration duration = transitionDuration;
    final Duration reverseDuration = reverseTransitionDuration;
    return AnimationController(
      duration: duration,
      reverseDuration: reverseDuration,
      debugLabel: debugLabel,
      vsync: navigator!,
    );
  }
```

创建用于驱动过渡动画的 `AnimationController`，使用 `navigator` 作为 `vsync`（垂直同步）提供者。

### 创建动画

```dart 244:251:packages/flutter/lib/src/widgets/routes.dart
  /// Called to create the animation that exposes the current progress of
  /// the transition controlled by the animation controller created by
  /// [createAnimationController()].
  Animation<double> createAnimation() {
    assert(!debugTransitionCompleted(), 'Cannot reuse a $runtimeType after disposing it.');
    assert(_controller != null);
    return _controller!.view;
  }
```

创建暴露过渡进度的动画对象，默认返回控制器的 `view`（一个只读的 `Animation<double>`）。

### 创建模拟动画

```dart 255:278:packages/flutter/lib/src/widgets/routes.dart
  /// Creates the simulation that drives the transition animation for this route.
  ///
  /// By default, this method returns null, indicating that the route doesn't
  /// use simulations, but initiates the transition by calling either
  /// [AnimationController.forward] or [AnimationController.reverse] with
  /// [transitionDuration] and the controller's curve.
  ///
  /// Subclasses can override this method to return a non-null [Simulation]. In
  /// this case, the [controller] will instead use the provided simulation to
  /// animate the transition using [AnimationController.animateWith] or
  /// [AnimationController.animateBackWith], and the [Simulation.x] is forwarded
  /// to the value of [animation]. The [controller]'s curve and
  /// [transitionDuration] are ignored.
  ///
  /// This method is invoked each time the navigator pushes or pops this route.
  /// The `forward` parameter indicates the direction of the transition: true when
  /// the route is pushed, and false when it is popped.
  Simulation? createSimulation({required bool forward}) {
    assert(
      transitionDuration >= Duration.zero,
      'The `duration` must be positive for a non-simulation animation. Received $transitionDuration.',
    );
    return null;
  }
```

子类可以重写此方法返回一个 `Simulation`，用于实现基于物理的动画效果（如弹性动画）。如果返回 `null`，则使用标准的 `forward`/`reverse` 方法。

## 生命周期方法

### install - 安装路由

```dart 323:334:packages/flutter/lib/src/widgets/routes.dart
  @override
  void install() {
    assert(!debugTransitionCompleted(), 'Cannot install a $runtimeType after disposing it.');
    _controller = createAnimationController();
    assert(_controller != null, '$runtimeType.createAnimationController() returned null.');
    _animation = createAnimation()..addStatusListener(_handleStatusChanged);
    assert(_animation != null, '$runtimeType.createAnimation() returned null.');
    super.install();
    if (_animation!.isCompleted && overlayEntries.isNotEmpty) {
      overlayEntries.first.opaque = opaque;
    }
  }
```

在路由安装时：

1. 创建动画控制器
2. 创建动画并添加状态监听器
3. 如果动画已经完成，设置 overlay entry 的不透明度

### didPush - 路由被推入

```dart 336:350:packages/flutter/lib/src/widgets/routes.dart
  @override
  TickerFuture didPush() {
    assert(
      _controller != null,
      '$runtimeType.didPush called before calling install() or after calling dispose().',
    );
    assert(!debugTransitionCompleted(), 'Cannot reuse a $runtimeType after disposing it.');
    super.didPush();
    _simulation = _createSimulationAndVerify(forward: true);
    if (_simulation == null) {
      return _controller!.forward();
    } else {
      return _controller!.animateWith(_simulation!);
    }
  }
```

当路由被推入时，启动进入动画。如果有模拟动画，使用 `animateWith`，否则使用 `forward`。

### didAdd - 路由被添加

```dart 352:361:packages/flutter/lib/src/widgets/routes.dart
  @override
  void didAdd() {
    assert(
      _controller != null,
      '$runtimeType.didPush called before calling install() or after calling dispose().',
    );
    assert(!debugTransitionCompleted(), 'Cannot reuse a $runtimeType after disposing it.');
    super.didAdd();
    _controller!.value = _controller!.upperBound;
  }
```

当路由被添加到导航器时（通常用于初始路由），直接将动画值设置为上限（1.0），跳过动画。

### didReplace - 路由被替换

```dart 363:374:packages/flutter/lib/src/widgets/routes.dart
  @override
  void didReplace(Route<dynamic>? oldRoute) {
    assert(
      _controller != null,
      '$runtimeType.didReplace called before calling install() or after calling dispose().',
    );
    assert(!debugTransitionCompleted(), 'Cannot reuse a $runtimeType after disposing it.');
    if (oldRoute is TransitionRoute) {
      _controller!.value = oldRoute._controller!.value;
    }
    super.didReplace(oldRoute);
  }
```

当路由被替换时，如果旧路由也是 `TransitionRoute`，则继承其动画控制器的当前值，实现平滑过渡。

### didPop - 路由被弹出

```dart 376:391:packages/flutter/lib/src/widgets/routes.dart
  @override
  bool didPop(T? result) {
    assert(
      _controller != null,
      '$runtimeType.didPop called before calling install() or after calling dispose().',
    );
    assert(!_transitionCompleter.isCompleted, 'Cannot reuse a $runtimeType after disposing it.');
    _result = result;
    _simulation = _createSimulationAndVerify(forward: false);
    if (_simulation == null) {
      _controller!.reverse();
    } else {
      _controller!.animateBackWith(_simulation!);
    }
    return super.didPop(result);
  }
```

当路由被弹出时，启动退出动画。保存返回结果，然后反向播放动画。

## 动画状态处理

### 状态变化处理

```dart 293:321:packages/flutter/lib/src/widgets/routes.dart
  void _handleStatusChanged(AnimationStatus status) {
    switch (status) {
      case AnimationStatus.completed:
        if (overlayEntries.isNotEmpty) {
          overlayEntries.first.opaque = opaque;
        }
        _performanceModeRequestHandle?.dispose();
        _performanceModeRequestHandle = null;
      case AnimationStatus.forward:
      case AnimationStatus.reverse:
        if (overlayEntries.isNotEmpty) {
          overlayEntries.first.opaque = false;
        }
        _performanceModeRequestHandle ??= SchedulerBinding.instance.requestPerformanceMode(
          ui.DartPerformanceMode.latency,
        );
      case AnimationStatus.dismissed:
        // We might still be an active route if a subclass is controlling the
        // transition and hits the dismissed status. For example, the iOS
        // back gesture drives this animation to the dismissed status before
        // removing the route and disposing it.
        if (!isActive) {
          navigator!.finalizeRoute(this);
          _popFinalized = true;
          _performanceModeRequestHandle?.dispose();
          _performanceModeRequestHandle = null;
        }
    }
  }
```

动画状态监听器处理不同状态：

- **completed**：动画完成时，设置 overlay entry 的不透明度，释放性能模式请求
- **forward/reverse**：动画进行时，设置 overlay entry 为透明，请求延迟敏感性能模式
- **dismissed**：动画被取消时，如果路由不再活跃，则完成路由的最终化

## 辅助动画（Secondary Animation）

辅助动画用于协调当前路由与上层路由的过渡效果，实现更复杂的动画交互。

### 更新辅助动画

```dart 422:496:packages/flutter/lib/src/widgets/routes.dart
  void _updateSecondaryAnimation(Route<dynamic>? nextRoute) {
    // There is an existing train hopping in progress. Unfortunately, we cannot
    // dispose current train hopping animation until we replace it with a new
    // animation.
    final VoidCallback? previousTrainHoppingListenerRemover = _trainHoppingListenerRemover;
    _trainHoppingListenerRemover = null;

    if (nextRoute is TransitionRoute<dynamic> &&
        canTransitionTo(nextRoute) &&
        nextRoute.canTransitionFrom(this)) {
      final Animation<double>? current = _secondaryAnimation.parent;
      if (current != null) {
        final Animation<double> currentTrain = (current is TrainHoppingAnimation
            ? current.currentTrain
            : current)!;
        final Animation<double> nextTrain = nextRoute._animation!;
        if (currentTrain.value == nextTrain.value || !nextTrain.isAnimating) {
          _setSecondaryAnimation(nextTrain, nextRoute.completed);
        } else {
          // Two trains animate at different values. We have to do train hopping.
          // There are three possibilities of train hopping:
          //  1. We hop on the nextTrain when two trains meet in the middle using
          //     TrainHoppingAnimation.
          //  2. There is no chance to hop on nextTrain because two trains never
          //     cross each other. We have to directly set the animation to
          //     nextTrain once the nextTrain stops animating.
          //  3. A new _updateSecondaryAnimation is called before train hopping
          //     finishes. We leave a listener remover for the next call to
          //     properly clean up the existing train hopping.
          TrainHoppingAnimation? newAnimation;
          void jumpOnAnimationEnd(AnimationStatus status) {
            if (!status.isAnimating) {
              // The nextTrain has stopped animating without train hopping.
              // Directly sets the secondary animation and disposes the
              // TrainHoppingAnimation.
              _setSecondaryAnimation(nextTrain, nextRoute.completed);
              if (_trainHoppingListenerRemover != null) {
                _trainHoppingListenerRemover!();
                _trainHoppingListenerRemover = null;
              }
            }
          }

          _trainHoppingListenerRemover = () {
            nextTrain.removeStatusListener(jumpOnAnimationEnd);
            newAnimation?.dispose();
          };
          nextTrain.addStatusListener(jumpOnAnimationEnd);
          newAnimation = TrainHoppingAnimation(
            currentTrain,
            nextTrain,
            onSwitchedTrain: () {
              assert(_secondaryAnimation.parent == newAnimation);
              assert(newAnimation!.currentTrain == nextRoute._animation);
              // We can hop on the nextTrain, so we don't need to listen to
              // whether the nextTrain has stopped.
              _setSecondaryAnimation(newAnimation!.currentTrain, nextRoute.completed);
              if (_trainHoppingListenerRemover != null) {
                _trainHoppingListenerRemover!();
                _trainHoppingListenerRemover = null;
              }
            },
          );
          _setSecondaryAnimation(newAnimation, nextRoute.completed);
        }
      } else {
        _setSecondaryAnimation(nextRoute._animation, nextRoute.completed);
      }
    } else {
      _setSecondaryAnimation(kAlwaysDismissedAnimation);
    }
    // Finally, we dispose any previous train hopping animation because it
    // has been successfully updated at this point.
    previousTrainHoppingListenerRemover?.call();
  }
```

这个方法实现了"列车跳跃"（train hopping）机制，用于在不同路由的动画之间平滑切换：

1. **直接切换**：如果当前动画和下一个动画的值相同，或下一个动画未在播放，直接切换
2. **列车跳跃**：如果两个动画值不同且都在播放，使用 `TrainHoppingAnimation` 在中间点切换
3. **等待完成**：如果无法跳跃，等待下一个动画完成后再切换

### 设置辅助动画

```dart 498:510:packages/flutter/lib/src/widgets/routes.dart
  void _setSecondaryAnimation(Animation<double>? animation, [Future<dynamic>? disposed]) {
    _secondaryAnimation.parent = animation;
    // Releases the reference to the next route's animation when that route
    // is disposed.
    disposed?.then((dynamic _) {
      if (_secondaryAnimation.parent == animation) {
        _secondaryAnimation.parent = kAlwaysDismissedAnimation;
        if (animation is TrainHoppingAnimation) {
          animation.dispose();
        }
      }
    });
  }
```

设置辅助动画的父动画，并在路由销毁时自动清理。

### 过渡协调方法

```dart 512:561:packages/flutter/lib/src/widgets/routes.dart
  /// Returns true if this route supports a transition animation that runs
  /// when [nextRoute] is pushed on top of it or when [nextRoute] is popped
  /// off of it.
  ///
  /// Subclasses can override this method to restrict the set of routes they
  /// need to coordinate transitions with.
  ///
  /// If true, and `nextRoute.canTransitionFrom()` is true, then the
  /// [ModalRoute.buildTransitions] `secondaryAnimation` will run from 0.0 - 1.0
  /// when [nextRoute] is pushed on top of this one. Similarly, if
  /// the [nextRoute] is popped off of this route, the
  /// `secondaryAnimation` will run from 1.0 - 0.0.
  ///
  /// If false, this route's [ModalRoute.buildTransitions] `secondaryAnimation` parameter
  /// value will be [kAlwaysDismissedAnimation]. In other words, this route
  /// will not animate when [nextRoute] is pushed on top of it or when
  /// [nextRoute] is popped off of it.
  ///
  /// Returns true by default.
  ///
  /// See also:
  ///
  ///  * [canTransitionFrom], which must be true for [nextRoute] for the
  ///    [ModalRoute.buildTransitions] `secondaryAnimation` to run.
  bool canTransitionTo(TransitionRoute<dynamic> nextRoute) => true;

  /// Returns true if [previousRoute] should animate when this route
  /// is pushed on top of it or when then this route is popped off of it.
  ///
  /// Subclasses can override this method to restrict the set of routes they
  /// need to coordinate transitions with.
  ///
  /// If true, and `previousRoute.canTransitionTo()` is true, then the
  /// previous route's [ModalRoute.buildTransitions] `secondaryAnimation` will
  /// run from 0.0 - 1.0 when this route is pushed on top of
  /// it. Similarly, if this route is popped off of [previousRoute]
  /// the previous route's `secondaryAnimation` will run from 1.0 - 0.0.
  ///
  /// If false, then the previous route's [ModalRoute.buildTransitions]
  /// `secondaryAnimation` value will be kAlwaysDismissedAnimation. In
  /// other words [previousRoute] will not animate when this route is
  /// pushed on top of it or when then this route is popped off of it.
  ///
  /// Returns true by default.
  ///
  /// See also:
  ///
  ///  * [canTransitionTo], which must be true for [previousRoute] for its
  ///    [ModalRoute.buildTransitions] `secondaryAnimation` to run.
  bool canTransitionFrom(TransitionRoute<dynamic> previousRoute) => true;
```

这两个方法控制路由之间的过渡协调：

- `canTransitionTo`：当前路由是否支持与下一个路由协调过渡
- `canTransitionFrom`：当前路由是否支持与前一个路由协调过渡

子类可以重写这些方法以限制需要协调过渡的路由集合。

## 预测性返回手势支持

`TransitionRoute` 实现了 `PredictiveBackRoute` 接口，支持 Android 的预测性返回功能，允许用户在返回手势过程中预览下一个路由。

### 开始返回手势

```dart 565:570:packages/flutter/lib/src/widgets/routes.dart
  @override
  void handleStartBackGesture({double progress = 0.0}) {
    assert(isCurrent);
    _controller?.value = progress;
    navigator?.didStartUserGesture();
  }
```

当用户开始返回手势时，将动画控制器的值设置为手势进度，并通知导航器用户手势已开始。

### 更新返回手势进度

```dart 572:580:packages/flutter/lib/src/widgets/routes.dart
  @override
  void handleUpdateBackGestureProgress({required double progress}) {
    // If some other navigation happened during this gesture, don't mess with
    // the transition anymore.
    if (!isCurrent) {
      return;
    }
    _controller?.value = progress;
  }
```

在手势进行过程中，持续更新动画控制器的值以反映手势进度。如果路由不再是当前路由，则忽略更新。

### 取消返回手势

```dart 582:585:packages/flutter/lib/src/widgets/routes.dart
  @override
  void handleCancelBackGesture() {
    _handleDragEnd(animateForward: true);
  }
```

当用户取消返回手势时，将动画向前播放到完成状态。

### 提交返回手势

```dart 587:590:packages/flutter/lib/src/widgets/routes.dart
  @override
  void handleCommitBackGesture() {
    _handleDragEnd(animateForward: false);
  }
```

当用户完成返回手势时，执行实际的弹出操作。

### 处理拖拽结束

```dart 592:623:packages/flutter/lib/src/widgets/routes.dart
  void _handleDragEnd({required bool animateForward}) {
    if (isCurrent) {
      if (animateForward) {
        // Typically, handleUpdateBackGestureProgress will have already
        // completed the animation. If not, animate to completion.
        if (!_controller!.isCompleted) {
          _controller!.forward();
        }
      } else {
        // This route is destined to pop at this point. Reuse navigator's pop.
        navigator?.pop();

        // The popping may have finished inline if already at the target destination.
        if (_controller?.isAnimating ?? false) {
          _controller!.reverse(from: _controller!.upperBound);
        }
      }
    }

    if (_controller?.isAnimating ?? false) {
      // Keep the userGestureInProgress in true state since AndroidBackGesturePageTransitionsBuilder
      // depends on userGestureInProgress.
      late final AnimationStatusListener animationStatusCallback;
      animationStatusCallback = (AnimationStatus status) {
        navigator?.didStopUserGesture();
        _controller!.removeStatusListener(animationStatusCallback);
      };
      _controller!.addStatusListener(animationStatusCallback);
    } else {
      navigator?.didStopUserGesture();
    }
  }
```

处理拖拽结束的逻辑：

- **向前动画**：如果手势被取消，将动画播放到完成状态
- **弹出路由**：如果手势被提交，调用 `navigator.pop()` 弹出路由
- **手势状态管理**：在动画完成后通知导航器用户手势已结束

## 资源清理

### dispose - 释放资源

```dart 627:639:packages/flutter/lib/src/widgets/routes.dart
  @override
  void dispose() {
    assert(!_transitionCompleter.isCompleted, 'Cannot dispose a $runtimeType twice.');
    assert(!debugTransitionCompleted(), 'Cannot dispose a $runtimeType twice.');
    _animation?.removeStatusListener(_handleStatusChanged);
    _performanceModeRequestHandle?.dispose();
    _performanceModeRequestHandle = null;
    if (willDisposeAnimationController) {
      _controller?.dispose();
    }
    _transitionCompleter.complete(_result);
    super.dispose();
  }
```

在路由销毁时：

1. 移除动画状态监听器
2. 释放性能模式请求句柄
3. 根据 `willDisposeAnimationController` 决定是否释放动画控制器
4. 完成过渡完成器，传递路由返回结果
5. 调用父类的 `dispose` 方法

## 调试支持

```dart 209:224:packages/flutter/lib/src/widgets/routes.dart
  /// Returns true if the transition has completed.
  ///
  /// It is equivalent to whether the future returned by [completed] has
  /// completed.
  ///
  /// This method only works if assert is enabled. Otherwise it always returns
  /// false.
  @protected
  bool debugTransitionCompleted() {
    bool disposed = false;
    assert(() {
      disposed = _transitionCompleter.isCompleted;
      return true;
    });
    return disposed;
  }
```

```dart 641:645:packages/flutter/lib/src/widgets/routes.dart
  /// A short description of this route useful for debugging.
  String get debugLabel => objectRuntimeType(this, 'TransitionRoute');

  @override
  String toString() => '${objectRuntimeType(this, 'TransitionRoute')}(animation: $_controller)';
```

提供了调试方法用于检查过渡是否完成，以及 `toString` 方法用于调试输出。

## 总结

`TransitionRoute` 是 Flutter 路由系统中处理过渡动画的核心抽象类，它提供了：

1. **完整的动画生命周期管理**：从创建到销毁的完整流程
2. **灵活的动画控制**：支持标准动画和基于物理的模拟动画
3. **辅助动画协调**：实现复杂路由之间的动画协调
4. **预测性返回支持**：支持 Android 预测性返回手势
5. **性能优化**：通过快照和性能模式请求优化动画性能

子类如 `PageRoute`、`ModalRoute` 等都继承自 `TransitionRoute`，实现了具体的过渡效果（如淡入淡出、滑动等）。
