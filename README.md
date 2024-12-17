# -Vehicle-Management-App-Android-Studios-
Android Application
# -Vehicle-Management-App-Android-Studios-
Android Application

Creating a basic Vehicle Management app using Kotlin for Android involves several components like user interface, database management, and handling user inputs. Below is a simple guide and code structure to help you build a basic vehicle management app.
//1. Project Setup	
•	Create a new project: Open Android Studio and create a new project with an “Empty Activity” template.	
•	Select Kotlin as the programming language.


//2. Add Dependencies
Ensure that you have the necessary dependencies for RecyclerView and Room (SQLite database) in your build.gradle file.
dependencies {    
    implementation "androidx.recyclerview:recyclerview:1.2.1"    
    implementation "androidx.room:room-runtime:2.5.0"    
    kapt "androidx.room:room-compiler:2.5.0"    
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.6.1"    
    implementation "androidx.room:room-ktx:2.5.0"}

//3. Define the Data Model
We will create a simple data model for the vehicles. This will contain fields like vehicle ID, name, and registration number.
@Entity(tableName = "vehicles")

data class Vehicle(

    @PrimaryKey(autoGenerate = true) val id: Int = 0,

    @ColumnInfo(name = "name") val name: String,

    @ColumnInfo(name = "registration_number") val registrationNumber: String

)
//4. Create the Room Database
Create a VehicleDao interface to define the database operations.



@Dao

interface VehicleDao {

    @Insert

    suspend fun insert(vehicle: Vehicle)



    @Delete

    suspend fun delete(vehicle: Vehicle)



    @Query("SELECT * FROM vehicles")

    fun getAllVehicles(): LiveData<List<Vehicle>>

}



Next, create the database class.



@Database(entities = [Vehicle::class], version = 1, exportSchema = false)

abstract class VehicleDatabase : RoomDatabase() {

    abstract fun vehicleDao(): VehicleDao

}
//5. Repository for Data Handling
We will create a repository to abstract the data source and provide a clean API for accessing the data.
class VehicleRepository(private val vehicleDao: VehicleDao) {



    val allVehicles: LiveData<List<Vehicle>> = vehicleDao.getAllVehicles()



    suspend fun addVehicle(vehicle: Vehicle) {

        vehicleDao.insert(vehicle)

    }



    suspend fun removeVehicle(vehicle: Vehicle) {

        vehicleDao.delete(vehicle)

    }

}


//6. ViewModel for UI Communication
Now, create a VehicleViewModel to manage UI-related data in a lifecycle-conscious way.
class VehicleViewModel(application: Application) : AndroidViewModel(application) {



    private val repository: VehicleRepository

    val allVehicles: LiveData<List<Vehicle>>



    init {

        val vehicleDao = VehicleDatabase.getDatabase(application).vehicleDao()

        repository = VehicleRepository(vehicleDao)

        allVehicles = repository.allVehicles

    }



    fun addVehicle(vehicle: Vehicle) {

        viewModelScope.launch(Dispatchers.IO) {

            repository.addVehicle(vehicle)

        }

    }



    fun removeVehicle(vehicle: Vehicle) {

        viewModelScope.launch(Dispatchers.IO) {

            repository.removeVehicle(vehicle)

        }

    }

}


//7. Create the User Interface
Now, define a simple interface to interact with the user. We’ll create a simple list of vehicles and a button to add vehicles.
activity_main.xml (for displaying vehicles and adding a new one):
<?xml version="1.0" encoding="utf-8"?><androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:app="http://schemas.android.com/apk/res-auto"

    xmlns:tools="http://schemas.android.com/tools"

    android:layout_width="match_parent"

    android:layout_height="match_parent"

    tools:context=".MainActivity">

<androidx.recyclerview.widget.RecyclerView        
android:id="@+id/vehicleRecyclerView"        
android:layout_width="0dp"        
android:layout_height="0dp"        
app:layout_constraintBottom_toBottomOf="parent"        
app:layout_constraintEnd_toEndOf="parent"        
app:layout_constraintStart_toStartOf="parent"        
app:layout_constraintTop_toTopOf="parent"/>
<Button

        android:id="@+id/addButton"

        android:layout_width="wrap_content"

        android:layout_height="wrap_content"

        android:text="Add Vehicle"

        app:layout_constraintBottom_toBottomOf="parent"

        app:layout_constraintEnd_toEndOf="parent"

        app:layout_constraintStart_toStartOf="parent"

        app:layout_constraintTop_toTopOf="parent"/>

