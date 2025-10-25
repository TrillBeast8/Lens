# INMO AR Development Ultimate Guide
## Complete Reference for INMO AIR2 AR Glasses Development

---

## Table of Contents
1. [Platform Overview](#platform-overview)
2. [Hardware Specifications](#hardware-specifications)
3. [Development Setup](#development-setup)
4. [Android Native Development](#android-native-development)
5. [Unity Development](#unity-development)
6. [Input Methods & Interactions](#input-methods--interactions)
7. [AR Service Implementation](#ar-service-implementation)
8. [Best Practices & Optimization](#best-practices--optimization)
9. [Publishing Requirements](#publishing-requirements)

---

## Platform Overview

### What is INMO AIR2?
INMO AIR2 is an AR glasses platform that provides:
- **6DOF spatial positioning** (SLAM-based tracking)
- **Dual input methods**: INMO Ring2 controller & touchpad on glasses temple
- **Android-based OS** (deeply customized Android system)
- **SDK Support**: Native Android & Unity platforms

### Current Capabilities
✅ **Supported:**
- 6DOF spatial positioning
- Custom UI rendering
- OpenGL ES 2.0+ rendering
- Camera access
- Sensor integration
- Ring controller input
- Touchpad gestures

❌ **Not Yet Supported:**
- Loop closure detection
- Plane detection
- Occlusion detection
- Light estimation
- Environment understanding

### System Requirements
- **Minimum Android API**: 28 (Android 9.0)
- **Glass Firmware**: Version 2.4+
- **Development Tools**:
  - Android Studio (Native)
  - Unity 2020.3 LTS+ (Unity)
  - JDK 11

---

## Hardware Specifications

### INMO AIR2 Glasses
- **Display**: Optical projection display
- **Camera**: RGB camera (ID: 0)
- **Sensors**: IMU, gyroscope, accelerometer
- **Connectivity**: Bluetooth HID (for Ring2)
- **Touchpad**: Left and right temple touchpads

### INMO Ring2 Controller
- **Connection**: Bluetooth HID profile (Profile ID: 4)
- **Buttons**: OK, Up, Down, Left, Right, Back
- **Modes**:
  1. Remote Control (default)
  2. Mouse Mode
  3. Ray Mode (3DOF orientation tracking)

---

## Development Setup

### Android Native Development

#### 1. Repository Setup
```groovy
// In settings.gradle or project-level build.gradle
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        jcenter()
        maven { url 'https://jitpack.io' }
        // INMO SDK Repository
        maven { url 'https://gitee.com/inmolens/inmo-ar-sdk/raw/master' }
    }
}
```

#### 2. Add SDK Dependency
```groovy
// In app/build.gradle
dependencies {
    // INMO AR SDK
    implementation 'com.inmo:inmo_arsdk:0.0.1'

    // Standard Android dependencies
    implementation 'androidx.appcompat:appcompat:1.4.1'
    implementation 'com.google.android.material:material:1.5.0'
}

android {
    namespace 'com.yourcompany.yourapp'
    compileSdk 32

    defaultConfig {
        applicationId "com.yourcompany.yourapp"
        minSdk 28  // CRITICAL: Must be 28 or higher
        targetSdk 32
        versionCode 1
        versionName "1.0"
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

#### 3. Clone Sample Projects
```bash
# AR SDK Sample (6DOF tracking with OpenGL)
git clone https://gitee.com/inmolens/inmo-ar-sdk-sample.git

# UI SDK Sample (Input handling examples)
git clone https://gitee.com/inmolens/inmo-ui-sdk-sample.git
```

### Unity Development

#### 1. Software Requirements
- **Unity Hub** + **Unity 2020.3 LTS** or higher
- **Android Support Module** (install via Unity Hub)
- **Visual Studio 2019**

#### 2. NDK Version Requirements
| Unity Version | Required NDK |
|--------------|--------------|
| 2020.3 LTS   | r19          |
| 2021.3 LTS   | r21d         |

#### 3. Import SDK
```bash
# Clone Unity SDK Sample
git clone https://gitee.com/inmolens/inmo-sdk-sample-unity.git
```

**Import Steps:**
1. Create new Unity project or open existing
2. Switch platform to Android: `File → Build Settings → Android`
3. Drag `.unitypackage` into Project window, or
4. `Assets → Import Package → Custom Package`

#### 4. Build Settings Configuration

**Player Settings (Critical):**
```
Minimum API Level: 28
Scripting Backend: IL2CPP (recommended for performance)
Target Architectures: ARMv7 (uncheck ARM64 to reduce size)
```

**Rendering Settings:**
```
Camera FOV: 13.4 (recommended for AR)
Camera Near: 0.1
Camera Far: 15
Render Shadows: OFF
Directional Light Shadow Type: No Shadows
```

**⚠️ Important:** Uncheck "Optimize Frame Rate" in build settings!

---

## Android Native Development

### Core Architecture

#### Basic Activity Structure
```java
import com.arglasses.arsdk.ArServiceSession;

public class MainActivity extends Activity {
    GLView mView;
    ArServiceSession mArSession;
    GlRenderer mRenderer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Initialize AR Session
        mArSession = new ArServiceSession(getApplication());
        mArSession.create();

        // Setup renderer and view
        mRenderer = new GlRenderer(mArSession);
        mView = new GLView(getApplication(), mRenderer);
        setContentView(mView);
    }

    @Override
    protected void onPause() {
        super.onPause();
        mView.onPause();
        mArSession.pause();  // Release camera but keep data
    }

    @Override
    protected void onResume() {
        super.onResume();
        mView.onResume();
        mArSession.resume();  // Restart tracking
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mArSession.destroy();  // Clean up all resources
    }
}
```

### ArServiceSession API Reference

#### Constructor
```java
/**
 * Create ArServiceSession bound to context
 * @param context Application context
 */
public ArServiceSession(Context context)
```

#### Lifecycle Methods
```java
/**
 * Get AR Service version
 * @return Version string
 */
public String getVersion()

/**
 * Create and start AR Service
 * @return true if successful, false otherwise
 */
public boolean create()

/**
 * Destroy session and clean data
 * Service may continue if other processes are using it
 */
public void destroy()

/**
 * Pause tracking, release camera but keep data
 */
public void pause()

/**
 * Resume tracking and camera
 */
public void resume()
```

#### Pose & Matrix Methods
```java
/**
 * Get device pose relative to world coordinates
 * Data format:
 * [0-2]: Position vector (x, y, z)
 * [3-6]: Rotation quaternion (w, x, y, z)
 * @return float array of 7 elements
 */
public float[] getPose()

/**
 * Get projection matrix for rendering
 * @param width Viewport width in pixels
 * @param height Viewport height in pixels
 * @param near Near clipping plane (meters)
 * @param far Far clipping plane (meters)
 * @return 4x4 projection matrix (16 floats)
 */
public float[] getProjectionMatrix(int width, int height, float near, float far)

/**
 * Get view matrix for rendering
 * @return 4x4 view matrix (16 floats)
 */
public float[] getViewMatrix()
```

### OpenGL Rendering Example

**Key Points from GlRenderer.java:**
```java
public class GlRenderer implements GLSurfaceView.Renderer {
    private ArServiceSession mArSession;
    private float[] mMVPMatrix = new float[16];
    private float[] mViewMatrix = new float[16];
    private float[] mModelMatrix = new float[16];
    private float[] mProjectionMatrix = new float[16];

    @Override
    public void onDrawFrame(GL10 gl) {
        // Get view and projection from AR service
        mViewMatrix = mArSession.getViewMatrix();
        mProjectionMatrix = mArSession.getProjectionMatrix(width, height, 0.1f, 100.0f);

        // Apply transformations
        Matrix.setIdentityM(mModelMatrix, 0);
        Matrix.multiplyMM(mMVPMatrix, 0, mViewMatrix, 0, mModelMatrix, 0);
        Matrix.multiplyMM(mMVPMatrix, 0, mProjectionMatrix, 0, mMVPMatrix, 0);

        // Render your 3D content
        drawCube();
    }
}
```

### Camera Access
```java
private Camera camera;
private int INMO_AIR2_RGB_CAMERA_ID = 0;  // RGB camera ID

// Open camera
camera = Camera.open(INMO_AIR2_RGB_CAMERA_ID);
```

### Sensor Access
Use standard Android sensor APIs:
```java
SensorManager sensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
Sensor accelerometer = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
```
**Reference:** https://developer.android.google.cn/docs

---

## Unity Development

### Core Architecture

#### ARPoseService Class

**File Location:** `Assets/InmoUnitySdk/SDK/Scripts/AR/ARPoseService.cs`

```csharp
using inmo.unity.sdk;
using UnityEngine;

public class ARCamera : MonoBehaviour
{
    private ARPoseService _arService;
    private ARPoseService.ARPose _arPose;

    [SerializeField] private float _fieldOfView = 13.4f;  // Recommended FOV

    private void Start()
    {
        // Initialize AR Service
        _arService = new ARPoseService(Camera.main, _fieldOfView);
        _arService.Start();
    }

    private void Update()
    {
        // Get current AR pose
        if (_arService != null && _arService.isARPoseServiceRuning)
        {
            _arPose = _arService.GetARPose();
        }
    }

    private void LateUpdate()
    {
        // Apply pose to camera
        if (_arPose != null)
        {
            Camera.main.transform.position = _arPose.position;
            Camera.main.transform.rotation = _arPose.rotation;
        }
    }

    private void OnDestroy()
    {
        if (_arService != null)
        {
            _arService.Stop();
        }
    }
}
```

#### ARPoseService API Reference

```csharp
namespace inmo.unity.sdk
{
    public class ARPoseService
    {
        public struct ARPose
        {
            public Vector3 position;      // Position in Unity coordinates
            public Quaternion rotation;   // Rotation in Unity coordinates
        }

        public bool isARPoseServiceRuning;  // Service status flag

        // Constructor
        public ARPoseService(Camera camera, float fov = 13.9f)

        // Lifecycle
        public void Start()              // Create and start service
        public void Stop()               // Destroy service
        public void Pause()              // Pause tracking
        public void Resume()             // Resume tracking

        // Data Access
        public string GetVersion()       // Get AR service version
        public ARPose GetARPose()        // Get pose (Unity coordinates)
        public ARPose GetOriginARPose()  // Get pose (raw coordinates)
        public float[] GetOriginSixDofData()  // Get raw 6DOF data
    }
}
```

**Coordinate System Conversion:**
Unity automatically handles coordinate conversion:
```csharp
// Raw data → Unity coordinates
arPose.position.x = rawData[0];
arPose.position.y = rawData[2];  // Y and Z swapped
arPose.position.z = rawData[1];

arPose.rotation.x = -rawData[3];  // Inverted
arPose.rotation.y = -rawData[5];  // Inverted & swapped
arPose.rotation.z = -rawData[4];  // Inverted & swapped
arPose.rotation.w = rawData[6];
```

### RingPoseService (3DOF Ring Tracking)

**File Location:** `Assets/InmoUnitySdk/SDK/Scripts/AR/RingPoseService.cs`

```csharp
using inmo.unity.sdk;

public class RingController : MonoBehaviour
{
    private RingPoseService _ringService;

    private void Start()
    {
        _ringService = new RingPoseService();
        _ringService.Start();
    }

    private void Update()
    {
        if (_ringService.isRingThreeDofServiceRuning)
        {
            Quaternion ringRotation = _ringService.GetRingPose();
            // Apply to ray or cursor object
            transform.rotation = ringRotation;
        }
    }

    // Reset ring orientation (useful for recalibration)
    public void RecalibrateRing()
    {
        _ringService.ResetRingPose();
    }

    private void OnDestroy()
    {
        _ringService.Stop();
    }
}
```

#### RingPoseService API Reference

```csharp
public class RingPoseService
{
    public bool isRingThreeDofServiceRuning;

    public RingPoseService()           // Constructor
    public void Start()                // Start 3DOF tracking
    public void Stop()                 // Stop tracking
    public void Pause()                // Pause tracking
    public void Resume()               // Resume tracking

    public Quaternion GetRingPose()    // Get ring orientation (Unity coords)
    public void ResetRingPose()        // Reset/calibrate ring orientation
    public Quaternion GetOriginRingPose()  // Get raw orientation
    public float[] GetRingThreeDofData()   // Get raw 3DOF data
}
```

---

## Input Methods & Interactions

### INMO Ring2 Controller

#### Connection Check (Android Native)
```java
private boolean isBluetoothHidConnected() {
    BluetoothAdapter adapter = BluetoothAdapter.getDefaultAdapter();
    if (adapter.isEnabled()) {
        int state = adapter.getProfileConnectionState(4);  // HID profile
        return state == BluetoothProfile.STATE_CONNECTED;
    }
    return false;
}
```

#### Key Event Handling

**Android (Native Java):**
```java
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    switch (keyCode) {
        case KeyEvent.KEYCODE_ENTER:        // 66 - OK button
            handleConfirm();
            break;
        case KeyEvent.KEYCODE_DPAD_UP:      // 19 - Up
            handleUp();
            break;
        case KeyEvent.KEYCODE_DPAD_DOWN:    // 20 - Down
            handleDown();
            break;
        case KeyEvent.KEYCODE_DPAD_LEFT:    // 21 - Left
            handleLeft();
            break;
        case KeyEvent.KEYCODE_DPAD_RIGHT:   // 22 - Right
            handleRight();
            break;
        case KeyEvent.KEYCODE_BACK:         // 4 - Back (single click)
            handleBack();
            break;
        case KeyEvent.KEYCODE_HOME:         // 3 - Home (double-click back)
            handleHome();
            break;
        case 289:  // Single finger long press
            handleLongPress1();
            break;
        case 290:  // Special key 1
            handleSpecial1();
            break;
        case 291:  // Special key 2
            handleSpecial2();
            break;
        case 292:  // Long press special
            handleLongPress2();
            break;
    }
    return super.onKeyDown(keyCode, event);
}
```

**Unity (C#):**
```csharp
private void Update()
{
    // OK Button
    if (Input.GetKeyDown(KeyCode.Return))  // 13
    {
        HandleConfirm();
    }

    // Directional Buttons
    if (Input.GetKeyDown(KeyCode.UpArrow))     // 273
        HandleUp();
    if (Input.GetKeyDown(KeyCode.DownArrow))   // 274
        HandleDown();
    if (Input.GetKeyDown(KeyCode.LeftArrow))   // 276
        HandleLeft();
    if (Input.GetKeyDown(KeyCode.RightArrow))  // 275
        HandleRight();

    // Back Button
    if (Input.GetKeyDown(KeyCode.Escape))      // 27
    {
        HandleBack();
    }
}
```

#### Complete Key Code Reference

**Android Native:**
| Button | Function | KeyCode | Constant |
|--------|----------|---------|----------|
| OK (single/double/long) | Confirm | 66 | KEYCODE_ENTER |
| Up (single/double/long) | Direction Up | 19 | KEYCODE_DPAD_UP |
| Down (single/double/long) | Direction Down | 20 | KEYCODE_DPAD_DOWN |
| Left (single/double/long) | Direction Left | 21 | KEYCODE_DPAD_LEFT |
| Right (single/double/long) | Direction Right | 22 | KEYCODE_DPAD_RIGHT |
| Back (single) | Exit/Return | 4 | KEYCODE_BACK |
| Back (double) | Home | 3 | KEYCODE_HOME |
| Single finger long press | Custom | 289 | - |
| Double finger long press | Custom | 290 | - |

**Unity:**
| Button | Function | KeyCode | Unity Constant |
|--------|----------|---------|----------------|
| OK | Confirm | 13 | KeyCode.Return |
| Up | Direction Up | 273 | KeyCode.UpArrow |
| Down | Direction Down | 274 | KeyCode.DownArrow |
| Left | Direction Left | 276 | KeyCode.LeftArrow |
| Right | Direction Right | 275 | KeyCode.RightArrow |
| Back | Exit/Return | 27 | KeyCode.Escape |

### Ring2 Operating Modes

#### 1. Remote Control Mode (Default)
- Key events only sent when buttons pressed
- No continuous data
- **OK Button:** KeyCode.Return

#### 2. Mouse Mode
```
Long press Back button to switch modes
```
- Ring acts as HID mouse
- Sensor motion controls cursor X/Y
- System handles mouse events automatically
- **OK Button:** Acts as LEFT MOUSE BUTTON (not KeyCode.Return!)

#### 3. Ray Mode (3DOF)
```
Long press Back button to switch modes
```
- Provides ring orientation as quaternion
- Use for 3D pointer/ray casting
- Buttons work normally
- Access via `RingPoseService` in Unity

**Unity Implementation:**
```csharp
public class RayController : MonoBehaviour
{
    private RingPoseService ringService;
    public LineRenderer rayLine;
    public float rayLength = 10f;

    void Start()
    {
        ringService = new RingPoseService();
        ringService.Start();
    }

    void Update()
    {
        if (ringService.isRingThreeDofServiceRuning)
        {
            // Get ring orientation
            Quaternion ringRotation = ringService.GetRingPose();

            // Calculate ray direction
            Vector3 rayDirection = ringRotation * Vector3.forward;

            // Raycast from camera
            Ray ray = new Ray(Camera.main.transform.position, rayDirection);
            RaycastHit hit;

            if (Physics.Raycast(ray, out hit, rayLength))
            {
                // Hit something
                SelectObject(hit.collider.gameObject);
            }

            // Visualize ray
            rayLine.SetPosition(0, Camera.main.transform.position);
            rayLine.SetPosition(1, Camera.main.transform.position + rayDirection * rayLength);
        }

        // Recalibrate on button press
        if (Input.GetKeyDown(KeyCode.Return))
        {
            ringService.ResetRingPose();
        }
    }
}
```

### Touchpad Interaction

#### Android Native Implementation
```java
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    switch (keyCode) {
        case 66:  // Single tap (one or two fingers)
            handleTap();
            break;
        case 4:   // Double tap
            handleDoubleTap();
            break;
        case 289: // Single finger long press
            handleLongPress();
            break;
        case 290: // Two finger long press
            handleTwoFingerLongPress();
            break;
        case 22:  // Swipe forward (single finger)
            handleSwipeForward();
            break;
        case 21:  // Swipe back (single finger)
            handleSwipeBack();
            break;
        case 19:  // Swipe up (single finger)
            handleSwipeUp();
            break;
        case 20:  // Swipe down (single finger)
            handleSwipeDown();
            break;
        // Two-finger swipes send continuous events
    }
    return super.onKeyDown(keyCode, event);
}
```

#### Unity Touchpad Handling
```csharp
private void Update()
{
    // Single tap (confirm)
    if (Input.GetKeyDown(KeyCode.Return))  // 13
    {
        OnTap();
    }

    // Double tap (back)
    if (Input.GetKeyDown(KeyCode.Escape))  // 27
    {
        OnDoubleTap();
    }

    // Swipe gestures
    if (Input.GetKeyDown(KeyCode.UpArrow))
        OnSwipeUp();
    if (Input.GetKeyDown(KeyCode.DownArrow))
        OnSwipeDown();
    if (Input.GetKeyDown(KeyCode.LeftArrow))
        OnSwipeBack();
    if (Input.GetKeyDown(KeyCode.RightArrow))
        OnSwipeForward();
}
```

#### Complete Touchpad Key Reference

**Right Touchpad (Primary):**
| Gesture | Function | Android KeyCode | Unity KeyCode |
|---------|----------|-----------------|---------------|
| Single/double finger tap | Confirm | 66 | 13 (Return) |
| Single/double finger double-tap | Back | 4 | 27 (Escape) |
| Single finger long press | Custom | 289 | - |
| Single finger swipe forward | Navigate right | 22 | 275 |
| Single finger swipe back | Navigate left | 21 | 276 |
| Single finger swipe up | Navigate up | 19 | 273 |
| Single finger swipe down | Navigate down | 20 | 274 |
| Double finger long press | Custom | 290 | - |
| Double finger swipe forward | Continuous right | 22 (repeated) | 275 (repeated) |
| Double finger swipe back | Continuous left | 21 (repeated) | 276 (repeated) |
| Double finger swipe up | Continuous up | 19 (repeated) | 273 (repeated) |
| Double finger swipe down | Continuous down | 20 (repeated) | 274 (repeated) |

### Advanced Gesture Handling (Android)

**Custom GestureDetector:**
```java
public class GestureBaseActivity extends AppCompatActivity {
    private static final float FLIP_DISTANCE = 130;
    private GestureDetector mDetector;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        startListenGesture();
    }

    protected void startListenGesture() {
        mDetector = new GestureDetector(this, new GestureDetector.SimpleOnGestureListener() {

            @Override
            public boolean onSingleTapConfirmed(MotionEvent e) {
                onSingleClick();
                return super.onSingleTapConfirmed(e);
            }

            @Override
            public void onLongPress(MotionEvent e) {
                onLongPressDetected();
            }

            @Override
            public boolean onDoubleTap(MotionEvent e) {
                onDoubleClick();
                return super.onDoubleTap(e);
            }

            @Override
            public boolean onScroll(MotionEvent e1, MotionEvent e2,
                                   float distanceX, float distanceY) {
                float deltaX = e1.getX() - e2.getX();
                float deltaY = e2.getY() - e1.getY();
                float ratio = Math.abs(deltaX) / Math.abs(deltaY);

                // Detect diagonal swipes
                if (deltaX > FLIP_DISTANCE && ratio > 0.5 && ratio < 2) {
                    if (deltaY > 0) {
                        onLeftDownSwipe();
                    } else {
                        onLeftUpSwipe();
                    }
                    return true;
                }

                if (deltaX < -FLIP_DISTANCE && ratio > 0.5 && ratio < 2) {
                    if (deltaY > 0) {
                        onRightDownSwipe();
                    } else {
                        onRightUpSwipe();
                    }
                    return true;
                }

                return super.onScroll(e1, e2, distanceX, distanceY);
            }
        });
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (mDetector != null) {
            mDetector.onTouchEvent(event);
        }
        return super.onTouchEvent(event);
    }
}
```

---

## AR Service Implementation

### Coordinate Systems

#### Raw AR Data Format
```
float[] getPose() returns 7 values:
[0] = X position (meters)
[1] = Y position (meters)
[2] = Z position (meters)
[3] = Quaternion W
[4] = Quaternion X
[5] = Quaternion Y
[6] = Quaternion Z
```

#### Unity Coordinate Conversion
Unity SDK automatically converts between coordinate systems:
```
Raw → Unity Position:
  Unity.X = Raw[0]
  Unity.Y = Raw[2]  ← Y and Z swapped
  Unity.Z = Raw[1]  ←

Raw → Unity Rotation:
  Unity.X = -Raw[3]  ← Inverted
  Unity.Y = -Raw[5]  ← Inverted & swapped
  Unity.Z = -Raw[4]  ← Inverted & swapped
  Unity.W = Raw[6]
```

### Performance Optimization

#### Android Native GL Rendering
```java
public class GlRenderer implements GLSurfaceView.Renderer {

    @Override
    public void onSurfaceChanged(GL10 gl, int width, int height) {
        GLES20.glViewport(0, 0, width, height);

        // Get projection matrix from AR service
        mProjectionMatrix = mArSession.getProjectionMatrix(
            width, height,
            0.1f,  // Near plane
            15.0f  // Far plane - keep small for performance
        );
    }

    @Override
    public void onDrawFrame(GL10 gl) {
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT | GLES20.GL_DEPTH_BUFFER_BIT);

        // Get fresh view matrix each frame
        mViewMatrix = mArSession.getViewMatrix();

        // Calculate MVP matrix
        Matrix.setIdentityM(mModelMatrix, 0);
        Matrix.translateM(mModelMatrix, 0, 0, 0, -2.0f);  // Move object
        Matrix.multiplyMM(mMVPMatrix, 0, mViewMatrix, 0, mModelMatrix, 0);
        Matrix.multiplyMM(mMVPMatrix, 0, mProjectionMatrix, 0, mMVPMatrix, 0);

        // Render geometry
        drawGeometry();
    }
}
```

#### Unity Rendering Loop
```csharp
public class ARCamera : MonoBehaviour
{
    private ARPoseService _arService;
    private ARPoseService.ARPose _currentPose;

    void Update()
    {
        // Get pose in Update for game logic
        if (_arService != null && _arService.isARPoseServiceRuning)
        {
            _currentPose = _arService.GetARPose();
        }
    }

    void LateUpdate()
    {
        // Apply to camera in LateUpdate for smooth rendering
        if (_currentPose != null)
        {
            Camera.main.transform.position = _currentPose.position;
            Camera.main.transform.rotation = _currentPose.rotation;
        }
    }
}
```

### Lifecycle Management Best Practices

#### Android Activity Lifecycle
```java
public class ARActivity extends Activity {
    private ArServiceSession mArSession;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Create session ONCE
        mArSession = new ArServiceSession(getApplication());
        if (!mArSession.create()) {
            Log.e(TAG, "Failed to create AR session");
            // Handle error - maybe show dialog to user
            showARUnavailableDialog();
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        // Resume tracking when app becomes foreground
        if (mArSession != null) {
            mArSession.resume();
        }
    }

    @Override
    protected void onPause() {
        super.onPause();
        // Pause to release camera for other apps
        if (mArSession != null) {
            mArSession.pause();
        }
    }

    @Override
    protected void onDestroy() {
        // Clean up when activity destroyed
        if (mArSession != null) {
            mArSession.destroy();
            mArSession = null;
        }
        super.onDestroy();
    }
}
```

#### Unity Scene Lifecycle
```csharp
public class ARManager : MonoBehaviour
{
    private ARPoseService arService;

    void OnEnable()
    {
        // Initialize when scene loads or object enabled
        if (arService == null)
        {
            arService = new ARPoseService(Camera.main, 13.4f);
        }
        arService.Start();
    }

    void OnDisable()
    {
        // Pause when scene unloads or object disabled
        if (arService != null)
        {
            arService.Pause();
        }
    }

    void OnDestroy()
    {
        // Clean up when object destroyed
        if (arService != null)
        {
            arService.Stop();
            arService = null;
        }
    }

    void OnApplicationPause(bool pauseStatus)
    {
        // Handle app pause/resume (home button, etc.)
        if (arService != null)
        {
            if (pauseStatus)
                arService.Pause();
            else
                arService.Resume();
        }
    }
}
```

---

## Best Practices & Optimization

### 3D Model Requirements (Unity)

#### Polygon Budget
- **Maximum on-screen polys:** 100,000 - 200,000 triangles
- **Per-model recommendation:** Keep models < 20,000 polys
- **Use LOD (Level of Detail)** for complex objects

#### Lighting Setup
```csharp
// Recommended lighting configuration
- ONE Directional Light only (dynamic)
- Use lightmaps for additional lights (baked)
- Disable shadows on main camera
- Set Directional Light Shadow Type to "No Shadows"
```

#### Camera Configuration
```csharp
Camera mainCamera = Camera.main;
mainCamera.fieldOfView = 13.4f;  // CRITICAL: Match AR optics
mainCamera.nearClipPlane = 0.1f;
mainCamera.farClipPlane = 15f;   // Keep small for performance
mainCamera.allowHDR = false;
mainCamera.allowMSAA = false;
```

#### Material & Shader Optimization
```
✅ DO:
- Use Unity Standard Shader (Mobile variants)
- Use texture atlases to reduce draw calls
- Combine meshes where possible
- Use GPU instancing for repeated objects

❌ DON'T:
- Avoid full-screen post-processing effects
- No screen-space reflections
- No complex outline/glow shaders (doubles poly count!)
- Minimize transparent materials
```

### Android Native Optimization

#### OpenGL Best Practices
```java
// Use VBOs (Vertex Buffer Objects)
private int[] vboHandles = new int[2];

void setupVBOs() {
    GLES20.glGenBuffers(2, vboHandles, 0);

    // Position buffer
    GLES20.glBindBuffer(GLES20.GL_ARRAY_BUFFER, vboHandles[0]);
    GLES20.glBufferData(GLES20.GL_ARRAY_BUFFER,
        vertexData.length * BYTES_PER_FLOAT,
        vertexBuffer, GLES20.GL_STATIC_DRAW);

    // Color buffer
    GLES20.glBindBuffer(GLES20.GL_ARRAY_BUFFER, vboHandles[1]);
    GLES20.glBufferData(GLES20.GL_ARRAY_BUFFER,
        colorData.length * BYTES_PER_FLOAT,
        colorBuffer, GLES20.GL_STATIC_DRAW);
}
```

#### Minimize State Changes
```java
// Bad: Many state changes
for (Mesh mesh : meshes) {
    GLES20.glUseProgram(mesh.shader);
    GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, mesh.texture);
    drawMesh(mesh);
}

// Good: Sort by material first
Map<Material, List<Mesh>> batches = sortByMaterial(meshes);
for (Material mat : batches.keySet()) {
    GLES20.glUseProgram(mat.shader);
    GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, mat.texture);
    for (Mesh mesh : batches.get(mat)) {
        drawMesh(mesh);
    }
}
```

### Memory Management

#### Texture Size Guidelines
```
Recommended texture sizes:
- UI elements: 512x512 or smaller
- 3D object textures: 1024x1024 maximum
- Use compressed formats: ETC2, ASTC
```

#### Unity Memory Optimization
```csharp
// Unload unused assets
Resources.UnloadUnusedAssets();

// Clean memory explicitly when scene changes
System.GC.Collect();

// Use object pooling for frequently created/destroyed objects
public class ObjectPool : MonoBehaviour
{
    public GameObject prefab;
    private Queue<GameObject> pool = new Queue<GameObject>();

    public GameObject Get()
    {
        if (pool.Count > 0)
        {
            GameObject obj = pool.Dequeue();
            obj.SetActive(true);
            return obj;
        }
        return Instantiate(prefab);
    }

    public void Return(GameObject obj)
    {
        obj.SetActive(false);
        pool.Enqueue(obj);
    }
}
```

### Power Consumption Tips

```
Battery-saving recommendations:
1. Reduce rendering frame rate if not needed for smooth motion
2. Minimize camera access when not needed (use pause())
3. Avoid continuous sensor polling - use event-driven approach
4. Implement idle timeout to pause AR tracking
5. Reduce screen brightness (if controllable)
```

### UI/UX Best Practices

#### Text Readability
```
Recommended font sizes (Unity):
- Minimum: 24pt
- Body text: 32pt
- Headers: 48pt+

Colors:
- High contrast white/black on solid backgrounds
- Avoid pure red/green (AR display limitations)
- Add drop shadows or outlines for readability
```

#### Interaction Design
```
1. Provide visual feedback for all inputs
   - Highlight selected items
   - Show button press animations
   - Use audio feedback

2. Design for one-handed operation
   - Ring2 should be primary input
   - Touchpad as secondary/backup

3. Keep UI in center of field-of-view
   - Don't place critical UI at edges
   - FOV is narrow (13.4°)

4. Use large, easy-to-hit targets
   - Minimum button size: 100x100 pixels
   - Space elements generously
```

---

## Publishing Requirements

### Package Naming Rules

#### Package Name Requirements
```
Format: com.company.appname

Rules:
- Only A-Z, a-z, 0-9, and underscores
- Separated by dots (periods)
- Minimum 2 segments
- Each segment must start with a letter

✅ Valid:
  com.mycompany.mygame
  com.studio_name.game_name

❌ Invalid:
  myapp                    (only 1 segment)
  com.123studio.game       (segment starts with number)
  com.my-company.app       (contains hyphen)
```

#### Package Name Best Practices
```
- Use unique package name to avoid conflicts
- NEVER change package name after publishing
  (treated as completely new app)
- Check existing apps in store before finalizing
- Use reverse domain notation
  (e.g., if domain is mycompany.com, use com.mycompany.*)
```

### Application Signing

#### Development Phase
```bash
# Use debug keystore for testing
# Location: ~/.android/debug.keystore (Linux/Mac)
#           %USERPROFILE%\.android\debug.keystore (Windows)

# Debug signing happens automatically in Android Studio
```

#### Release Signing

**Generate Release Keystore:**
```bash
keytool -genkey -v -keystore my-release-key.keystore \
  -alias my-key-alias \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000

# Keep this file SAFE! You cannot update your app without it!
```

**Sign APK:**
```bash
# Using jarsigner
jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 \
  -keystore my-release-key.keystore \
  my-app-release-unsigned.apk \
  my-key-alias

# Or use Android Studio: Build → Generate Signed Bundle/APK
```

**Optimize APK:**
```bash
# Use zipalign to optimize before signing
zipalign -v 4 my-app-unsigned.apk my-app-aligned.apk

# Then sign the aligned APK
apksigner sign --ks my-release-key.keystore \
  --out my-app-release.apk \
  my-app-aligned.apk
```

### Required AndroidManifest.xml Permissions

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.yourcompany.yourapp">

    <!-- CRITICAL: Camera permission for AR -->
    <uses-permission android:name="android.permission.CAMERA" />

    <!-- AR Service requires -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <!-- Optional: For sensor access -->
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />

    <!-- Declare camera features -->
    <uses-feature android:name="android.hardware.camera" android:required="true" />
    <uses-feature android:name="android.hardware.camera.autofocus" android:required="false" />

    <!-- GL ES version -->
    <uses-feature android:glEsVersion="0x00020000" android:required="true" />

    <application
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher"
        android:allowBackup="true"
        android:theme="@style/AppTheme">

        <activity
            android:name=".MainActivity"
            android:screenOrientation="landscape"
            android:configChanges="orientation|screenSize|keyboardHidden"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

### Build Configuration Requirements

**Android (build.gradle):**
```groovy
android {
    compileSdk 32

    defaultConfig {
        applicationId "com.yourcompany.yourapp"
        minSdk 28        // REQUIRED: Must be 28+
        targetSdk 32
        versionCode 1    // Increment for each update
        versionName "1.0.0"
    }

    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                         'proguard-rules.pro'
            // Sign with release key
            signingConfig signingConfigs.release
        }
    }

    signingConfigs {
        release {
            storeFile file("path/to/your/keystore.keystore")
            storePassword "your_store_password"
            keyAlias "your_key_alias"
            keyPassword "your_key_password"
        }
    }
}
```

**Unity Build Settings:**
```
Player Settings:
✓ Company Name: YourCompany
✓ Product Name: YourApp
✓ Package Name: com.yourcompany.yourapp
✓ Version: 1.0.0
✓ Bundle Version Code: 1

