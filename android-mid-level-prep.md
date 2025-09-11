# Android Developer Interview Guide (3+ Years Experience)

## Overview for Mid-Level Android Developers

As a developer with 3+ years of experience, you're expected to demonstrate:
- **Solid fundamentals** with Android core concepts
- **Practical experience** with modern Android development tools
- **Architectural understanding** and design pattern implementation
- **Problem-solving skills** for complex development challenges
- **Code quality awareness** including testing and optimization
- **Team collaboration** and project management understanding

---

## Core Android Concepts (Expected Mastery)

### Application Architecture & Design Patterns

**Q: Explain MVVM architecture and its benefits over other patterns.**
**A:** MVVM (Model-View-ViewModel) separates UI logic from business logic:

**Components:**
- **Model**: Data layer (repositories, network, database)
- **View**: UI layer (Activities, Fragments, Compose)
- **ViewModel**: Holds UI data, survives configuration changes

**Benefits over MVP:**
- **Lifecycle awareness**: Automatic handling of Activity/Fragment lifecycle
- **Data binding**: Two-way binding with LiveData/StateFlow
- **Testability**: ViewModel can be tested without Android framework
- **Less boilerplate**: Automatic UI updates through observers

**Implementation Example:**
```kotlin
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    private val _loading = MutableLiveData<Boolean>()
    val loading: LiveData<Boolean> = _loading
    
    fun loadUsers() {
        viewModelScope.launch {
            _loading.value = true
            try {
                _users.value = repository.getUsers()
            } catch (e: Exception) {
                // Handle error
            } finally {
                _loading.value = false
            }
        }
    }
}
```

**Q: What is Dependency Injection and why is it important?**
**A:** DI is providing dependencies to a class rather than the class creating them.

**Benefits:**
- **Testability**: Easy to mock dependencies
- **Flexibility**: Change implementations without modifying client code
- **Single Responsibility**: Classes focus on their core logic
- **Loose coupling**: Reduces dependencies between components

**Hilt Implementation:**
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideApiService(): ApiService {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
}

@HiltAndroidApp
class MyApplication : Application()

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var apiService: ApiService
}
```

### Modern Android Development

**Q: Compare Coroutines with RxJava for Android development.**
**A:** Both handle asynchronous operations but with different approaches:

| Aspect | Coroutines | RxJava |
|--------|------------|---------|
| **Learning Curve** | Easier, sequential syntax | Steeper, functional style |
| **Performance** | Lightweight | More overhead |
| **Error Handling** | try-catch blocks | onError operators |
| **Threading** | Dispatchers | Schedulers |
| **Cancellation** | Structured concurrency | Manual disposal |
| **Backpressure** | Flow with buffer | Built-in operators |

**When to use each:**
- **Coroutines**: New projects, simple async operations, Kotlin-first teams
- **RxJava**: Complex stream operations, existing RxJava codebase, reactive programming needs

**Q: Explain StateFlow vs LiveData in detail.**
**A:** Both are observable data holders but with different characteristics:

**StateFlow:**
```kotlin
class ViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    fun updateState() {
        _uiState.value = _uiState.value.copy(loading = true)
    }
}
```

**LiveData:**
```kotlin
class ViewModel : ViewModel() {
    private val _uiState = MutableLiveData<UiState>()
    val uiState: LiveData<UiState> = _uiState
    
    fun updateState() {
        _uiState.value = UiState(loading = true)
    }
}
```

**Comparison:**
| Feature | StateFlow | LiveData |
|---------|-----------|----------|
| **Platform** | Kotlin multiplatform | Android-specific |
| **Lifecycle awareness** | Manual | Automatic |
| **Initial value** | Required | Optional |
| **Threading** | Any thread | Main thread posting |
| **Operators** | Flow operators | Transformations class |
| **Compose integration** | Native | Bridge required |

---

## Architecture Components (Intermediate Level)

### Repository Pattern

**Q: Implement a Repository pattern with caching strategy.**
**A:** Repository abstracts data sources and implements caching:

```kotlin
@Singleton
class UserRepository @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao,
    private val networkBoundResource: NetworkBoundResource
) {
    
    fun getUsers(): Flow<Resource<List<User>>> = networkBoundResource.fetch(
        query = { userDao.getAllUsers() },
        fetch = { apiService.getUsers() },
        saveFetchResult = { users -> userDao.insertAll(users) },
        shouldFetch = { users -> 
            users.isEmpty() || isCacheExpired()
        }
    )
    
    suspend fun refreshUsers() {
        try {
            val users = apiService.getUsers()
            userDao.clearAndInsert(users)
        } catch (e: Exception) {
            // Handle network error
        }
    }
    
    private fun isCacheExpired(): Boolean {
        // Implement cache expiration logic
        return System.currentTimeMillis() - lastCacheTime > CACHE_TIMEOUT
    }
}

