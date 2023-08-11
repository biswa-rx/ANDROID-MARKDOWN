# BlurView

![BlurView](https://user-images.githubusercontent.com/1433500/174389657-f52837db-005b-4a68-b9c6-ce196fa03395.jpg)

Dynamic iOS-like blur for Android Views. Includes library and small example project.

BlurView can be used as a regular FrameLayout. It blurs its underlying content and draws it as a background for its children. The children of the BlurView are not blurred. BlurView redraws its blurred content when changes in view hierarchy are detected (draw() called). It honors its position and size changes, including view animation and property animation.


### How to use

_Gradle (app: module)_
```kotlin
implementation ("com.github.Dimezis:BlurView:version-2.0.3")
```

_Setting Gradle_
```kotlin
repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://www.jitpack.io" )
        }
    }
```

```xml
<eightbitlab.com.blurview.BlurView
        android:id="@+id/blurView"
        android:layout_width="300dp"
        android:layout_height="100dp"
        app:blurOverlayColor="@color/white"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.5">

        <!--Any child View here, TabLayout for example. This View will NOT be blurred -->

    </eightbitlab.com.blurview.BlurView>
```

```java
BlurView blurView = findViewById(R.id.blurView);

        float radius = 20f;

        View decorView = getWindow().getDecorView();
        // ViewGroup you want to start blur from. Choose root as close to BlurView in hierarchy as possible.
        ViewGroup rootView = (ViewGroup) decorView.findViewById(R.id.layout_parent);

        // Optional:
        // Set drawable to draw in the beginning of each blurred frame.
        // Can be used in case your layout has a lot of transparent space and your content
        // gets a too low alpha value after blur is applied.
        Drawable windowBackground = decorView.getBackground();

        blurView.setupWith(rootView, new RenderScriptBlur(this)) // or RenderEffectBlur
                .setFrameClearDrawable(windowBackground) // Optional
                .setBlurRadius(radius);


        blurView.setOutlineProvider(ViewOutlineProvider.BACKGROUND);
        blurView.setClipToOutline(true);
        blurView.setAlpha(0.8f);
        blurView.setBackgroundColor(Color.BLACK);
```