Other Settings:
✓ Minimum API Level: Android 9.0 'Pie' (API level 28)
✓ Target API Level: 32 or higher
✓ Scripting Backend: IL2CPP (recommended)
✓ Target Architectures: ARMv7 (uncheck ARM64)
✓ [ ] Optimize Frame Rate: UNCHECKED (CRITICAL!)

Graphics Settings:
✓ Auto Graphics API: OFF
✓ Graphics APIs: OpenGLES3, OpenGLES2 (in that order)

Publishing Settings:
✓ Build: Release
✓ Create symbols.zip: Enabled (for crash reports)
✓ Custom Keystore: Browse to your .keystore file
✓ Keystore password: ********
✓ Alias: your-key-alias
✓ Alias password: ********
```

### ADB Debugging Access

To test on INMO AIR2 device, you need ADB access:

**Enable Developer Mode on INMO AIR2:**
1. Go to Settings on the glasses
2. Navigate to "My Glasses" (我的眼镜)
3. **Long press the temple twice** to trigger ADB risk warning page
4. **Long press the temple again** to enter password input
5. Enter the correct password to enable ADB debugging

**Note:** This is a security feature specific to INMO AIR2. You'll need to contact INMO for the ADB access password.

**Connect via ADB:**
```bash
# Check device connected
adb devices