// Usage in ViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    
    val users = repository.getUsers()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = Resource.Loading()
        )
    
    fun refreshUsers() {
        viewModelScope.launch {
            repository.refreshUsers()
        }
    }
}
```

### Advanced Room Usage

**Q: How do you handle complex relationships in Room?**
**A:** Room supports various relationship types:

**One-to-Many Relationship:**
```kotlin
@Entity
data class User(
    @PrimaryKey val id: Int,
    val name: String
)

@Entity
data class Post(
    @PrimaryKey val id: Int,
    val userId: Int,
    val title: String,
    val content: String
)

data class UserWithPosts(
    @Embedded val user: User,
    @Relation(
        parentColumn = "id",
        entityColumn = "userId"
    )
    val posts: List<Post>
)

@Dao
interface UserDao {
    @Transaction
    @Query("SELECT * FROM user WHERE id = :userId")
    suspend fun getUserWithPosts(userId: Int): UserWithPosts
}
```

**Many-to-Many Relationship:**
```kotlin
@Entity(tableName = "student_course_cross_ref")
data class StudentCourseCrossRef(
    val studentId: Int,
    val courseId: Int
)

data class StudentWithCourses(
    @Embedded val student: Student,
    @Relation(
        parentColumn = "studentId",
        entityColumn = "courseId",
        associateBy = Junction(StudentCourseCrossRef::class)
    )
    val courses: List<Course>
)
```

---

## Performance & Optimization (Critical for 3+ Years)

### Memory Management

**Q: How do you identify and fix memory leaks in Android?**
**A:** Multi-step approach:

**1. Detection Tools:**
- **LeakCanary**: Automatic leak detection
- **Memory Profiler**: Heap analysis
- **MAT (Memory Analyzer Tool)**: Heap dump analysis

**2. Common Leak Patterns:**
```kotlin
// BAD: Static reference to Activity
class LeakyClass {
    companion object {
        private lateinit var activity: Activity // Memory leak!
    }
}

// GOOD: Use WeakReference or Application Context
class SafeClass {
    companion object {
        private var activityRef: WeakReference<Activity>? = null
    }
}

// BAD: Anonymous inner class in Activity
class MainActivity : AppCompatActivity() {
    private val handler = object : Handler(Looper.getMainLooper()) {
        override fun handleMessage(msg: Message) {
            // This holds reference to Activity
        }
    }
}

// GOOD: Static inner class with WeakReference
class MainActivity : AppCompatActivity() {
    private val handler = SafeHandler(this)
    
    private class SafeHandler(activity: MainActivity) : Handler(Looper.getMainLooper()) {
        private val activityRef = WeakReference(activity)
        
        override fun handleMessage(msg: Message) {
            activityRef.get()?.let { activity ->
                // Safe to use activity
            }
        }
    }
}
```

**3. ViewModel Scoping:**
```kotlin
// BAD: Holding Activity reference
class BadViewModel(private val activity: Activity) : ViewModel()

// GOOD: Use Application Context or Repository
class GoodViewModel(
    private val context: Application,
    private val repository: Repository
) : ViewModel()
```

### Performance Optimization

**Q: How do you optimize RecyclerView performance?**
**A:** Multiple optimization techniques:

**1. ViewHolder Pattern with ViewBinding:**
```kotlin
class UserAdapter : RecyclerView.Adapter<UserAdapter.UserViewHolder>() {
    