</androidx.constraintlayout.widget.ConstraintLayout>
vehicle_item.xml (for each vehicle entry in the RecyclerView):
<?xml version="1.0" encoding="utf-8"?><androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"

    android:layout_height="wrap_content"

    android:layout_margin="8dp">

    <LinearLayout

        android:orientation="vertical"

        android:padding="16dp"

        android:layout_width="match_parent"

        android:layout_height="wrap_content">

        <TextView

            android:id="@+id/vehicleName"

            android:layout_width="match_parent"

            android:layout_height="wrap_content"

            android:textSize="18sp"

            android:text="Vehicle Name"/>

             <TextView

            android:id="@+id/registrationNumber"

            android:layout_width="match_parent"

            android:layout_height="wrap_content"

            android:text="Registration Number"/>

            </LinearLayout></androidx.cardview.widget.CardView>



//8. RecyclerView Adapter
Now, create a RecyclerView adapter to bind data to the list of vehicles.
class VehicleAdapter(private val vehicleList: List<Vehicle>) : RecyclerView.Adapter<VehicleAdapter.VehicleViewHolder>() {



    class VehicleViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

        val vehicleName: TextView = itemView.findViewById(R.id.vehicleName)

        val registrationNumber: TextView = itemView.findViewById(R.id.registrationNumber)

    }



    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): VehicleViewHolder {

        val view = LayoutInflater.from(parent.context).inflate(R.layout.vehicle_item, parent, false)

        return VehicleViewHolder(view)

    }



    override fun onBindViewHolder(holder: VehicleViewHolder, position: Int) {

        val vehicle = vehicleList[position]

        holder.vehicleName.text = vehicle.name

        holder.registrationNumber.text = vehicle.registrationNumber

    }



    override fun getItemCount(): Int = vehicleList.size

}


//9. MainActivity
Finally, connect everything in the MainActivity.
class MainActivity : AppCompatActivity() {



    private lateinit var vehicleViewModel: VehicleViewModel

    private lateinit var vehicleAdapter: VehicleAdapter



    override fun onCreate(savedInstanceState: Bundle?) {

        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)



        // Set up RecyclerView

        val recyclerView: RecyclerView = findViewById(R.id.vehicleRecyclerView)

        recyclerView.layoutManager = LinearLayoutManager(this)



        vehicleViewModel = ViewModelProvider(this).get(VehicleViewModel::class.java)



        vehicleViewModel.allVehicles.observe(this, Observer { vehicles ->

            vehicleAdapter = VehicleAdapter(vehicles)

            recyclerView.adapter = vehicleAdapter

        })



        val addButton: Button = findViewById(R.id.addButton)

        addButton.setOnClickListener {

            // Example: Adding a new vehicle

            val newVehicle = Vehicle(name = "Car", registrationNumber = "XYZ123")

            vehicleViewModel.addVehicle(newVehicle)

        }

    }

}


//10. Room Database Initialization
Ensure that you initialize the Room database properly when accessing it.
object VehicleDatabase {

    @Volatile

    private var INSTANCE: VehicleDatabase? = null



    fun getDatabase(context: Context): VehicleDatabase {

        return INSTANCE ?: synchronized(this) {

            val instance = Room.databaseBuilder(

                context.applicationContext,

                VehicleDatabase::class.java,

                "vehicle_database"

            ).build()

            INSTANCE = instance

            instance

        }

    }

}


//11. Running the App	•	Running the app will display a list of vehicles. You can add vehicles by clicking the “Add Vehicle” button.	•	You can modify the addVehicle function to accept inputs (like name, registration number) from a form or another interface.
This is a simple foundation for a vehicle management app. You can extend it with additional features like vehicle detail views, update functionality, or filtering based on various criteria.