# Install your APK
adb install -r YourApp.apk

# View logs
adb logcat -s YourApp:D

# Uninstall
adb uninstall com.yourcompany.yourapp
```

**Wireless ADB (if supported):**
```bash
# On device via USB first
adb tcpip 5555

# Disconnect USB, then connect via IP
adb connect <device-ip>:5555

# Now can debug wirelessly
```

### Submission Checklist

```
Before submitting to INMO App Store:

Technical:
[ ] Minimum SDK 28 (Android 9.0)
[ ] App signed with release key (stored safely!)
[ ] Tested on actual INMO AIR2 hardware
[ ] AR tracking works smoothly
[ ] Ring2 and/or touchpad input implemented
[ ] No crashes during lifecycle events (pause/resume)
[ ] Permissions requested at runtime (Camera, etc.)
[ ] APK size optimized (< 100MB recommended)

Content:
[ ] App icon (all required densities)
[ ] App name and description prepared
[ ] Screenshots taken on INMO AIR2 device
[ ] Privacy policy URL (if collecting data)
[ ] Version number follows semantic versioning

Testing:
[ ] Tested in bright environments
[ ] Tested in low light
[ ] Battery consumption acceptable
[ ] No overheating issues
[ ] Handles loss of tracking gracefully
[ ] Works with Ring2 disconnected (touchpad fallback)
```

### Version Updates

```
When updating an existing app:

1. Increment versionCode (integer)
   Old: versionCode 1
   New: versionCode 2

2. Update versionName (string)
   Follow semantic versioning: MAJOR.MINOR.PATCH
   Example: 1.0.0 → 1.0.1 (bug fix)
            1.0.1 → 1.1.0 (new features)
            1.1.0 → 2.0.0 (breaking changes)

3. Use SAME package name
   CRITICAL: Never change this!

4. Sign with SAME keystore
   CRITICAL: Must use original release key!

5. Test upgrade path
   - Install old version
   - Install new version over it
   - Verify data preserved (if applicable)
```

---

## Quick Reference Tables

### Critical Version/API Requirements

| Component | Version/Level | Notes |
|-----------|---------------|-------|
| Android API Level | 28+ | Minimum required |
| INMO Glass Firmware | 2.4+ | For AR service |
| Unity | 2020.3 LTS+ | With Android Support |
| JDK | 11 | For Android development |
| NDK (Unity 2020.3) | r19 | Auto-installed via Unity Hub |
| NDK (Unity 2021.3) | r21d | Auto-installed via Unity Hub |

### SDK Dependencies

**Android Gradle:**
```groovy
// INMO SDK
implementation 'com.inmo:inmo_arsdk:0.0.1'

// Repository
maven { url 'https://gitee.com/inmolens/inmo-ar-sdk/raw/master' }
```

### Unity Package Structure

When importing Unity SDK, you'll see:
```
InmoUnitySdk/
├── SDK/                    ← Core AR services (AAR libraries)
├── Samples/                ← Example scenes and scripts
├── URP/                    ← Universal Render Pipeline config (optional)
└── Document.docx           ← Offline documentation
```

**Important Notes:**
- If using Built-in pipeline or already have URP config, delete the URP folder
- SDK folder contains Android AAR libraries for AR service
- Samples are optional but recommended for learning

### Camera Configuration Cheat Sheet

```
Unity:
  FOV: 13.4 (some docs say 13.9 - test which works best)
  Near: 0.1
  Far: 15
  Shadows: OFF
  HDR: OFF
  MSAA: OFF