    class UserViewHolder(
        private val binding: ItemUserBinding
    ) : RecyclerView.ViewHolder(binding.root) {
        
        fun bind(user: User) {
            binding.user = user
            binding.executePendingBindings() // Important for data binding
        }
    }
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): UserViewHolder {
        val binding = ItemUserBinding.inflate(
            LayoutInflater.from(parent.context), parent, false
        )
        return UserViewHolder(binding)
    }
    
    override fun onBindViewHolder(holder: UserViewHolder, position: Int) {
        holder.bind(getItem(position))
    }
}
```

**2. DiffUtil for Efficient Updates:**
```kotlin
class UserDiffCallback : DiffUtil.ItemCallback<User>() {
    override fun areItemsTheSame(oldItem: User, newItem: User): Boolean {
        return oldItem.id == newItem.id
    }
    
    override fun areContentsTheSame(oldItem: User, newItem: User): Boolean {
        return oldItem == newItem
    }
}

class UserAdapter : ListAdapter<User, UserViewHolder>(UserDiffCallback()) {
    // Adapter implementation
}
```

**3. RecyclerView Optimizations:**
```kotlin
recyclerView.apply {
    setHasFixedSize(true) // If item size is fixed
    setItemViewCacheSize(20) // Increase cache size
    setRecycledViewPool(RecyclerView.RecycledViewPool()) // Share pool between RecyclerViews
    
    // For nested RecyclerViews
    (layoutManager as? LinearLayoutManager)?.apply {
        isItemPrefetchEnabled = true
        initialPrefetchItemCount = 4
    }
}
```

**Q: Explain Android's rendering pipeline and common performance issues.**
**A:** Understanding the 16ms frame budget:

**Rendering Pipeline:**
1. **Input**: Touch events processed
2. **Animation**: Animate properties
3. **Measure**: Calculate view dimensions
4. **Layout**: Position views
5. **Draw**: Render to surface
6. **Sync**: Submit to GPU

**Common Issues:**
- **Overdraw**: Multiple layers painted over each other
- **Layout thrashing**: Frequent measure/layout passes
- **GPU overload**: Complex shaders or too many layers
- **Main thread blocking**: Heavy operations on UI thread

**Optimization Techniques:**
```kotlin
// Reduce overdraw
<View
    android:background="@null"  // Remove unnecessary backgrounds
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />

// Use ConstraintLayout to flatten hierarchy
<androidx.constraintlayout.widget.ConstraintLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    
    <!-- Flat layout structure -->
    
</androidx.constraintlayout.widget.ConstraintLayout>

// Lazy initialization
class ExpensiveView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null
) : View(context, attrs) {
    
    private val expensiveObject by lazy {
        // Create only when needed
        createExpensiveObject()
    }
}
```

---

## Testing (Expected for Mid-Level)

### Unit Testing Strategy

**Q: How do you structure tests for MVVM architecture?**
**A:** Layer-by-layer testing approach:

**1. ViewModel Testing:**
```kotlin
@ExtendWith(MockitoExtension::class)
class UserViewModelTest {
    
    @Mock
    private lateinit var repository: UserRepository
    
    @Mock
    private lateinit var savedStateHandle: SavedStateHandle
    
    private lateinit var viewModel: UserViewModel
    
    @Before
    fun setup() {
        viewModel = UserViewModel(repository, savedStateHandle)
    }
    
    @Test
    fun `loadUsers should update users LiveData`() = runTest {
        // Given
        val expectedUsers = listOf(
            User(1, "John", "john@example.com"),
            User(2, "Jane", "jane@example.com")
        )
        `when`(repository.getUsers()).thenReturn(flowOf(Resource.Success(expectedUsers)))
        
        // When
        viewModel.loadUsers()
        
        // Then
        val result = viewModel.users.getOrAwaitValue()
        assertThat(result.data).isEqualTo(expectedUsers)
    }
    
    @Test
    fun `loadUsers should show loading state`() = runTest {
        // Given
        `when`(repository.getUsers()).thenReturn(flowOf(Resource.Loading()))
        
        // When
        viewModel.loadUsers()
        
        // Then
        val result = viewModel.loading.getOrAwaitValue()
        assertThat(result).isTrue()
    }
}

