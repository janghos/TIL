# Loading Progress 구현

Loading Layout Dialog 

**ProgressDialog.class - Dialog 상속 받음**

```kotlin
class ProgressDialog(context: Context?) : Dialog(context!!) {
    init {
        // 다이얼 로그 제목을 안보이게
        requestWindowFeature(Window.FEATURE_NO_TITLE)
        setContentView(R.layout.dialog_progress)
    }
}
```

**dialog_progress.xml (Progress Dialog 레이아웃)**

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="6dp"
    android:background="@drawable/progress_bg" >
    <ProgressBar
        android:layout_width="72dp"
        android:layout_height="72dp"
        android:indeterminateDrawable="@drawable/progress_image" />
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:text="Loading..."
        android:textStyle="bold"
        android:textColor="@android:color/white"
        android:textSize="16dp" />
</LinearLayout>
```

**Vetor Asset - 톱니바퀴 이미지 생성 (setting 이미지 참조)**

```kotlin
<vector android:height="24dp" android:tint="#FFFFFF"
    android:viewportHeight="24" android:viewportWidth="24"
    android:width="24dp" xmlns:android="http://schemas.android.com/apk/res/android">
    <path android:fillColor="@android:color/white" android:pathData="M19.14,12.94c0.04,-0.3 0.06,-0.61 0.06,-0.94c0,-0.32 -0.02,-0.64 -0.07,-0.94l2.03,-1.58c0.18,-0.14 0.23,-0.41 0.12,-0.61l-1.92,-3.32c-0.12,-0.22 -0.37,-0.29 -0.59,-0.22l-2.39,0.96c-0.5,-0.38 -1.03,-0.7 -1.62,-0.94L14.4,2.81c-0.04,-0.24 -0.24,-0.41 -0.48,-0.41h-3.84c-0.24,0 -0.43,0.17 -0.47,0.41L9.25,5.35C8.66,5.59 8.12,5.92 7.63,6.29L5.24,5.33c-0.22,-0.08 -0.47,0 -0.59,0.22L2.74,8.87C2.62,9.08 2.66,9.34 2.86,9.48l2.03,1.58C4.84,11.36 4.8,11.69 4.8,12s0.02,0.64 0.07,0.94l-2.03,1.58c-0.18,0.14 -0.23,0.41 -0.12,0.61l1.92,3.32c0.12,0.22 0.37,0.29 0.59,0.22l2.39,-0.96c0.5,0.38 1.03,0.7 1.62,0.94l0.36,2.54c0.05,0.24 0.24,0.41 0.48,0.41h3.84c0.24,0 0.44,-0.17 0.47,-0.41l0.36,-2.54c0.59,-0.24 1.13,-0.56 1.62,-0.94l2.39,0.96c0.22,0.08 0.47,0 0.59,-0.22l1.92,-3.32c0.12,-0.22 0.07,-0.47 -0.12,-0.61L19.14,12.94zM12,15.6c-1.98,0 -3.6,-1.62 -3.6,-3.6s1.62,-3.6 3.6,-3.6s3.6,1.62 3.6,3.6S13.98,15.6 12,15.6z"/>
</vector>
```

**progress_image 이미지 회전 효과 (dialog_progress)**

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<animated-rotate
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/ic_baseline_settings_24"
    android:pivotX="50%"
    android:pivotY="50%">
</animated-rotate>
```

**progress_bg 테두리 둥근 배경 (dialog_progress)** 

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <solid android:color="#22ffffff"/>
    <corners android:radius="50dp"/>
    <stroke android:color="#22ffffff" android:width="1dp"/>
</shape>
```

**MainActivity** 

```kotlin
class MainActivity : AppCompatActivity() {
  var customProgressDialog: ProgressDialog? = null
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    val btnLoad = findViewById<Button>(R.id.btnload)
    //로딩창 객체 생성
    customProgressDialog = ProgressDialog(this)
    //로딩창을 투명하게
    customProgressDialog!!.window!!.setBackgroundDrawable(
			ColorDrawable(Color.TRANSPARENT)
		)
		//외부 클릭 막기
    customProgressDialog!!.setCancelable(false)
		//로딩바 생성
    btnLoad.setOnClickListener{
				customProgressDialog!!.show()
		}
	}
}
```