Android OpenGL:
  Resolution: 640x400 (native optical resolution)
  Near: 0.2f (or 0.1f)
  Far: 15.0f to 20.0f
  getProjectionMatrix(640, 400, 0.2f, 20.0f)
```

### Performance Targets

```
Polygon Count:
  Total on-screen: 100k-200k tris
  Per model: <20k tris

Texture Sizes:
  UI: 512x512
  3D: 1024x1024 max
  Format: ETC2/ASTC

Lighting:
  1 Directional Light (dynamic)
  Rest: Lightmapped
  No shadows

Draw Calls:
  Target: <50 per frame
  Method: Batch by material
```

---

## Troubleshooting Common Issues

### AR Service Won't Start

**Problem:** `mArSession.create()` returns false

**Solutions:**
1. Check firmware version ≥ 2.4
2. Verify Camera permission granted
3. Ensure another app isn't using AR service
4. Try device reboot
5. Check logcat for specific error
6. **Wait for projection matrix initialization** - it may return zeros initially

**Code:**
```java
if (!mArSession.create()) {
    Log.e(TAG, "AR Service failed to start");
    String version = mArSession.getVersion();
    Log.e(TAG, "AR Service version: " + version);
    // Show user-friendly error
}

// Important: Projection matrix may be all zeros initially
// Loop until valid matrix is returned
mProjectionMatrix = mArSession.getProjectionMatrix(640, 400, 0.2f, 20);
while (mProjectionMatrix[0] == 0 && mProjectionMatrix[1] == 0 &&
       mProjectionMatrix[2] == 0 && mProjectionMatrix[3] == 0) {
    Log.d(TAG, "Waiting for valid projection matrix...");
    mProjectionMatrix = mArSession.getProjectionMatrix(640, 400, 0.2f, 20);
    try { Thread.sleep(100); } catch (InterruptedException e) {}
}
```

### Tracking Drift/Instability

**Problem:** Virtual objects move when head stays still

**Solutions:**
1. Ensure good lighting conditions
2. Avoid textureless walls/surfaces
3. Implement visual feedback when tracking lost
4. Reset tracking if drift accumulates

**Code:**
```csharp
// Check if pose has changed significantly
float poseDelta = Vector3.Distance(lastPose.position, currentPose.position);
if (poseDelta > 0.5f && Time.deltaTime < 0.1f)
{
    // Sudden jump = tracking issue
    ShowTrackingWarning();
}
```

### Ring2 Not Connecting

**Problem:** Ring events not received

**Solutions:**
1. Check Bluetooth enabled
2. Verify Ring2 paired in system settings
3. Check battery level of Ring2
4. Restart Bluetooth service

**Code:**
```java
private boolean isRingConnected() {
    BluetoothAdapter adapter = BluetoothAdapter.getDefaultAdapter();
    if (adapter != null && adapter.isEnabled()) {
        int state = adapter.getProfileConnectionState(4);  // HID
        return state == BluetoothProfile.STATE_CONNECTED;
    }
    return false;
}
```

### Unity Build Errors

**Problem:** Build fails or app crashes on launch

**Common Causes & Fixes:**

1. **"Minimum API Level"** error
   ```
   Fix: Player Settings → Minimum API Level → Android 9.0 (28)
   ```

2. **IL2CPP build fails**
   ```
   Fix: Ensure correct NDK installed via Unity Hub
   ```

3. **App crashes immediately**
   ```
   Fix: Check AndroidManifest.xml has CAMERA permission
   Fix: Uncheck "Optimize Frame Rate"
   Fix: Set Scripting Backend to IL2CPP
   ```

4. **AR Service not found**
   ```
   Fix: Verify running on actual INMO AIR2 device (not emulator)
   ```

### Performance Issues

**Problem:** Low frame rate, stuttering

**Diagnostics:**
```csharp
// Unity: Show FPS
void OnGUI()
{
    float fps = 1.0f / Time.deltaTime;
    GUI.Label(new Rect(10, 10, 100, 20), $"FPS: {fps:F1}");
}
```

**Solutions:**
1. Reduce polygon count
2. Optimize materials (use mobile shaders)
3. Disable shadows
4. Reduce draw calls (batch meshes)
5. Lower texture resolutions
6. Remove post-processing effects

### Input Not Working

**Problem:** Ring2 or touchpad events not detected

**Debug Code:**
```java
// Android: Log all key events
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    Log.d(TAG, "KeyCode: " + keyCode + " Event: " + event);
    return super.onKeyDown(keyCode, event);
}
```

```csharp
// Unity: Log all input
void Update()
{
    if (Input.anyKeyDown)
    {
        foreach (KeyCode key in System.Enum.GetValues(typeof(KeyCode)))
        {
            if (Input.GetKeyDown(key))
            {
                Debug.Log("Key pressed: " + key);
            }
        }
    }
}
```

---

## Sample Project Structures

### Android Native Project
```
MyARApp/
├── app/
│   ├── src/
│   │   └── main/
│   │       ├── java/com/company/myapp/
│   │       │   ├── MainActivity.java          ← AR session lifecycle
│   │       │   ├── GlRenderer.java           ← OpenGL rendering
│   │       │   ├── GLView.java               ← GL surface view
│   │       │   └── InputHandler.java         ← Ring/touchpad input
│   │       ├── res/
│   │       │   ├── layout/
│   │       │   │   └── activity_main.xml
│   │       │   └── values/
│   │       │       └── strings.xml
│   │       └── AndroidManifest.xml           ← Permissions!
│   └── build.gradle                          ← SDK dependency
├── gradle/
└── build.gradle                              ← Repository config
```

### Unity Project
```
MyARApp/
├── Assets/
│   ├── Scenes/
│   │   └── MainScene.unity
│   ├── Scripts/
│   │   ├── ARManager.cs                     ← AR service control
│   │   ├── InputManager.cs                  ← Ring/touchpad
│   │   └── UIController.cs                  ← User interface
│   ├── Prefabs/
│   │   └── ARContent.prefab
│   ├── Materials/
│   │   └── LightweightMaterial.mat          ← Mobile shaders
│   ├── Plugins/
│   │   └── Android/
│   │       └── AndroidManifest.xml          ← Custom permissions
│   └── InmoUnitySdk/                        ← SDK import
│       ├── SDK/
│       │   └── Scripts/
│       │       └── AR/
│       │           ├── ARPoseService.cs
│       │           └── RingPoseService.cs
│       └── Samples/
├── ProjectSettings/
│   └── ProjectSettings.asset                ← Build config
└── Packages/
    └── manifest.json