// Extension function for testing LiveData
fun <T> LiveData<T>.getOrAwaitValue(): T {
    var data: T? = null
    val latch = CountDownLatch(1)
    
    val observer = object : Observer<T> {
        override fun onChanged(value: T) {
            data = value
            latch.countDown()
            this@getOrAwaitValue.removeObserver(this)
        }
    }
    
    this.observeForever(observer)
    latch.await(2, TimeUnit.SECONDS)
    
    @Suppress("UNCHECKED_CAST")
    return data as T
}
```

**2. Repository Testing:**
```kotlin
@ExtendWith(MockitoExtension::class)
class UserRepositoryTest {
    
    @Mock private lateinit var apiService: ApiService
    @Mock private lateinit var userDao: UserDao
    
    private lateinit var repository: UserRepository
    
    @Test
    fun `getUsers should return cached data when available`() = runTest {
        // Given
        val cachedUsers = listOf(User(1, "John", "john@example.com"))
        `when`(userDao.getAllUsers()).thenReturn(flowOf(cachedUsers))
        
        repository = UserRepository(apiService, userDao)
        
        // When
        val result = repository.getUsers().first()
        
        // Then
        assertThat(result).isInstanceOf(Resource.Success::class.java)
        assertThat((result as Resource.Success).data).isEqualTo(cachedUsers)
    }
}
```

### Integration Testing

**Q: How do you test Room database operations?**
**A:** Use in-memory database for tests:

```kotlin
@RunWith(AndroidJUnit4::class)
class UserDaoTest {
    
    private lateinit var database: AppDatabase
    private lateinit var userDao: UserDao
    
    @Before
    fun createDb() {
        val context = ApplicationProvider.getApplicationContext<Context>()
        database = Room.inMemoryDatabaseBuilder(context, AppDatabase::class.java)
            .allowMainThreadQueries() // Only for testing
            .build()
        userDao = database.userDao()
    }
    
    @After
    fun closeDb() {
        database.close()
    }
    
    @Test
    fun insertAndGetUser() = runTest {
        // Given
        val user = User(1, "John", "john@example.com")
        
        // When
        userDao.insert(user)
        val retrieved = userDao.getUserById(1)
        
        // Then
        assertThat(retrieved).isEqualTo(user)
    }
    
    @Test
    fun updateUser() = runTest {
        // Given
        val user = User(1, "John", "john@example.com")
        userDao.insert(user)
        
        // When
        val updatedUser = user.copy(name = "John Updated")
        userDao.update(updatedUser)
        
        // Then
        val retrieved = userDao.getUserById(1)
        assertThat(retrieved.name).isEqualTo("John Updated")
    }
}
```

---

## Advanced Topics for 3+ Years Experience

### Custom Views & Animations

**Q: How do you create a performant custom view?**
**A:** Focus on measurement, drawing, and touch handling:

```kotlin
class ProgressCircleView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {
    
    private val paint = Paint().apply {
        isAntiAlias = true
        style = Paint.Style.STROKE
        strokeWidth = 20f
        strokeCap = Paint.Cap.ROUND
    }
    
    private val backgroundPaint = Paint().apply {
        isAntiAlias = true
        style = Paint.Style.STROKE
        strokeWidth = 20f
        color = Color.LTGRAY
    }
    
    private var progress = 0f
        set(value) {
            field = value.coerceIn(0f, 100f)
            invalidate() // Trigger redraw
        }
    
    private val rect = RectF()
    
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        val size = minOf(
            MeasureSpec.getSize(widthMeasureSpec),
            MeasureSpec.getSize(heightMeasureSpec)
        )
        setMeasuredDimension(size, size)
    }
    
    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        val padding = paint.strokeWidth / 2
        rect.set(padding, padding, w - padding, h - padding)
    }
    
    override fun onDraw(canvas: Canvas?) {
        super.onDraw(canvas)
        canvas?.let {
            // Draw background circle
            it.drawCircle(width / 2f, height / 2f, rect.width() / 2f, backgroundPaint)
            
            // Draw progress arc
            val angle = (progress / 100f) * 360f
            it.drawArc(rect, -90f, angle, false, paint)
        }
    }
    
    fun setProgressAnimated(targetProgress: Float) {
        ValueAnimator.ofFloat(progress, targetProgress).apply {
            duration = 1000
            interpolator = AccelerateDecelerateInterpolator()
            addUpdateListener { animation ->
                progress = animation.animatedValue as Float
            }
            start()
        }
    }
}
```

### Modularization & Build Optimization

**Q: How do you structure a multi-module Android project?**
**A:** Feature-based modularization:

```
app/                          # Main app module
├── feature/
│   ├── user/                 # User feature module
│   ├── dashboard/            # Dashboard feature module
│   └── settings/             # Settings feature module
├── core/
│   ├── common/               # Common utilities
│   ├── network/              # Networking module
│   ├── database/             # Database module
│   └── ui/                   # UI components
├── data/
│   ├── user/                 # User data module
│   └── analytics/            # Analytics data module
└── domain/
    ├── user/                 # User domain module
    └── analytics/            # Analytics domain module
