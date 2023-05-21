# Retrofit2 Base Study

[https://velog.io/@dldmswo1209/MVVM-패턴-공부-Retrofit-ViewModel](https://velog.io/@dldmswo1209/MVVM-%ED%8C%A8%ED%84%B4-%EA%B3%B5%EB%B6%80-Retrofit-ViewModel)

레트로핏 mvvm 예시

[https://jsonplaceholder.typicode.com/users](https://jsonplaceholder.typicode.com/users)

- 해당 JSON 파일 API 주소로 호출하여 데이터 출력 (데이터 트리 구조)

```json
{
	"id": 1,
	"name": "Leanne Graham",
	"username": "Bret",
	"email": "[Sincere@april.biz](mailto:Sincere@april.biz)",
	"address": {
		"street": "Kulas Light",
		"suite": "Apt. 556",
		"city": "Gwenborough",
		"zipcode": "92998-3874",
		"geo": {
			"lat": "-37.3159",
			"lng": "81.1496"
		}
	},
	"phone": "1-770-736-8031 x56442",
	"website": "[hildegard.org](http://hildegard.org/)",
	"company": {
		"name": "Romaguera-Crona",
		"catchPhrase": "Multi-layered client-server neural-net",
		"bs": "harness real-time e-markets"
	}
},
```

- 바인딩 사용 위해서 앱 단위 gradle 추가

```kotlin
dataBinding {
        enabled = true
    }
```

- mainfest.xml retrofit 통신 위해서 인터넷 권한 추가

```cpp
<uses-permission android:name="android.permission.INTERNET"/>
```

이러한 데이터가 있다고 가정하면, 

Model 먼저 생성, 

```kotlin
data class User(
    val id: Int,
    val name: String,
    val username: String,
    val email: String,
    val address: Address,
    val phone: String,
    val website: String,
    val company: Company
)

data class Address(
    val street: String,
    val suite: String,
    val city: String,
    val zipcode: String,
    val geo: Geo
)

data class Geo(
    val lat: String,
    val lng: String
)

data class Company(
    val name: String,
    val catchPhrase: String,
    val bs: String
)
```

Retrofit 사용 Build.gradle 의존성 추가

```kotlin
//retrofit
implementation 'com.squareup.retrofit2:retrofit:2.9.0' // retrofit
implementation 'com.squareup.retrofit2:converter-gson:2.9.0' // GsonConverter

// viewModelScope
implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.0'

// coroutine
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9")
```

Retrofit 객체 가져오기 위한 Instance 생성

```kotlin
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitInstance{
    val BASE_URL = "https://jsonplaceholder.typicode.com/"

    val clinent = Retrofit
        .Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    fun getInstance(): Retrofit {
        return clinent
    }
}
```

GET 요청을 하기 위한 인터페이스 구현

JSON 데이터가 리스트 형태이기 때문에 return 값은 User 객체를 요소로 가지는 List

```kotlin
import co.kr.filaerp.retrofitbase.data.model.User
import retrofit2.http.GET

interface MyApi {
    @GET("/users")
    suspend fun getAllUser(): List<User>
}
```

Repository 에서 Retrofit 객체 생성, 서버에 데이터 요청 작업

```kotlin
class Repository {
  private val client = RetrofitInstance.getInstance().create(MyApi::class.java)

  suspend fun getAllData() = client.getAllUser()
}
```

ViewModel에서는 Repository에게 데이터를 가져오라고 명령,

***LiveData***에 업데이트 해줍니다.

```kotlin
import android.util.Log
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import co.kr.filaerp.retrofitbase.Repository
import co.kr.filaerp.retrofitbase.data.model.User
import kotlinx.coroutines.launch

class MainViewModel : ViewModel() {

    private val repository = Repository()

    private val _result = MutableLiveData<List<User>>()
    val result: LiveData<List<User>>
        get() = _result

    fun getAllData() =viewModelScope.launch{
			Log.d("MainViewModel", repository.getAllData().toString())
	    _result.value= repository.getAllData()
		}
}
```

## activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout>
<androidx.constraintlayout.widget.ConstraintLayout 
	xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:app="http://schemas.android.com/apk/res-auto"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  tools:context=".MainActivity">

	<androidx.recyclerview.widget.RecyclerView
	    android:id="@+id/rcv"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"/>
</androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

## test_row_item.xml (데이터 반복 출력할 RecyclerView item)

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout>
<androidx.appcompat.widget.LinearLayoutCompat xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:orientation="vertical">

    <TextView
        android:id="@+id/textId"
        android:text="textId"
        android:textSize="40sp"
        android:layout_gravity="center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    <TextView
        android:id="@+id/textName"
        android:text="textName"
        android:textSize="40sp"
        android:layout_gravity="center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    <TextView
        android:id="@+id/companyName"
        android:text="companyName"
        android:textSize="40sp"
        android:layout_gravity="center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    <TextView
        android:id="@+id/address_geo_lat"
        android:text="address_geo_lat"
        android:textSize="40sp"
        android:layout_gravity="center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</androidx.appcompat.widget.LinearLayoutCompat>
</layout>
```

## Adapter

```kotlin
class CustomAdapter(val context: Context, val dataSet: List<User>) : RecyclerView.Adapter<CustomAdapter.ViewHolder>() {
  inner class ViewHolder(val binding: TestRowItemBinding): RecyclerView.ViewHolder(binding.root){
        fun bind(user: User){
            binding.textId.text= user.id.toString()
            binding.textName.text= user.name
            binding.companyName.text= user.company.name
            binding.addressGeoLat.text= user.address.geo.lat
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        return ViewHolder(TestRowItemBinding.inflate(LayoutInflater.from(parent.context),parent,false))
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(dataSet[position])
    }

    override fun getItemCount(): Int {
        return dataSet.size
    }
}
```

## MainActivity

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModelProvider
import co.kr.filaerp.retrofitbase.Adapter.CustomAdapter
import co.kr.filaerp.retrofitbase.databinding.ActivityMainBinding
import co.kr.filaerp.retrofitbase.viewModel.MainViewModel

class MainActivity : AppCompatActivity() {

  private val binding by lazy{
		ActivityMainBinding.inflate(layoutInflater)
	}

			override fun onCreate(savedInstanceState: Bundle?) {
			    super.onCreate(savedInstanceState)
			    setContentView(binding.root)
			
			    val viewModel = ViewModelProvider(this)[MainViewModel::class.java]
			    viewModel.getAllData()
			
			    viewModel.result.observe(this,Observer{
			val customAdapter = CustomAdapter(this,it)
			        binding.rcv.adapter= customAdapter
			})
	 }
}
```

![Untitled](Retrofit2%20Base%20Study%208cc7bacb3f6e458cb598243558482125/Untitled.png)