```

---

## Additional Resources

### Official Repositories
```
AR SDK (Android):
https://gitee.com/inmolens/inmo-ar-sdk-sample.git

UI SDK (Android):
https://gitee.com/inmolens/inmo-ui-sdk-sample.git

Unity SDK:
https://gitee.com/inmolens/inmo-sdk-sample-unity.git

Documentation:
https://gitee.com/inmolens/docs

SDK Maven Repository:
https://gitee.com/inmolens/inmo-ar-sdk/raw/master
```

### Unity Resources
```
Unity Official Site (China):
https://unity.cn/

Unity Documentation:
https://docs.unity3d.com/
```

### Community & Support
```
For issues with SDK:
- Check existing issues on Gitee
- Create new issue with:
  • Device firmware version
  • SDK version
  • Minimal reproduction steps
  • Logcat output (Android)
  • Unity version (if using Unity)

Android Developer Reference:
https://developer.android.google.cn/docs

INMO Contact:
- Check developer portal for official support channels
- ADB access requires password from INMO
```

### Important Notes
```
⚠️ ADB Access:
  - Requires special password from INMO
  - Cannot be enabled without official password
  - Contact INMO developer support for access

⚠️ Firmware Version:
  - Check "About" in settings to verify version ≥ 2.4
  - Older firmware may not support AR service

