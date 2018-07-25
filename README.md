# Kyrie

[![Build status][travis-badge]][travis-badge-url]
[![Download](https://api.bintray.com/packages/alexjlockwood/maven/kyrie/images/download.svg)](https://bintray.com/alexjlockwood/maven/kyrie/_latestVersion)

Kyrie is a superset of Android's `VectorDrawable` and `AnimatedVectorDrawable` classes: it can do everything they can do and more.

![Screen capture of tool](art/screencap.gif)

## Motivation

`VectorDrawable`s are great because they provide density independence—they can be scaled arbitrarily on any device without loss of quality. `AnimatedVectorDrawable`s make them even more awesome, allowing us to animate specific properties of a `VectorDrawable` in a variety of ways.

However, these two classes also have several limitations:

- They can't be paused, resumed, or seeked.
- They can't be dynamically created at runtime (they must be inflated from a drawable resource).
- They only support a small subset of features that SVGs provide on the web.

Kyrie extends the functionality of `VectorDrawable`s and `AnimatedVectorDrawable`s by addressing these problems.

## Getting started

This section will give a brief overview of how to get started using the library. To play an animation using Kyrie, you first need to build a [`KyrieDrawable`][kyriedrawable]. There are two main ways to do this:

### Option #1: from an existing VD/AVD resource

With Kyrie, we can convert an existing VD/AVD resource into a `KyrieDrawable` with a single line:

```java
KyrieDrawable drawable = KyrieDrawable.create(context, R.drawable.my_vd_or_avd);
```

Once we do this, we can perform several actions that aren't currently possible using `AnimatedVectorDrawable`s, such as:

1.  Seek the animation using [`setCurrentPlayTime(long)`][kyriedrawable#setcurrentplaytime].
2.  Pause and resume the animation using [`pause()`][kyriedrawable#pause] and [`resume()`][kyriedrawable#resume].
3.  Listen for animation events using [`addListener(KyrieDrawable.Listener)`][kyriedrawable#addlistener].

### Option #2: programatically using a [`KyrieDrawable.Builder`][kyriedrawable#builder]

We can also build `KyrieDrawable`s at runtime using the builder pattern. `KyrieDrawable`s are similar to SVGs and `VectorDrawable`s in that they are tree-like structures built of [`Node`][node]s. As we build the tree, we can optionally assign [`Animation`][animation]s to the properties of each `Node` to create a more elaborate animation.

Here is a snippet of code from the [sample app][pathmorphfragment] that shows how we can create a path morphing animation this way:

```java
// Fill colors.
int hippoFillColor = ContextCompat.getColor(ctx, R.color.hippo);
int elephantFillColor = ContextCompat.getColor(ctx, R.color.elephant);
int buffaloFillColor = ContextCompat.getColor(ctx, R.color.buffalo);

// SVG path data objects.
PathData hippoPathData = PathData.parse(getString(R.string.hippo));
PathData elephantPathData = PathData.parse(getString(R.string.elephant));
PathData buffaloPathData = PathData.parse(getString(R.string.buffalo));

KyrieDrawable drawable =
    KyrieDrawable.builder()
        .viewport(409, 280)
        .child(
            PathNode.builder()
                .strokeColor(Color.BLACK)
                .strokeWidth(1f)
                .fillColor(
                    Animation.ofArgb(hippoFillColor, elephantFillColor).duration(300),
                    Animation.ofArgb(buffaloFillColor).startDelay(600).duration(300),
                    Animation.ofArgb(hippoFillColor).startDelay(1200).duration(300))
                .pathData(
                    Animation.ofPathMorph(
                            Keyframe.of(0, hippoPathData),
                            Keyframe.of(0.2f, elephantPathData),
                            Keyframe.of(0.4f, elephantPathData),
                            Keyframe.of(0.6f, buffaloPathData),
                            Keyframe.of(0.8f, buffaloPathData),
                            Keyframe.of(1, hippoPathData))
                       .duration(1500)))
            .build();
```

Or in Kotlin:

```kotlin
// Fill colors.
val hippoFillColor = ContextCompat.getColor(ctx, R.color.hippo)
val elephantFillColor = ContextCompat.getColor(ctx, R.color.elephant)
val buffaloFillColor = ContextCompat.getColor(ctx, R.color.buffalo)

// SVG path data objects.
val hippoPathData = getString(R.string.hippo)).asPathData();
val elephantPathData = getString(R.string.elephant).asPathData();
val buffaloPathData = getString(R.string.buffalo).asPathData();

val drawable =
    kyrieDrawable {
        viewport = size(409f, 280f)
        path {
            strokeColor(Color.BLACK)
            strokeWidth(1f)
            fillColor(
                Animation.ofArgb(hippoFillColor, elephantFillColor).duration(300),
                Animation.ofArgb(buffaloFillColor).startDelay(600).duration(300),
                Animation.ofArgb(hippoFillColor).startDelay(1200).duration(300)
            )
            pathData(
                Animation.ofPathMorph(
                    Keyframe.of(0f, hippoPathData),
                    Keyframe.of(0.2f, elephantPathData),
                    Keyframe.of(0.4f, elephantPathData),
                    Keyframe.of(0.6f, buffaloPathData),
                    Keyframe.of(0.8f, buffaloPathData),
                    Keyframe.of(1f, hippoPathData)
                ).duration(1500)
            )
        }
    }
```

## Useful links

- Check out [the blog post](https://www.androiddesignpatterns.com/2018/03/introducing-kyrie-animated-vector-drawables.html) for the motivation behind the library.
- Take a look at the [sample application](https://github.com/alexjlockwood/kyrie/tree/master/sample) for some example usages.
- Check out the [documentation][documentation] for a listing of all supported `Animation`s and `Node`s that can be used when constructing `KyrieDrawable`s programatically.
- Check out the [source code][kotlin-dsl-source-code] for a complete listing of the Kotlin-specific features Kyrie provides.

## Dependency

Add this to your root `build.gradle` file (_not_ your module's `build.gradle` file):

```gradle
allprojects {
    repositories {
        // ...
        jcenter()
    }
}
```

Then add the library to your module's `build.gradle` file:

```gradle
dependencies {
    // ...
    implementation 'com.github.alexjlockwood:kyrie:0.1.2'
}
```

If you want to use the additional Kotlin features provided by this library, such as the `KyrieDrawable` DSL and/or Kotlin extension functions, use this instead:

```gradle
dependencies {
    // ...
    implementation 'com.github.alexjlockwood:kyrie-kotlin-dsl:0.1.2'
}
```

## Compatibility

- **Minimum Android SDK**: Kyrie requires a minimum API level of 14.
- **Compile Android SDK**: Kyrie requires you to compile against API 27 or later.

  [travis-badge]: https://travis-ci.org/alexjlockwood/kyrie.svg?branch=master
  [travis-badge-url]: https://travis-ci.org/alexjlockwood/kyrie
  [kyriedrawable]: https://alexjlockwood.github.io/kyrie/com/github/alexjlockwood/kyrie/KyrieDrawable.html
  [node]: https://alexjlockwood.github.io/kyrie/com/github/alexjlockwood/kyrie/Node.html
  [animation]: https://alexjlockwood.github.io/kyrie/com/github/alexjlockwood/kyrie/Animation.html
  [pathmorphfragment]: https://github.com/alexjlockwood/kyrie/blob/master/sample/src/main/java/com/example/kyrie/PathMorphFragment.java
  [kyriedrawable#setcurrentplaytime]: https://alexjlockwood.github.io/kyrie/com/github/alexjlockwood/kyrie/KyrieDrawable.html#setCurrentPlayTime-long-
  [kyriedrawable#pause]: https://alexjlockwood.github.io/kyrie/com/github/alexjlockwood/kyrie/KyrieDrawable.html#pause--
  [kyriedrawable#resume]: https://alexjlockwood.github.io/kyrie/com/github/alexjlockwood/kyrie/KyrieDrawable.html#resume--
  [kyriedrawable#addlistener]: https://alexjlockwood.github.io/kyrie/com/github/alexjlockwood/kyrie/KyrieDrawable.html#addListener-com.github.alexjlockwood.kyrie.KyrieDrawable.Listener-
  [kyriedrawable#builder]: https://alexjlockwood.github.io/kyrie/com/github/alexjlockwood/kyrie/KyrieDrawable.Builder.html
  [documentation]: https://alexjlockwood.github.io/kyrie
  [kotlin-dsl-source-code]: https://github.com/alexjlockwood/kyrie/tree/master/kyrie-kotlin-dsl/src/main/java/com/github/alexjlockwood/kyrie
