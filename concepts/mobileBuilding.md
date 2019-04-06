---
layout: tutorial
title: Mobile
---

The `engo` game engine, and Go in general, has two ways to build for a
mobile device: binding or building.

First, you'll need to [install Go Mobile] (https://github.com/golang/go/wiki/Mobile).

## Using gomobile build

To use gomobile build, simply go to the folder that you have your main package
and run

```go
gomobile build
```

A few caveats:

* All your asset files must be in the asset folder to be accessed
* You only have access to touch input and ogl. So you can't use things like
Facebook or AdMob's sdks. This includes things like gradle and pro-guard
* You'll get a generic AndroidManifest, unless you include one yourself in the
same folder as your main package

## Using gomobile bind

Using bindings allows access to and from the Android system, and you can just
build into your glue whatever functions you need to be called or to call.
However, it is more complicated to initially setup. This is taken from [here]
(https://github.com/Noofbiz/MouseTests) if you want to look at the code while
reading this.

#### Setting up your game library
First thing to notice is the file game.go here. It's an (almost) exact copy of main.go, except it's a library instead of a main package. You'll need to add

```go
//+build !mobilebind
```
to the top (before package declaration) of your main, and then

```go
//+build mobilebind
```

to the top of the library. The other differences are that when loading a file, you can't keep assets in an asset folder for binding, so instead you'll need to use go-bindata to turn your assets into binary data. Then to load the files use

```go
	b, err := assets.Asset("icon.png")
	if err != nil {
		log.Panic("no such icon")
	}
	engo.Files.LoadReaderData("icon.png", bytes.NewReader(b))
```
and when you're starting a game, your main becomes a start function that can be called from the androidglue package

```go
func Start(width, height int) {
	opts := engo.RunOptions{
		Title:        "Mouse Demo",
		Width:        1024,
		Height:       640,
		MobileWidth:  width,
		MobileHeight: height,
	}
	engo.Run(opts, &DefaultScene{})
}
```
The additional options, as well as the width and height being passed in, are for Android to tell your game the actual dimensions of the screen.

#### Setting up the android glue
Next, you'll need a package that handles communication between your Go library and Android. That's where the package androidglue comes in! androidglue.go just contains exported functions that Android can call to communicate with your game, such as tell it when to start, stop, or when touch events occur. You could potentially add more to this, for handling when to trigger a handler for the Facebook SDK or to preload an intersitial ad.

```go
//+build mobilebind

package androidglue

import (
	"github.com/Noofbiz/mousetests"

	"github.com/EngoEngine/engo"
)

var running bool

func Start(width, height int) {
	running = true
	mousetests.Start(width, height)
}

func Update() {
	engo.RunIteration()
}

func IsRunning() bool {
	return running
}

func Touch(x, y, action int) {
	engo.TouchEvent(x, y, action)
}

func Stop() {
	running = false
	engo.MobileStop()
}

```

And, that's it for the Go side of things. All that's left for Go is to use (inside the androidglue folder)

```
gomobile bind -target=android -tags=mobilebind
```

to compile the .aar library to import into Android Studio.

#### Setting up Android side

To setup the Android side, I used Android Studio. There's also a way to do it via gradle and the command line, but I don't know the exact details on doing that at the moment.

Start by creating a new project in Android Studio. Phone and Tablet with minimum SDK set to 15 and with no starting activity. In the AndroidManifest.xml, add a main activity. Mine ends up looking like this

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"

    package="io.engo.mousetests">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity
            android:name=".MainActivity"
            android:screenOrientation="landscape">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

and then to styles.xml I added the following to give the full screen to the gl surface view

```xml
		<item name="windowNoTitle">true</item>
        <item name="android:windowFullscreen">true</item>
        <item name="android:windowContentOverlay">@null</item>
```

Next we have to add a main activity, by creating a MainActivity.java file.

```java
package io.engo.mousetests;

import android.opengl.GLSurfaceView;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    private GLSurfaceView glSurfaceView() {
        return (GLSurfaceView)this.findViewById(R.id.glview);
    }

    @Override
    protected void onPause() {
        super.onPause();
        this.glSurfaceView().onPause();
    }

    @Override
    protected void onResume() {
        super.onResume();
        this.glSurfaceView().onResume();
    }
}
```

as well as a style xml for it. Here's activity_main.xml in the layout folder

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="io.engo.mousetests.MainActivity"
    android:keepScreenOn="true" >
    <io.engo.mousetests.EngoGLSurfaceView
        android:id="@+id/glview"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true" />
</RelativeLayout>
```

and finally, create the glSurfaceView. EngoGLSurfaceView.java

```java
package io.engo.mousetests;

import android.content.Context;
import android.content.res.Resources;
import android.opengl.GLSurfaceView;
import android.util.Log;
import android.util.AttributeSet;
import android.view.MotionEvent;

import javax.microedition.khronos.egl.EGLConfig;
import javax.microedition.khronos.opengles.GL10;

import androidglue.*;

public class EngoGLSurfaceView extends GLSurfaceView {

    private class EngoRenderer implements Renderer {

        private boolean mErrored;

        @Override
        public void onDrawFrame(GL10 gl) {
            if (mErrored) {
                return;
            }
            try {
                Androidglue.update();
            } catch (Exception e) {
                Log.e("Go Error", e.toString());
                mErrored = true;
            }
        }

        @Override
        public void onSurfaceCreated(GL10 gl, EGLConfig config) {
        }

        @Override
        public void onSurfaceChanged(GL10 gl, int width, int height) {
        }
    }

    public EngoGLSurfaceView(Context context) {
        super(context);
        initialize();
    }

    public EngoGLSurfaceView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initialize();
    }

    private void initialize() {
        setEGLContextClientVersion(2);
        setEGLConfigChooser(8, 8, 8, 8, 0, 0);
        setRenderer(new EngoRenderer());
    }

    @Override
    public void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);

        try {
            if (!Androidglue.isRunning()) {
                Androidglue.start(Resources.getSystem().getDisplayMetrics().widthPixels, Resources.getSystem().getDisplayMetrics().heightPixels);
            }
        } catch (Exception e) {
            Log.e("Go Error", e.toString());
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent e) {
        for (int i = 0; i < e.getPointerCount(); i++) {
            Androidglue.touch((int)e.getX(i), (int)e.getY(i), e.getActionMasked());
        }
        return true;
    }

    @Override
    public void onPause() {
        try {
            if (Androidglue.isRunning()) {
                Androidglue.stop();
            }
        } catch (Exception e) {
            Log.e("Go Error", e.toString());
        }
    }
}
```

Now the last part is to import the aar created by gomobile bind

File -> New -> New Module...

Pick Import .JAR/.AAR Package

Navigate to the .AAR package you created and import it

Then add it as a dependency to your android project

File -> Project Structure -> app -> on the dependencies tab

Then click the +, and under module dependencies you'll find your imported androidglue library. Everything should build nicely and you can then run it in an emulator or a device!