⚠️ Repository Access:
  - Primary repos hosted on Gitee (China)
  - May require VPN if accessing from outside China
  - GitHub mirrors may be available
```

---

## Appendix: Complete Code Examples

### A. Minimal Android AR App

**MainActivity.java:**
```java
package com.example.minimalapp;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;
import android.view.KeyEvent;
import com.arglasses.arsdk.ArServiceSession;

public class MainActivity extends Activity {
    private static final String TAG = "MinimalARApp";
    private ArServiceSession mArSession;
    private ARView mView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Initialize AR
        mArSession = new ArServiceSession(getApplication());
        if (!mArSession.create()) {
            Log.e(TAG, "Failed to create AR session");
            finish();
            return;
        }

        Log.i(TAG, "AR Version: " + mArSession.getVersion());

        // Setup view
        mView = new ARView(this, mArSession);
        setContentView(mView);
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_ENTER) {
            Log.i(TAG, "Confirm button pressed");
            // Handle input
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }

    @Override
    protected void onResume() {
        super.onResume();
        mView.onResume();
        mArSession.resume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        mView.onPause();
        mArSession.pause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mArSession.destroy();
    }
}
```

### B. Minimal Unity AR Scene

**ARController.cs:**
```csharp
using UnityEngine;
using UnityEngine.UI;
using inmo.unity.sdk;

