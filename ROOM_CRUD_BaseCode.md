# ROOM - CRUD

CRUD 모두 구현한 ROOM 예제 소스

→ 로컬 DB에 데이터 저장하여 빠른 정보 조회 가능

## build.gradle

```kotlin
plugins {
   ...
}
apply plugin: 'kotlin-kapt'

...

dependencies {
	...

// Room components
    def roomVersion = '2.2.5'
    implementation "androidx.room:room-runtime:$roomVersion"
    kapt "androidx.room:room-compiler:$roomVersion"
    implementation "androidx.room:room-ktx:$roomVersion"
    androidTestImplementation "androidx.room:room-testing:$roomVersion"
}
```

[https://developer.android.com/studio/build/migrate-to-ksp?hl=ko](https://developer.android.com/studio/build/migrate-to-ksp?hl=ko)

최신 플러그인 추가 kapt → ksp ( 23.03 )

***MainData*** - Entity

***MainDao*** - DAO

***RoomDB*** - Database

***MainAdapter*** - RecyclerView Adapter

## MainData

```kotlin
package co.kr.filaerp.room3
import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey
import java.io.Serializable

@Entity(tableName = "table_name")
class MainData : Serializable {
    @PrimaryKey(autoGenerate = true)
    var id = 0

    @ColumnInfo(name = "text")
    var text: String? = null
}
```

## MainDao

```kotlin
package co.kr.filaerp.room3

import androidx.room.*

@Dao
interface MainDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun insert(mainData: MainData?)

    @Delete
    fun delete(mainData: MainData?)

    @Delete
    fun reset(mainData: List<MainData?>?)

    @Query("UPDATE table_name SET text = :sText WHERE ID = :sID")
    fun update(sID: Int, sText: String?)

    @get:Query("SELECT * FROM table_name")
    val all: List<MainData?>?
}
```

## RoomDB

```kotlin
package co.kr.filaerp.room3

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(entities = [MainData::class], version = 1, exportSchema = false)
abstract class RoomDB : RoomDatabase() {
    abstract fun mainDao(): MainDao?

    companion object {
        private var database: RoomDB? = null
        private const val DATABASE_NAME = "database"
        @Synchronized
        fun getInstance(context: Context): RoomDB? {
            if (database == null) {
                database = Room.databaseBuilder(
                    context.applicationContext,
                    RoomDB::class.java, DATABASE_NAME
                )
                    .allowMainThreadQueries()
                    .fallbackToDestructiveMigration()
                    .build()
            }
            return database
        }
    }
}
```

## MainAdapter

```kotlin
package co.kr.filaerp.room3

import android.app.Activity
import android.app.Dialog
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.view.WindowManager
import android.widget.Button
import android.widget.EditText
import android.widget.ImageView
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class MainAdapter(private val context: Activity, private val dataList: MutableList<MainData?>) :
    RecyclerView.Adapter<MainAdapter.ViewHolder>() {
    private var database: RoomDB? = null
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view: View =
            LayoutInflater.from(parent.context).inflate(R.layout.list_row_main, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val data = dataList[position]
        database = RoomDB.getInstance(context)
        holder.textView.text= data!!.text
        holder.btEdit.setOnClickListener{
val mainData = dataList[holder.adapterPosition]
            val sID = mainData!!.id
            val sText = mainData.text
            val dialog = Dialog(context)
            dialog.setContentView(R.layout.dialog_update)
            val width = WindowManager.LayoutParams.MATCH_PARENT
val height = WindowManager.LayoutParams.WRAP_CONTENT
dialog.window!!.setLayout(width, height)
            dialog.show()
            val editText = dialog.findViewById<EditText>(R.id.dialog_edit_text)
            val bt_update = dialog.findViewById<Button>(R.id.bt_update)
            editText.setText(sText)
	bt_update.setOnClickListener{
			dialog.dismiss()
      val uText = editText.text.toString().trim{ it<= ' '}
			database!!.mainDao()!!.update(sID, uText)
      dataList.clear()
      dataList.addAll(database!!.mainDao()!!.all!!)
      
			notifyDataSetChanged()
			}
}

/* 삭제 클릭 */
holder.btDelete.setOnClickListener{
		val mainData = dataList[holder.adapterPosition]
		database!!.mainDao()!!.delete(mainData)
		val position = holder.adapterPosition
		dataList.removeAt(position)
		notifyItemRemoved(position)
		notifyItemRangeChanged(position, dataList.size)
	}
}

    override fun getItemCount(): Int {
        return dataList.size
    }

    inner class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        var textView: TextView
        var btEdit: ImageView
        var btDelete: ImageView

        init {
            textView = view.findViewById(R.id.text_view)
            btEdit = view.findViewById(R.id.bt_edit)
            btDelete = view.findViewById(R.id.bt_delete)
        }
    }

    init {
        notifyDataSetChanged()
    }
}
```

- 각 리스트의 삭제 버튼 클릭시 Dao에서 쿼리 Delete 실행
    - MainData (data class) dataList에서 삭제
- 각 holder클릭 시 update dialog 띄움
    - dialog에 수정 버튼 클릭 시 텍스트가 공백이 아니라면, update 쿼리 Dao에서 실행

## MainActivity

```kotlin
package co.kr.filaerp.room3

import android.os.Bundle
import android.view.View
import android.widget.Button
import android.widget.EditText
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {

	override fun onCreate(savedInstanceState: Bundle?) {
	super.onCreate(savedInstanceState)
	setContentView(R.layout.activity_main)
	var editText = findViewById<EditText>(R.id.edit_text)
	var btAdd = findViewById<Button>(R.id.bt_add)
	var btReset = findViewById<Button>(R.id.bt_reset)
	var recyclerView = findViewById<RecyclerView>(R.id.recycler_view)
	var database = RoomDB.getInstance(this)
	var dataList = database!!.mainDao()!!.all
	
	recyclerView.setLayoutManager(LinearLayoutManager(this))

	var adapter = MainAdapter(this@MainActivity, 
														dataList as MutableList<MainData?>)

  recyclerView.setAdapter(adapter)

  btAdd.setOnClickListener(View.OnClickListener{

	val sText = editText.getText().toString().trim{ it<= ' '}
	if (sText != "") {
    val data = MainData()
    data.text = sText
    database!!.mainDao()!!.insert(data)
    editText.setText("")
    dataList.clear()
    database!!.mainDao()!!.all?.let{it1->dataList.addAll(it1)}

		adapter!!.notifyDataSetChanged()
  }
})

  btReset.setOnClickListener(View.OnClickListener{
	database!!.mainDao()!!.reset(dataList)
            dataList.clear()
            database!!.mainDao()!!.all?.let{it1->dataList.addAll(it1)}
	adapter!!.notifyDataSetChanged()
})
  }
}
```

- reset 버튼 클릭시 dataList 모두 Clear, Dao reset(delete 모든 리스트) 쿼리 실행
- add버튼 클릭시  dataList에 추가 Dao insert 쿼리 실행