```

**Benefits:**
- **Build performance**: Parallel compilation
- **Code isolation**: Clear boundaries between features
- **Team scaling**: Different teams can work on different modules
- **Reusability**: Modules can be shared across apps

**Gradle Configuration:**
```kotlin
// feature/user/build.gradle.kts
dependencies {
    implementation(project(":core:common"))
    implementation(project(":core:ui"))
    implementation(project(":domain:user"))
    
    // Don't depend on other features directly
    // Use navigation or shared interfaces
}
```

### Security Implementation

**Q: How do you implement secure storage in Android?**
**A:** Multi-layered security approach:

```kotlin
class SecureStorage @Inject constructor(
    @ApplicationContext private val context: Context
) {
    
    private val masterKey = MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build()
    
    private val encryptedPrefs = EncryptedSharedPreferences.create(
        context,
        "secure_prefs",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
    
    fun saveToken(token: String) {
        encryptedPrefs.edit()
            .putString(KEY_TOKEN, token)
            .apply()
    }
    
    fun getToken(): String? {
        return encryptedPrefs.getString(KEY_TOKEN, null)
    }
    
    // For very sensitive data, use Android Keystore
    private fun generateSecretKey(): SecretKey {
        val keyGenerator = KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore")
        val keyGenParameterSpec = KeyGenParameterSpec.Builder(
            "MySecretKey",
            KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
        )
            .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
            .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
            .setUserAuthenticationRequired(true)
            .setUserAuthenticationValidityDurationSeconds(300)
            .build()
        
        keyGenerator.init(keyGenParameterSpec)
        return keyGenerator.generateKey()
    }
}
```

---

## System Design Questions (3+ Years Level)

### Chat Application Architecture

**Q: Design a chat application architecture.**
**A:** Complete system design:

**High-Level Architecture:**
```
Client (Android) ←→ Load Balancer ←→ Chat Service ←→ Database
                                      ↓
                              Message Queue (WebSocket)
```

**Android Implementation:**
```kotlin
// WebSocket Manager
@Singleton
class ChatWebSocketManager @Inject constructor() {
    private var webSocket: WebSocket? = null
    private val messageFlow = MutableSharedFlow<ChatMessage>()
    
    fun connect(userId: String) {
        val request = Request.Builder()
            .url("wss://chat.example.com/ws?userId=$userId")
            .build()
        
        webSocket = OkHttpClient().newWebSocket(request, object : WebSocketListener() {
            override fun onMessage(webSocket: WebSocket, text: String) {
                val message = Json.decodeFromString<ChatMessage>(text)
                messageFlow.tryEmit(message)
            }
            
            override fun onFailure(webSocket: WebSocket, t: Throwable, response: Response?) {
                // Handle reconnection
                scheduleReconnect()
            }
        })
    }
    
    fun sendMessage(message: ChatMessage) {
        val json = Json.encodeToString(message)
        webSocket?.send(json)
    }
    
    fun observeMessages(): Flow<ChatMessage> = messageFlow.asSharedFlow()
}

// Repository Implementation
@Singleton
class ChatRepository @Inject constructor(
    private val webSocketManager: ChatWebSocketManager,
    private val chatDao: ChatDao,
    private val apiService: ChatApiService
) {
    
    fun observeMessages(chatId: String): Flow<List<ChatMessage>> = combine(
        chatDao.getMessages(chatId),
        webSocketManager.observeMessages()
    ) { cached, newMessage ->
        if (newMessage.chatId == chatId) {
            cached + newMessage
        } else {
            cached
        }
    }
    
    suspend fun sendMessage(message: ChatMessage) {
        // Optimistic UI update
        chatDao.insertMessage(message.copy(status = MessageStatus.SENDING))
        
        try {
            webSocketManager.sendMessage(message)
            chatDao.updateMessageStatus(message.id, MessageStatus.SENT)
        } catch (e: Exception) {
            chatDao.updateMessageStatus(message.id, MessageStatus.FAILED)
        }
    }
}
```

**Data Layer:**
```kotlin
@Entity
data class ChatMessage(
    @PrimaryKey val id: String,
    val chatId: String,
    val senderId: String,
    val content: String,
    val timestamp: Long,
    val status: MessageStatus = MessageStatus.SENT
)

enum class MessageStatus {
    SENDING, SENT, DELIVERED, READ, FAILED
}

@Dao
interface ChatDao {
    @Query("SELECT * FROM chatmessage WHERE chatId = :chatId ORDER BY timestamp ASC")
    fun getMessages(chatId: String): Flow<List<ChatMessage>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertMessage(message: ChatMessage)
    
    @Query("UPDATE chatmessage SET status = :status WHERE id = :messageId")
    suspend fun updateMessageStatus(messageId: String, status: MessageStatus)
}
```

---

## Interview Preparation Strategy

### Technical Deep Dives (What Interviewers Look For)

**1. Code Review Skills:**
- Identify potential bugs and improvements
- Discuss performance implications
- Suggest better architectural approaches
- Security considerations

**2. Problem-Solving Approach:**
- Break down complex problems
- Consider multiple solutions
- Discuss trade-offs
- Implement scalable solutions

**3. System Design Thinking:**
- Understand requirements clearly
- Design scalable architecture
- Consider offline capabilities
- Handle edge cases and errors

### Common Interview Formats

**1. Technical Discussion (30-45 minutes):**
- Architecture patterns and when to use them
- Android framework deep dive
- Performance optimization scenarios
- Testing strategies

**2. Live Coding (45-60 minutes):**
- Implement a feature end-to-end
- Debug existing code
- Optimize performance bottlenecks
- Write unit tests

**3. System Design (45-60 minutes):**
- Design a complete Android application
- Discussion of offline-first architecture
- Scalability and performance considerations
- Security and data privacy

### Sample Questions by Experience Level

**Architecture & Design (Must Know):**
- "Explain the Repository pattern and its benefits"
- "How do you handle configuration changes?"
- "Describe your approach to dependency injection"
- "What's your strategy for handling different screen sizes?"

**Performance & Optimization (Expected):**
- "How do you debug ANR issues?"
- "Explain your approach to memory leak detection"
- "How do you optimize network calls?"
- "Describe bitmap optimization strategies"

**Advanced Topics (Bonus):**
- "How do you implement offline-first architecture?"
- "Explain your approach to modularization"
- "How do you handle app security?"
- "Describe your testing pyramid"

---

## Final Interview Tips

### Before the Interview
1. **Review recent projects**: Be ready to discuss architecture decisions
2. **Practice coding**: Implement common Android patterns from scratch
3. **Study system design**: Understand how to build scalable Android apps
4. **Prepare questions**: Show interest in the company's technical challenges

### During the Interview
1. **Think out loud**: Explain your reasoning process
2. **Ask clarifying questions**: Understand requirements fully
3. **Start simple**: Begin with basic solution, then optimize
4. **Consider edge cases**: Think about error handling and edge cases
5. **Discuss trade-offs**: Explain why you chose one approach over another

### Red Flags to Avoid
- Not understanding basic Android concepts
- Unable to explain architecture decisions
- No experience with modern Android development tools
- Lack of testing knowledge
- Poor problem-solving approach

Remember: As a developer with 3+ years of experience, you're expected to not just implement features, but to make informed architectural decisions, mentor junior developers, and contribute to the overall technical direction of projects. Focus on demonstrating your depth of knowledge, problem-solving skills, and ability to build maintainable, scalable Android applications.

Good luck with your interview! 🚀