public class ARController : MonoBehaviour
{
    private ARPoseService arService;
    private ARPoseService.ARPose currentPose;

    [SerializeField] private Text statusText;
    [SerializeField] private GameObject arContent;

    void Start()
    {
        // Initialize AR service
        arService = new ARPoseService(Camera.main, 13.4f);
        arService.Start();

        if (arService.isARPoseServiceRuning)
        {
            statusText.text = "AR Active";
            arContent.SetActive(true);
        }
        else
        {
            statusText.text = "AR Failed";
            arContent.SetActive(false);
        }
    }

    void Update()
    {
        // Get AR pose
        if (arService != null && arService.isARPoseServiceRuning)
        {
            currentPose = arService.GetARPose();
        }

        // Handle input
        if (Input.GetKeyDown(KeyCode.Return))
        {
            OnConfirmButton();
        }

        if (Input.GetKeyDown(KeyCode.Escape))
        {
            OnBackButton();
        }
    }

    void LateUpdate()
    {
        // Apply AR pose to camera
        if (currentPose != null)
        {
            transform.position = currentPose.position;
            transform.rotation = currentPose.rotation;
        }
    }

    void OnConfirmButton()
    {
        Debug.Log("Confirm pressed");
        // Your logic here
    }

    void OnBackButton()
    {
        Debug.Log("Back pressed");
        Application.Quit();
    }

    void OnDestroy()
    {
        if (arService != null)
        {
            arService.Stop();
        }
    }

    void OnApplicationPause(bool pause)
    {
        if (arService != null)
        {
            if (pause)
                arService.Pause();
            else
                arService.Resume();
        }
    }
}
```

---

## End of Guide

**Document Version:** 1.0
**Last Updated:** 2025
**Target SDK:** INMO AR SDK 0.0.1
**Compatible Hardware:** INMO AIR2 (Firmware 2.4+)

This guide consolidates information from:
- inmo-ar-sdk-sample (Android native examples)
- inmo-ui-sdk-sample (UI/input examples)
- inmo-sdk-sample-unity (Unity integration)
- Official INMO documentation (Chinese & English)

---

**Quick Start Checklist:**

For Android Native:
1. ✓ Add INMO SDK repository to build.gradle
2. ✓ Add inmo_arsdk dependency
3. ✓ Set minSdk 28
4. ✓ Request CAMERA permission
5. ✓ Create ArServiceSession
6. ✓ Handle lifecycle (onCreate/onPause/onResume/onDestroy)
7. ✓ Implement input handling (onKeyDown)

For Unity:
1. ✓ Install Unity 2020.3+ with Android Support
2. ✓ Import INMO Unity SDK package
3. ✓ Switch platform to Android
4. ✓ Set Minimum API Level 28
5. ✓ Configure camera FOV 13.4
6. ✓ Create ARPoseService in script
7. ✓ Handle Unity input (Input.GetKeyDown)
8. ✓ Uncheck "Optimize Frame Rate"

**You're now ready to develop for INMO AIR2! 🚀**
