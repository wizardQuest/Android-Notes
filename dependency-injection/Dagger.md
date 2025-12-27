# Dagger 2.0 and Dependency Injection - Interview Preparation Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Dagger Basics](#dagger-basics)
3. [Using Dagger in Android Apps](#using-dagger-in-android-apps)
4. [Using Dagger in Multi-Module Apps](#using-dagger-in-multi-module-apps)
5. [Scoping in Dagger](#scoping-in-dagger)
6. [Hilt - Simplified Dagger for Android](#hilt---simplified-dagger-for-android)
7. [Koin - Lightweight DI for Kotlin](#koin---lightweight-di-for-kotlin)
8. [Framework Comparison](#framework-comparison)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Introduction

**Dependency Injection (DI)** is a design pattern in which an object receives other objects that it depends on. These other objects are called dependencies.

### Why Dependency Injection?

- **Testability**: Easy to replace dependencies with mocks or test doubles
- **Code Reusability**: Dependencies can be reused across different classes
- **Maintainability**: Centralized management of object creation and dependencies
- **Loose Coupling**: Classes don't need to know how to create their dependencies

### Dagger Overview

Dagger is a fully static, compile-time dependency injection framework for Java and Android. It uses annotation processing to generate code that connects objects and their dependencies.

**Key Benefits:**
- ✅ Compile-time safety (errors caught at build time)
- ✅ Performance (no reflection at runtime)
- ✅ Type-safe dependency resolution
- ✅ Generated code is traceable and debuggable

---

## Dagger Basics

Reference: [Dagger Basics - Android Developers](https://developer.android.com/training/dependency-injection/dagger-basics)

### Core Concepts

#### 1. **@Inject Annotation**

The `@Inject` annotation tells Dagger how to create instances of a class. It can be used on:
- **Constructor**: Preferred method, tells Dagger to use this constructor to create instances
- **Fields**: For field injection (after object is created)
- **Methods**: For method injection (after object is created)

**Kotlin Example:**
```kotlin
class UserRepository @Inject constructor(
    private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource
) {
    // Dagger will automatically provide UserLocalDataSource and UserRemoteDataSource
}
```

**Java Example:**
```java
public class UserRepository {
    private final UserLocalDataSource localDataSource;
    private final UserRemoteDataSource remoteDataSource;

    @Inject
    public UserRepository(UserLocalDataSource localDataSource, 
                         UserRemoteDataSource remoteDataSource) {
        this.localDataSource = localDataSource;
        this.remoteDataSource = remoteDataSource;
    }
}
```

#### 2. **@Module and @Provides**

When you can't use constructor injection (e.g., interfaces, third-party libraries), use `@Module` with `@Provides` methods.

**Kotlin Example:**
```kotlin
@Module
class NetworkModule {
    @Provides
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    @Provides
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}
```

**Java Example:**
```java
@Module
public class NetworkModule {
    @Provides
    public Retrofit provideRetrofit() {
        return new Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build();
    }

    @Provides
    public ApiService provideApiService(Retrofit retrofit) {
        return retrofit.create(ApiService.class);
    }
}
```

#### 3. **@Component Interface**

A `@Component` interface is the bridge between modules and the classes that need dependencies. It defines:
- Which modules to use
- Where dependencies should be injected
- What types can be provided

**Kotlin Example:**
```kotlin
@Component(modules = [NetworkModule::class, DatabaseModule::class])
interface ApplicationGraph {
    // Methods to get instances
    fun repository(): UserRepository
    
    // Methods to inject into classes
    fun inject(activity: MainActivity)
}
```

**Java Example:**
```java
@Component(modules = {NetworkModule.class, DatabaseModule.class})
public interface ApplicationGraph {
    UserRepository userRepository();
    void inject(MainActivity activity);
}
```

#### 4. **Using the Generated Component**

Dagger generates an implementation of your component with the prefix `Dagger`:

**Kotlin Example:**
```kotlin
// Create component instance
val applicationGraph: ApplicationGraph = DaggerApplicationGraph.create()

// Get dependency
val userRepository: UserRepository = applicationGraph.repository()

// Inject into Activity
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var userRepository: UserRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        DaggerApplicationGraph.create().inject(this)
        // Now userRepository is initialized
    }
}
```

**Java Example:**
```java
// Create component instance
ApplicationGraph applicationGraph = DaggerApplicationGraph.create();

// Get dependency
UserRepository userRepository = applicationGraph.userRepository();

// Inject into Activity
public class MainActivity extends AppCompatActivity {
    @Inject
    UserRepository userRepository;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        DaggerApplicationGraph.create().inject(this);
        // Now userRepository is initialized
    }
}
```

### Dependency Graph Example

```kotlin
// Data sources
class UserLocalDataSource @Inject constructor() { }
class UserRemoteDataSource @Inject constructor(
    private val apiService: ApiService
) { }

// Repository
class UserRepository @Inject constructor(
    private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource
) { }

// Component
@Component(modules = [NetworkModule::class])
interface ApplicationGraph {
    fun repository(): UserRepository
}
```

Dagger automatically resolves the dependency chain:
- `UserRepository` needs `UserLocalDataSource` and `UserRemoteDataSource`
- `UserRemoteDataSource` needs `ApiService`
- `ApiService` is provided by `NetworkModule`

---

## Using Dagger in Android Apps

Reference: [Using Dagger in Android Apps - Android Developers](https://developer.android.com/training/dependency-injection/dagger-android)

### Challenges with Android Framework Classes

Android framework classes (Activities, Fragments, Services) are instantiated by the system, not by Dagger. This makes constructor injection impossible.

### Solution: dagger-android

The `dagger-android` library provides Android-specific support to simplify dependency injection into Android components.

### Setup

**1. Add Dependencies**

```gradle
// build.gradle (app level)
dependencies {
    implementation 'com.google.dagger:dagger:2.x'
    kapt 'com.google.dagger:dagger-compiler:2.x'
    
    implementation 'com.google.dagger:dagger-android:2.x'
    implementation 'com.google.dagger:dagger-android-support:2.x'
    kapt 'com.google.dagger:dagger-android-processor:2.x'
}
```

**2. Create Application Component**

```kotlin
@Singleton
@Component(
    modules = [
        AndroidInjectionModule::class,
        AppModule::class,
        ActivityBindingModule::class
    ]
)
interface AppComponent {
    @Component.Factory
    interface Factory {
        fun create(@BindsInstance application: Application): AppComponent
    }
    
    fun inject(app: MyApplication)
}
```

**3. Create Application Class**

```kotlin
class MyApplication : Application(), HasAndroidInjector {
    @Inject
    lateinit var androidInjector: DispatchingAndroidInjector<Any>

    override fun onCreate() {
        super.onCreate()
        DaggerAppComponent.factory()
            .create(this)
            .inject(this)
    }

    override fun androidInjector(): AndroidInjector<Any> = androidInjector
}
```

**4. Create Activity Binding Module**

```kotlin
@Module
abstract class ActivityBindingModule {
    @ContributesAndroidInjector
    abstract fun contributeMainActivity(): MainActivity
}
```

**5. Inject into Activity**

```kotlin
class MainActivity : DaggerAppCompatActivity() {
    @Inject
    lateinit var userRepository: UserRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Dependencies are automatically injected before super.onCreate()
        // userRepository is ready to use
    }
}
```

### Alternative: Manual Activity Injection

If you don't want to extend `DaggerAppCompatActivity`, you can inject manually:

```kotlin
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var userRepository: UserRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        AndroidInjection.inject(this)
        super.onCreate(savedInstanceState)
        // userRepository is ready to use
    }
}
```

### Injecting into Fragments

**1. Add Fragment Binding**

```kotlin
@Module
abstract class FragmentBindingModule {
    @ContributesAndroidInjector
    abstract fun contributeHomeFragment(): HomeFragment
}
```

**2. Inject in Fragment**

```kotlin
class HomeFragment : DaggerFragment() {
    @Inject
    lateinit var userRepository: UserRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // userRepository is ready to use
    }
}
```

### Injecting into ViewModels

**1. Create ViewModel Factory**

```kotlin
@Module
abstract class ViewModelModule {
    @Binds
    abstract fun bindViewModelFactory(
        factory: DaggerViewModelFactory
    ): ViewModelProvider.Factory
}
```

**2. ViewModel Factory Implementation**

```kotlin
class DaggerViewModelFactory @Inject constructor(
    private val creators: Map<Class<out ViewModel>, @JvmSuppressWildcards Provider<ViewModel>>
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        val creator = creators[modelClass] 
            ?: creators.entries.firstOrNull { modelClass.isAssignableFrom(it.key) }?.value
            ?: throw IllegalArgumentException("Unknown ViewModel class: $modelClass")
        
        return try {
            creator.get() as T
        } catch (e: Exception) {
            throw RuntimeException(e)
        }
    }
}
```

**3. Create ViewModel**

```kotlin
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    // ViewModel code
}
```

---

## Using Dagger in Multi-Module Apps

Reference: [Using Dagger in Multi-Module Apps - Android Developers](https://developer.android.com/training/dependency-injection/dagger-multi-module)

### Module Structure

In a multi-module project, you typically have:
- **:app** module (main application module)
- **:core** module (shared dependencies)
- **:feature-*** modules (feature-specific modules)

### Component Dependencies vs Subcomponents

#### Option 1: Component Dependencies

Components can depend on other components to access their provided dependencies.

**App Component (Core Module):**
```kotlin
@Singleton
@Component(modules = [CoreModule::class])
interface AppComponent {
    fun repository(): UserRepository
    fun apiService(): ApiService
}
```

**Feature Component (Feature Module):**
```kotlin
@Component(
    dependencies = [AppComponent::class],
    modules = [FeatureModule::class]
)
interface FeatureComponent {
    fun inject(activity: FeatureActivity)
    
    @Component.Factory
    interface Factory {
        fun create(appComponent: AppComponent): FeatureComponent
    }
}
```

**Usage:**
```kotlin
val featureComponent = DaggerFeatureComponent.factory()
    .create(appComponent)
featureComponent.inject(this)
```

#### Option 2: Subcomponents (Recommended)

Subcomponents inherit all dependencies from their parent component.

**App Component:**
```kotlin
@Singleton
@Component(modules = [CoreModule::class])
interface AppComponent {
    fun featureComponent(): FeatureComponent.Factory
}
```

**Feature Subcomponent:**
```kotlin
@FeatureScope
@Subcomponent(modules = [FeatureModule::class])
interface FeatureComponent {
    fun inject(activity: FeatureActivity)
    
    @Subcomponent.Factory
    interface Factory {
        fun create(): FeatureComponent
    }
}
```

**Usage:**
```kotlin
val featureComponent = appComponent.featureComponent().create()
featureComponent.inject(this)
```

### Best Practices for Multi-Module Setup

1. **Expose Only What's Needed**
   ```kotlin
   // Good: Expose only public API
   @Component
   interface AppComponent {
       fun apiService(): ApiService  // Expose what features need
   }
   
   // Bad: Don't expose internal dependencies
   // fun database(): Database  // Keep internal
   ```

2. **Use @Binds for Interfaces**
   ```kotlin
   @Module
   abstract class CoreModule {
       @Binds
       abstract fun bindRepository(
           impl: UserRepositoryImpl
       ): UserRepository  // Interface
   }
   ```

3. **Feature-Specific Modules**
   ```kotlin
   // Feature module only knows about its own dependencies
   @Module
   class FeatureModule {
       @Provides
       @FeatureScope
       fun provideFeatureData(): FeatureData {
           return FeatureData()
       }
   }
   ```

### Example: Multi-Module Project Structure

```
:app
  └── AppComponent (depends on all feature components)
:core
  └── AppComponent (provides core dependencies)
:feature-login
  └── LoginComponent (subcomponent of AppComponent)
:feature-home
  └── HomeComponent (subcomponent of AppComponent)
```

---

## Scoping in Dagger

### What is Scoping?

Scoping limits the lifetime of an object to the lifetime of its component. The same instance is provided every time the type is requested within that scope.

### Default Behavior (No Scope)

Without scoping, Dagger creates a new instance every time:

```kotlin
@Component
interface ApplicationGraph {
    fun repository(): UserRepository
}

val graph = DaggerApplicationGraph.create()
val repo1 = graph.repository()
val repo2 = graph.repository()
// repo1 != repo2 (different instances)
```

### @Singleton Scope

Makes an instance unique within the component:

```kotlin
@Singleton
@Component(modules = [AppModule::class])
interface ApplicationGraph {
    fun repository(): UserRepository
}

@Singleton
class UserRepository @Inject constructor() { }

val graph = DaggerApplicationGraph.create()
val repo1 = graph.repository()
val repo2 = graph.repository()
// repo1 == repo2 (same instance)
```

### Custom Scopes

You can create custom scopes for specific lifetimes:

**Kotlin:**
```kotlin
@Scope
@MustBeDocumented
@Retention(AnnotationRetention.RUNTIME)
annotation class ActivityScope

@Scope
@MustBeDocumented
@Retention(AnnotationRetention.RUNTIME)
annotation class FragmentScope
```

**Java:**
```java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface ActivityScope { }
```

**Usage:**
```kotlin
@ActivityScope
@Component
interface ActivityComponent {
    fun userSession(): UserSession
}

@ActivityScope
class UserSession @Inject constructor() { }
```

### Scope Rules

1. **Component and dependency must have the same scope** (or no scope)
2. **Subcomponents cannot have a scope longer than parent**
3. **Scope annotations are checked at compile time**

---

## Hilt - Simplified Dagger for Android

Hilt is a dependency injection library built on top of Dagger that simplifies DI setup for Android applications.

Reference: [Hilt Documentation - Android Developers](https://developer.android.com/training/dependency-injection/hilt-android)

### Key Advantages Over Dagger

- ✅ **Less boilerplate**: Automatic component generation
- ✅ **Standard components**: Predefined components for Android classes
- ✅ **Integrated with Jetpack**: Works seamlessly with ViewModel, WorkManager, etc.
- ✅ **Easier setup**: Less configuration needed

### Setup

**1. Add Dependencies**

```gradle
// build.gradle (project level)
buildscript {
    dependencies {
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.x'
    }
}

// build.gradle (app level)
plugins {
    id 'kotlin-kapt'
    id 'dagger.hilt.android.plugin'
}

dependencies {
    implementation 'com.google.dagger:hilt-android:2.x'
    kapt 'com.google.dagger:hilt-android-compiler:2.x'
}
```

**2. Annotate Application Class**

```kotlin
@HiltAndroidApp
class MyApplication : Application() {
    // No manual component creation needed!
}
```

**3. Inject into Activities/Fragments**

```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var userRepository: UserRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // userRepository is automatically injected
    }
}

@AndroidEntryPoint
class HomeFragment : Fragment() {
    @Inject
    lateinit var userRepository: UserRepository
}
```

### Hilt Modules

Hilt modules use `@InstallIn` to specify which component they belong to:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .build()
    }
}

@Module
@InstallIn(ActivityComponent::class)
object ActivityModule {
    @Provides
    @ActivityScoped
    fun provideActivityData(): ActivityData {
        return ActivityData()
    }
}
```

### Hilt Scopes

Hilt provides predefined scopes that align with Android component lifecycles:

| Scope | Component | Lifetime |
|-------|-----------|----------|
| `@Singleton` | `SingletonComponent` | Application lifetime |
| `@ActivityScoped` | `ActivityComponent` | Activity lifetime |
| `@FragmentScoped` | `FragmentComponent` | Fragment lifetime |
| `@ViewScoped` | `ViewComponent` | View lifetime |
| `@ServiceScoped` | `ServiceComponent` | Service lifetime |
| `@ViewModelScoped` | `ViewModelComponent` | ViewModel lifetime |

### Hilt with ViewModel

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    // ViewModel code
}

// In Activity/Fragment
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
}
```

### Hilt Entry Points

For classes that can't use `@AndroidEntryPoint` (like custom views):

```kotlin
@EntryPoint
@InstallIn(SingletonComponent::class)
interface UserRepositoryEntryPoint {
    fun getUserRepository(): UserRepository
}

// Usage
class CustomView @JvmOverloads constructor(
    context: Context
) : View(context) {
    private val repository = EntryPointAccessors.fromApplication(
        context.applicationContext,
        UserRepositoryEntryPoint::class.java
    ).getUserRepository()
}
```

### Multi-Module with Hilt

Hilt simplifies multi-module setup:

```kotlin
// :core module
@Module
@InstallIn(SingletonComponent::class)
object CoreModule {
    @Provides
    @Singleton
    fun provideRepository(): UserRepository {
        return UserRepositoryImpl()
    }
}

// :feature module
@Module
@InstallIn(ActivityComponent::class)
object FeatureModule {
    @Provides
    @ActivityScoped
    fun provideFeatureData(): FeatureData {
        return FeatureData()
    }
}
```

No manual component creation or dependency management needed!

### Advanced Hilt Features

#### Multibinding

Multibinding allows you to bind multiple implementations to a collection.

**IntoSet - Multiple implementations as Set**:

```kotlin
interface Plugin {
    fun execute()
}

@Module
@InstallIn(SingletonComponent::class)
abstract class PluginModule {
    @Binds
    @IntoSet
    abstract fun bindAnalyticsPlugin(impl: AnalyticsPlugin): Plugin
    
    @Binds
    @IntoSet
    abstract fun bindLoggingPlugin(impl: LoggingPlugin): Plugin
}

class AnalyticsPlugin @Inject constructor() : Plugin {
    override fun execute() { /* Analytics logic */ }
}

class LoggingPlugin @Inject constructor() : Plugin {
    override fun execute() { /* Logging logic */ }
}

// Inject all plugins
class PluginManager @Inject constructor(
    private val plugins: Set<@JvmSuppressWildcards Plugin>
) {
    fun executeAll() {
        plugins.forEach { it.execute() }
    }
}
```

**IntoMap - Multiple implementations as Map**:

```kotlin
@MapKey
annotation class ViewModelKey(val value: KClass<out ViewModel>)

@Module
@InstallIn(SingletonComponent::class)
abstract class ViewModelModule {
    @Binds
    @IntoMap
    @ViewModelKey(UserViewModel::class)
    abstract fun bindUserViewModel(viewModel: UserViewModel): ViewModel
    
    @Binds
    @IntoMap
    @ViewModelKey(ProfileViewModel::class)
    abstract fun bindProfileViewModel(viewModel: ProfileViewModel): ViewModel
}

// Custom ViewModelFactory using map
class ViewModelFactory @Inject constructor(
    private val viewModels: Map<Class<out ViewModel>, @JvmSuppressWildcards Provider<ViewModel>>
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        val provider = viewModels[modelClass]
            ?: throw IllegalArgumentException("Unknown ViewModel: $modelClass")
        @Suppress("UNCHECKED_CAST")
        return provider.get() as T
    }
}
```

#### Qualifiers

Qualifiers distinguish between multiple bindings of the same type.

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthInterceptor

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class LoggingInterceptor

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @AuthInterceptor
    fun provideAuthInterceptor(): Interceptor {
        return Interceptor { chain ->
            val request = chain.request().newBuilder()
                .addHeader("Authorization", "Bearer $token")
                .build()
            chain.proceed(request)
        }
    }
    
    @Provides
    @LoggingInterceptor
    fun provideLoggingInterceptor(): Interceptor {
        return HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY
        }
    }
    
    @Provides
    @Singleton
    fun provideOkHttpClient(
        @AuthInterceptor authInterceptor: Interceptor,
        @LoggingInterceptor loggingInterceptor: Interceptor
    ): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(authInterceptor)
            .addInterceptor(loggingInterceptor)
            .build()
    }
}
```

**Named Qualifier** (simpler alternative):

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object UrlModule {
    @Provides
    @Named("base_url")
    fun provideBaseUrl(): String = "https://api.example.com/"
    
    @Provides
    @Named("image_url")
    fun provideImageUrl(): String = "https://images.example.com/"
}

class ApiService @Inject constructor(
    @Named("base_url") private val baseUrl: String,
    @Named("image_url") private val imageUrl: String
)
```

#### Assisted Injection

For dependencies that require runtime parameters along with injected dependencies.

```kotlin
// Define factory interface
interface UserDetailViewModelFactory {
    fun create(userId: String): UserDetailViewModel
}

// ViewModel with assisted parameter
class UserDetailViewModel @AssistedInject constructor(
    @Assisted private val userId: String,
    private val repository: UserRepository
) : ViewModel() {
    
    @AssistedFactory
    interface Factory : UserDetailViewModelFactory {
        override fun create(userId: String): UserDetailViewModel
    }
    
    init {
        loadUser(userId)
    }
}

// In Activity/Fragment
@AndroidEntryPoint
class UserDetailActivity : AppCompatActivity() {
    @Inject
    lateinit var viewModelFactory: UserDetailViewModel.Factory
    
    private lateinit var viewModel: UserDetailViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val userId = intent.getStringExtra("user_id") ?: ""
        viewModel = ViewModelProvider(
            this,
            object : ViewModelProvider.Factory {
                override fun <T : ViewModel> create(modelClass: Class<T>): T {
                    @Suppress("UNCHECKED_CAST")
                    return viewModelFactory.create(userId) as T
                }
            }
        )[UserDetailViewModel::class.java]
    }
}
```

#### Testing with Hilt

**Setup Test Dependencies**:

```gradle
dependencies {
    // Hilt testing
    androidTestImplementation 'com.google.dagger:hilt-android-testing:2.x'
    kaptAndroidTest 'com.google.dagger:hilt-android-compiler:2.x'
}
```

**Create Test Application**:

```kotlin
@CustomTestApplication(BaseApplication::class)
interface HiltTestApplication

// Generated by Hilt
class HiltTestApplication_Application : Application()
```

**Write Tests with Custom Modules**:

```kotlin
@HiltAndroidTest
@RunWith(AndroidJUnit4::class)
class RepositoryTest {
    @get:Rule
    var hiltRule = HiltAndroidRule(this)
    
    @Inject
    lateinit var repository: UserRepository
    
    @Before
    fun init() {
        hiltRule.inject()
    }
    
    @Test
    fun testRepository() {
        // Test using injected repository
        val users = repository.getUsers()
        assertThat(users).isNotEmpty()
    }
}

// Replace module for testing
@Module
@TestInstallIn(
    components = [SingletonComponent::class],
    replaces = [NetworkModule::class]
)
object TestNetworkModule {
    @Provides
    @Singleton
    fun provideTestApiService(): ApiService {
        return MockApiService()
    }
}
```

**Test ViewModel with Mocks**:

```kotlin
@HiltAndroidTest
@RunWith(AndroidJUnit4::class)
class UserViewModelTest {
    @get:Rule
    var hiltRule = HiltAndroidRule(this)
    
    @Inject
    lateinit var viewModelFactory: UserViewModel.Factory
    
    private lateinit var viewModel: UserViewModel
    
    @Before
    fun setup() {
        hiltRule.inject()
        viewModel = viewModelFactory.create()
    }
    
    @Test
    fun `loadUser should update state`() = runTest {
        viewModel.loadUser("123")
        
        val state = viewModel.uiState.value
        assertThat(state.user).isNotNull()
    }
}

@Module
@TestInstallIn(
    components = [SingletonComponent::class],
    replaces = [RepositoryModule::class]
)
object TestRepositoryModule {
    @Provides
    @Singleton
    fun provideTestRepository(): UserRepository {
        return mockk {
            coEvery { getUser(any()) } returns User("123", "Test User")
        }
    }
}
```

#### Real-World Patterns

**Pattern 1: Environment-Based Configuration**

```kotlin
enum class Environment {
    DEV, STAGING, PRODUCTION
}

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AppEnvironment

@Module
@InstallIn(SingletonComponent::class)
object EnvironmentModule {
    @Provides
    @AppEnvironment
    fun provideEnvironment(): Environment {
        return when (BuildConfig.BUILD_TYPE) {
            "debug" -> Environment.DEV
            "staging" -> Environment.STAGING
            else -> Environment.PRODUCTION
        }
    }
    
    @Provides
    @Singleton
    fun provideBaseUrl(@AppEnvironment env: Environment): String {
        return when (env) {
            Environment.DEV -> "https://dev-api.example.com/"
            Environment.STAGING -> "https://staging-api.example.com/"
            Environment.PRODUCTION -> "https://api.example.com/"
        }
    }
}
```

**Pattern 2: Multiple Data Sources with Repository**

```kotlin
interface DataSource {
    suspend fun getData(): List<Item>
}

class RemoteDataSource @Inject constructor(
    private val api: ApiService
) : DataSource {
    override suspend fun getData(): List<Item> = api.getItems()
}

class LocalDataSource @Inject constructor(
    private val dao: ItemDao
) : DataSource {
    override suspend fun getData(): List<Item> = dao.getAllItems()
}

@Module
@InstallIn(SingletonComponent::class)
abstract class DataSourceModule {
    @Binds
    @Named("remote")
    abstract fun bindRemoteDataSource(impl: RemoteDataSource): DataSource
    
    @Binds
    @Named("local")
    abstract fun bindLocalDataSource(impl: LocalDataSource): DataSource
}

class ItemRepository @Inject constructor(
    @Named("remote") private val remoteDataSource: DataSource,
    @Named("local") private val localDataSource: DataSource
) {
    suspend fun getItems(): List<Item> {
        return try {
            // Try remote first
            val items = remoteDataSource.getData()
            // Cache locally
            cacheLocally(items)
            items
        } catch (e: Exception) {
            // Fallback to local
            localDataSource.getData()
        }
    }
}
```

**Pattern 3: Feature-Based Modules**

```kotlin
// :feature:auth module
@Module
@InstallIn(SingletonComponent::class)
abstract class AuthModule {
    @Binds
    @Singleton
    abstract fun bindAuthRepository(impl: AuthRepositoryImpl): AuthRepository
}

@HiltViewModel
class LoginViewModel @Inject constructor(
    private val authRepository: AuthRepository
) : ViewModel()

// :feature:profile module
@Module
@InstallIn(SingletonComponent::class)
abstract class ProfileModule {
    @Binds
    @Singleton
    abstract fun bindProfileRepository(impl: ProfileRepositoryImpl): ProfileRepository
}

@HiltViewModel
class ProfileViewModel @Inject constructor(
    private val profileRepository: ProfileRepository,
    private val authRepository: AuthRepository // Can depend on other features
) : ViewModel()
```

**Pattern 4: Conditional Bindings**

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object ConditionalModule {
    @Provides
    @Singleton
    fun provideAnalytics(): Analytics {
        return if (BuildConfig.ENABLE_ANALYTICS) {
            FirebaseAnalytics()
        } else {
            NoOpAnalytics()
        }
    }
    
    @Provides
    @Singleton
    fun provideLogger(context: Application): Logger {
        return if (BuildConfig.DEBUG) {
            ConsoleLogger()
        } else {
            RemoteLogger(context)
        }
    }
}
```

---

## Koin - Lightweight DI for Kotlin

Koin is a pragmatic lightweight dependency injection framework for Kotlin developers.

Reference: [Koin Documentation](https://insert-koin.io/)

### Key Features

- ✅ **No code generation**: Uses Kotlin DSL and reflection
- ✅ **Lightweight**: Small library size
- ✅ **Easy to learn**: Simple, intuitive API
- ✅ **Faster compilation**: No annotation processing

### Setup

**1. Add Dependencies**

```gradle
dependencies {
    implementation 'io.insert-koin:koin-android:3.x'
    implementation 'io.insert-koin:koin-androidx-viewmodel:3.x'
}
```

**2. Define Modules**

```kotlin
val appModule = module {
    // Singleton (one instance for the whole app)
    single { UserRepository(get()) }
    single { ApiService(get()) }
    single { createRetrofit() }
}

val viewModelModule = module {
    // Factory (new instance each time)
    viewModel { UserViewModel(get()) }
}
```

**3. Start Koin**

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@MyApplication)
            modules(appModule, viewModelModule)
        }
    }
}
```

**4. Inject Dependencies**

```kotlin
// Property injection
class MainActivity : AppCompatActivity() {
    private val repository: UserRepository by inject()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // repository is ready
    }
}

// Constructor injection
class UserRepository(
    private val apiService: ApiService
) {
    // ...
}

// ViewModel
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    // ...
}

// In Activity
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModel()
}
```

### Koin Scopes

```kotlin
// Activity scope
val activityModule = module {
    scope<MainActivity> {
        scoped { ActivityData() }
    }
}

// Usage
class MainActivity : AppCompatActivity() {
    private val activityData: ActivityData by inject() // Gets scoped instance
}
```

### Koin Named/Qualified Dependencies

```kotlin
val module = module {
    single(named("production")) { ApiService("https://api.prod.com") }
    single(named("staging")) { ApiService("https://api.staging.com") }
}

// Usage
class Repository(private val apiService: ApiService) {
    // ...
}
// Injection
val repository = get<Repository>(named("production"))
```

---

## Framework Comparison

### Dagger 2.0 vs Hilt vs Koin

| Feature | Dagger 2.0 | Hilt | Koin |
|---------|-----------|------|------|
| **Type Safety** | ✅ Compile-time | ✅ Compile-time | ❌ Runtime (reflection) |
| **Code Generation** | ✅ Yes | ✅ Yes | ❌ No |
| **Boilerplate** | ❌ High | ✅ Low | ✅ Very Low |
| **Learning Curve** | ❌ Steep | ✅ Moderate | ✅ Easy |
| **Performance** | ✅ Excellent | ✅ Excellent | ✅ Good |
| **Build Time** | ❌ Slower (annotation processing) | ❌ Slower (annotation processing) | ✅ Faster (no code gen) |
| **Android Integration** | ⚠️ Manual setup | ✅ Automatic | ✅ Easy setup |
| **Multi-Module** | ⚠️ Complex | ✅ Simplified | ✅ Simple |
| **Kotlin DSL** | ❌ No | ❌ No | ✅ Yes |
| **Debugging** | ⚠️ Generated code | ⚠️ Generated code | ✅ Easy (readable code) |
| **When to Use** | Full control needed | Standard Android apps | Quick setup, small projects |

### Migration Summary

#### How Hilt Achieves the Same as Dagger:

| Dagger Concept | Hilt Equivalent |
|----------------|----------------|
| `@Component` | Auto-generated components (e.g., `SingletonComponent`) |
| Manual component creation | `@HiltAndroidApp` auto-generates app component |
| `@ContributesAndroidInjector` | `@AndroidEntryPoint` |
| Custom scopes | Predefined scopes (`@Singleton`, `@ActivityScoped`, etc.) |
| Component dependencies | `@InstallIn` annotation |
| Manual injection | Automatic injection with `@AndroidEntryPoint` |

#### How Koin Achieves the Same:

| Dagger Concept | Koin Equivalent |
|----------------|----------------|
| `@Component` + `@Module` | `module { }` DSL |
| `@Provides` | `single { }`, `factory { }`, `viewModel { }` |
| `@Inject` constructor | Constructor parameters in `get()` |
| `@Inject` field | `by inject()` property delegate |
| Scopes | `scope<Activity> { }` |
| Component interfaces | `startKoin { modules(...) }` |

---

## Best Practices

### Dagger Best Practices

1. **Prefer Constructor Injection**
   ```kotlin
   // Good
   class UserRepository @Inject constructor(
       private val apiService: ApiService
   )
   
   // Avoid when possible
   class UserRepository {
       @Inject
       lateinit var apiService: ApiService
   }
   ```

2. **Use @Binds for Interfaces**
   ```kotlin
   @Module
   abstract class DataModule {
       @Binds
       abstract fun bindRepository(
           impl: UserRepositoryImpl
       ): UserRepository
   }
   ```

3. **Scope Appropriately**
   - Use `@Singleton` only when truly needed
   - Use custom scopes for feature-specific dependencies

4. **Keep Modules Focused**
   ```kotlin
   // Good: One responsibility
   @Module
   class NetworkModule { }
   
   @Module
   class DatabaseModule { }
   
   // Bad: Mixed responsibilities
   @Module
   class EverythingModule { }
   ```

### Hilt Best Practices

1. **Use @InstallIn Appropriately**
   - `SingletonComponent` for app-wide dependencies
   - `ActivityComponent` for activity-scoped dependencies

2. **Leverage Predefined Scopes**
   - Don't create custom scopes unless necessary
   - Use `@ViewModelScoped` for ViewModel dependencies

3. **Use @HiltViewModel**
   ```kotlin
   @HiltViewModel
   class UserViewModel @Inject constructor(
       private val repository: UserRepository
   ) : ViewModel()
   ```

### Koin Best Practices

1. **Organize Modules by Feature**
   ```kotlin
   val loginModule = module { }
   val homeModule = module { }
   val profileModule = module { }
   ```

2. **Use Appropriate Definitions**
   - `single` for singletons
   - `factory` for new instances
   - `viewModel` for ViewModels

3. **Avoid Over-Scoping**
   - Only use scopes when necessary
   - Default to singleton or factory

---

## Interview Questions

### Dagger 2.0 Interview Questions

#### Basic Questions

1. **What is Dependency Injection, and why is it important?**
   - DI is a pattern where objects receive dependencies from external sources rather than creating them
   - Benefits: Testability, maintainability, loose coupling, code reusability

2. **What is Dagger, and how does it work?**
   - Dagger is a compile-time DI framework
   - Uses annotation processing to generate code that wires dependencies
   - Provides type-safe dependency resolution at compile time

3. **Explain the difference between `@Inject`, `@Module`, and `@Component`.**
   - `@Inject`: Requests dependencies (constructor, field, method)
   - `@Module`: Provides dependencies that can't use constructor injection
   - `@Component`: Bridge between modules and injection targets

4. **What is the difference between `@Provides` and `@Binds`?**
   - `@Provides`: Method that creates and returns dependency instance
   - `@Binds`: Abstract method that binds implementation to interface (more efficient, no method body)

5. **How does Dagger resolve dependencies?**
   - Looks for `@Inject` constructor first
   - If not found, looks for `@Provides` method in modules
   - Builds dependency graph at compile time
   - Generates code to satisfy dependencies

#### Intermediate Questions

6. **What is scoping in Dagger, and why is it important?**
   - Scoping limits object lifetime to component lifetime
   - Same instance provided within scope
   - Important for: Performance (expensive objects), sharing state, lifecycle management

7. **Explain `@Singleton` scope. Can you have multiple `@Singleton` components?**
   - `@Singleton` makes instance unique within component
   - Yes, you can have multiple components with `@Singleton`, but instances are only shared within same component

8. **What is the difference between Component Dependencies and Subcomponents?**
   - **Component Dependencies**: Components depend on other components, must explicitly expose dependencies
   - **Subcomponents**: Inherit all dependencies from parent, no need to expose

9. **How do you inject dependencies into Activities and Fragments?**
   - Use `dagger-android` library
   - Extend `DaggerAppCompatActivity` or `DaggerFragment`
   - Or manually call `AndroidInjection.inject(this)`

10. **What is `@ContributesAndroidInjector`?**
    - Simplifies creation of subcomponents for Android classes
    - Generates `AndroidInjector` implementation
    - Reduces boilerplate

#### Advanced Questions

11. **How does Dagger handle circular dependencies?**
    - Dagger detects circular dependencies at compile time
    - Solution: Use `@Lazy` or `Provider` interface to break cycle
    ```kotlin
    class A @Inject constructor(@Lazy private val b: B)
    class B @Inject constructor(private val a: A)
    ```

12. **What is `@Lazy` and `Provider` in Dagger?**
    - `@Lazy`: Wraps dependency, creates instance on first access
    - `Provider<T>`: Factory interface, creates new instance each time `get()` is called
    - Useful for: Breaking circular dependencies, conditional instantiation

13. **How do you provide different implementations based on build variants?**
    ```kotlin
    @Module
    abstract class DataModule {
        @Binds
        @Named("production")
        abstract fun bindProductionRepo(impl: ProductionRepo): Repository
        
        @Binds
        @Named("debug")
        abstract fun bindDebugRepo(impl: DebugRepo): Repository
    }
    ```

14. **What are multibindings in Dagger?**
    - Allows binding multiple implementations to a collection
    ```kotlin
    @Module
    abstract class PluginModule {
        @Binds
        @IntoSet
        abstract fun bindPluginA(impl: PluginA): Plugin
        
        @Binds
        @IntoSet
        abstract fun bindPluginB(impl: PluginB): Plugin
    }
    ```

15. **How do you test code that uses Dagger?**
    - Create test modules that provide mock dependencies
    - Use `@Component.Builder` or `@Component.Factory` to allow test overrides
    - Use Dagger's test components or replace modules in tests

### Hilt Interview Questions

#### Basic Questions

1. **What is Hilt, and how does it relate to Dagger?**
   - Hilt is a DI library built on top of Dagger
   - Simplifies Dagger setup for Android
   - Provides standard components and annotations

2. **What is `@HiltAndroidApp`?**
   - Annotation for Application class
   - Triggers Hilt code generation
   - Creates application-level component

3. **What is `@AndroidEntryPoint`?**
   - Enables automatic dependency injection for Android classes
   - Can be used on Activities, Fragments, Views, Services, BroadcastReceivers
   - Replaces manual injection calls

4. **What are Hilt's predefined components?**
   - `SingletonComponent`: Application lifetime
   - `ActivityRetainedComponent`: Survives configuration changes
   - `ViewModelComponent`: ViewModel lifetime
   - `ActivityComponent`: Activity lifetime
   - `FragmentComponent`: Fragment lifetime
   - `ViewComponent`: View lifetime
   - `ServiceComponent`: Service lifetime

5. **What is `@InstallIn` annotation?**
   - Specifies which Hilt component a module belongs to
   - Determines where dependencies can be injected
   - Example: `@InstallIn(SingletonComponent::class)`

#### Intermediate Questions

6. **How do you inject ViewModels with Hilt?**
   ```kotlin
   @HiltViewModel
   class UserViewModel @Inject constructor(
       private val repository: UserRepository
   ) : ViewModel()
   
   // In Activity/Fragment
   private val viewModel: UserViewModel by viewModels()
   ```

7. **What is the difference between `@Singleton` and `@ActivityScoped` in Hilt?**
   - `@Singleton`: Lives for application lifetime
   - `@ActivityScoped`: Lives for activity lifetime, new instance per activity

8. **How does Hilt handle multi-module projects?**
   - Each module defines modules with `@InstallIn`
   - Hilt automatically aggregates modules
   - No manual component creation needed

9. **What are Hilt Entry Points?**
   - Interface to access Hilt-managed dependencies from classes that can't use `@AndroidEntryPoint`
   - Example: Custom Views, ContentProviders
   ```kotlin
   @EntryPoint
   @InstallIn(SingletonComponent::class)
   interface RepositoryEntryPoint {
       fun getRepository(): UserRepository
   }
   ```

10. **How do you provide different implementations for different build types with Hilt?**
    ```kotlin
    @Module
    @InstallIn(SingletonComponent::class)
    object NetworkModule {
        @Provides
        @Singleton
        fun provideApiService(): ApiService {
            return if (BuildConfig.DEBUG) {
                MockApiService()
            } else {
                RealApiService()
            }
        }
    }
    ```

#### Advanced Questions

11. **How do you migrate from Dagger to Hilt?**
    - Add Hilt dependencies
    - Annotate Application with `@HiltAndroidApp`
    - Replace `@Component` with `@InstallIn` modules
    - Replace `@ContributesAndroidInjector` with `@AndroidEntryPoint`
    - Remove manual component creation code

12. **What is `ActivityRetainedComponent` in Hilt?**
    - Component that survives configuration changes
    - Lives from Activity creation to Activity finish
    - Used by ViewModels internally

13. **How do you test with Hilt?**
    ```kotlin
    @HiltAndroidTest
    class MyTest {
        @get:Rule
        var hiltRule = HiltAndroidRule(this)
        
        @Before
        fun init() {
            hiltRule.inject()
        }
    }
    ```

14. **Can you use custom scopes in Hilt?**
    - Yes, but generally not recommended
    - Hilt provides predefined scopes that align with Android lifecycles
    - Custom scopes require custom components

15. **How does Hilt integrate with WorkManager?**
    ```kotlin
    @HiltWorker
    class MyWorker @AssistedInject constructor(
        @Assisted context: Context,
        @Assisted workerParams: WorkerParameters,
        private val repository: UserRepository
    ) : CoroutineWorker(context, workerParams)
    ```

### General Dependency Injection Questions

1. **What are the different types of dependency injection?**
   - Constructor Injection: Dependencies provided via constructor
   - Field Injection: Dependencies injected into fields
   - Method Injection: Dependencies injected via methods
   - Interface Injection: Dependencies provided via interface methods

2. **What are the advantages and disadvantages of compile-time vs runtime DI?**
   - **Compile-time (Dagger/Hilt)**: Type-safe, fast runtime, catches errors early, but slower builds
   - **Runtime (Koin)**: Faster builds, easier to learn, but less type-safe, potential runtime errors

3. **When would you choose Dagger over Hilt?**
   - Need full control over component structure
   - Custom component hierarchies
   - Legacy projects with complex Dagger setup
   - Non-standard Android architectures

4. **When would you choose Koin over Hilt/Dagger?**
   - Quick prototyping
   - Small to medium projects
   - Team prefers Kotlin DSL
   - Faster build times are critical
   - Simpler learning curve needed

5. **How do you handle dependency injection in a modular architecture?**
   - Define core dependencies in core module
   - Use component dependencies or subcomponents
   - Expose only necessary dependencies
   - Keep feature modules independent

### Architecture-Focused Scenario Questions

#### Scenario 1: Multi-Module App DI Structure

**Q: Design DI structure for an app with :app, :core:network, :core:database, :feature:auth, :feature:profile modules.**

**Answer:**

```kotlin
// :core:network module
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}

// :core:database module
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    @Provides
    @Singleton
    fun provideDatabase(app: Application): AppDatabase {
        return Room.databaseBuilder(
            app,
            AppDatabase::class.java,
            "app-db"
        ).build()
    }
    
    @Provides
    fun provideUserDao(db: AppDatabase): UserDao = db.userDao()
}

// :feature:auth module
interface AuthRepository {
    suspend fun login(email: String, password: String): Result<User>
}

class AuthRepositoryImpl @Inject constructor(
    private val api: ApiService,
    private val userDao: UserDao
) : AuthRepository {
    override suspend fun login(email: String, password: String): Result<User> {
        return try {
            val user = api.login(email, password)
            userDao.insert(user)
            Result.success(user)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

@Module
@InstallIn(SingletonComponent::class)
abstract class AuthModule {
    @Binds
    @Singleton
    abstract fun bindAuthRepository(impl: AuthRepositoryImpl): AuthRepository
}

@HiltViewModel
class LoginViewModel @Inject constructor(
    private val authRepository: AuthRepository
) : ViewModel() {
    fun login(email: String, password: String) {
        viewModelScope.launch {
            authRepository.login(email, password)
        }
    }
}

// :feature:profile module
interface ProfileRepository {
    suspend fun getProfile(): Profile
}

class ProfileRepositoryImpl @Inject constructor(
    private val api: ApiService,
    private val authRepository: AuthRepository // Depends on auth feature
) : ProfileRepository {
    override suspend fun getProfile(): Profile {
        // Can use authRepository for user context
        return api.getProfile()
    }
}

@Module
@InstallIn(SingletonComponent::class)
abstract class ProfileModule {
    @Binds
    @Singleton
    abstract fun bindProfileRepository(impl: ProfileRepositoryImpl): ProfileRepository
}

// :app module
@HiltAndroidApp
class MyApplication : Application()
```

**Key Principles:**
- Core modules provide base dependencies (network, database)
- Feature modules depend on core but are independent of each other
- Use interfaces for repositories to maintain loose coupling
- Hilt automatically aggregates all modules

#### Scenario 2: Repository with Multiple Data Sources

**Q: Implement a UserRepository that uses remote API as primary source and local database as cache/fallback, with proper DI setup.**

**Answer:**

```kotlin
// Define data source interfaces
interface UserDataSource {
    suspend fun getUser(id: String): User
    suspend fun getUsers(): List<User>
}

class UserRemoteDataSource @Inject constructor(
    private val api: ApiService
) : UserDataSource {
    override suspend fun getUser(id: String): User = api.getUser(id)
    override suspend fun getUsers(): List<User> = api.getUsers()
}

class UserLocalDataSource @Inject constructor(
    private val userDao: UserDao
) : UserDataSource {
    override suspend fun getUser(id: String): User = userDao.getUser(id)
    override suspend fun getUsers(): List<User> = userDao.getAllUsers()
}

// Qualifiers to distinguish data sources
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class RemoteDataSource

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class LocalDataSource

// Module to provide data sources
@Module
@InstallIn(SingletonComponent::class)
abstract class DataSourceModule {
    @Binds
    @RemoteDataSource
    abstract fun bindRemoteDataSource(impl: UserRemoteDataSource): UserDataSource
    
    @Binds
    @LocalDataSource
    abstract fun bindLocalDataSource(impl: UserLocalDataSource): UserDataSource
}

// Repository implementation
interface UserRepository {
    suspend fun getUser(id: String, forceRefresh: Boolean = false): Result<User>
    fun observeUsers(): Flow<List<User>>
}

class UserRepositoryImpl @Inject constructor(
    @RemoteDataSource private val remoteDataSource: UserDataSource,
    @LocalDataSource private val localDataSource: UserDataSource,
    private val networkMonitor: NetworkMonitor
) : UserRepository {
    
    override suspend fun getUser(id: String, forceRefresh: Boolean): Result<User> {
        return try {
            if (forceRefresh || networkMonitor.isOnline()) {
                // Fetch from remote
                val user = remoteDataSource.getUser(id)
                // Cache locally
                cacheUser(user)
                Result.success(user)
            } else {
                // Use cached data
                val user = localDataSource.getUser(id)
                Result.success(user)
            }
        } catch (e: Exception) {
            // Fallback to cache on error
            try {
                val cachedUser = localDataSource.getUser(id)
                Result.success(cachedUser)
            } catch (cacheError: Exception) {
                Result.failure(e)
            }
        }
    }
    
    override fun observeUsers(): Flow<List<User>> = flow {
        // Emit cached data first
        emit(localDataSource.getUsers())
        
        // Fetch and emit fresh data if online
        if (networkMonitor.isOnline()) {
            try {
                val users = remoteDataSource.getUsers()
                cacheUsers(users)
                emit(users)
            } catch (e: Exception) {
                // Keep showing cached data
            }
        }
    }
    
    private suspend fun cacheUser(user: User) {
        // Implementation
    }
    
    private suspend fun cacheUsers(users: List<User>) {
        // Implementation
    }
}

@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds
    @Singleton
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}
```

#### Scenario 3: Different API Base URLs for Dev/Staging/Prod

**Q: Configure Retrofit with different base URLs based on build variant using DI.**

**Answer:**

```kotlin
enum class BuildEnvironment {
    DEV, STAGING, PRODUCTION
}

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class BaseUrl

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class Environment

@Module
@InstallIn(SingletonComponent::class)
object ConfigModule {
    
    @Provides
    @Environment
    fun provideEnvironment(): BuildEnvironment {
        return when (BuildConfig.BUILD_TYPE) {
            "debug" -> BuildEnvironment.DEV
            "staging" -> BuildEnvironment.STAGING
            else -> BuildEnvironment.PRODUCTION
        }
    }
    
    @Provides
    @BaseUrl
    fun provideBaseUrl(@Environment env: BuildEnvironment): String {
        return when (env) {
            BuildEnvironment.DEV -> "https://dev-api.example.com/"
            BuildEnvironment.STAGING -> "https://staging-api.example.com/"
            BuildEnvironment.PRODUCTION -> "https://api.example.com/"
        }
    }
}

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideOkHttpClient(@Environment env: BuildEnvironment): OkHttpClient {
        return OkHttpClient.Builder()
            .apply {
                // Add logging only for non-production
                if (env != BuildEnvironment.PRODUCTION) {
                    addInterceptor(HttpLoggingInterceptor().apply {
                        level = HttpLoggingInterceptor.Level.BODY
                    })
                }
            }
            .addInterceptor { chain ->
                // Add auth header
                val request = chain.request().newBuilder()
                    .addHeader("Environment", env.name)
                    .build()
                chain.proceed(request)
            }
            .build()
    }
    
    @Provides
    @Singleton
    fun provideRetrofit(
        @BaseUrl baseUrl: String,
        okHttpClient: OkHttpClient
    ): Retrofit {
        return Retrofit.Builder()
            .baseUrl(baseUrl)
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .addCallAdapterFactory(CoroutineCallAdapterFactory())
            .build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}

// Alternative: Using BuildConfig with flavors
// build.gradle.kts
android {
    buildTypes {
        debug {
            buildConfigField("String", "API_BASE_URL", "\"https://dev-api.example.com/\"")
        }
        create("staging") {
            buildConfigField("String", "API_BASE_URL", "\"https://staging-api.example.com/\"")
        }
        release {
            buildConfigField("String", "API_BASE_URL", "\"https://api.example.com/\"")
        }
    }
}

// Then in module:
@Provides
@BaseUrl
fun provideBaseUrl(): String = BuildConfig.API_BASE_URL
```

#### Scenario 4: Testing Strategy with DI

**Q: Design testing strategy for ViewModels and Repositories using Hilt's testing support.**

**Answer:**

```kotlin
// Production module
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds
    @Singleton
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}

// Test module that replaces production
@Module
@TestInstallIn(
    components = [SingletonComponent::class],
    replaces = [RepositoryModule::class]
)
abstract class TestRepositoryModule {
    @Binds
    @Singleton
    abstract fun bindUserRepository(impl: FakeUserRepository): UserRepository
}

// Fake implementation for testing
class FakeUserRepository @Inject constructor() : UserRepository {
    private val users = mutableListOf<User>()
    var shouldThrowError = false
    
    override suspend fun getUser(id: String): Result<User> {
        return if (shouldThrowError) {
            Result.failure(Exception("Test error"))
        } else {
            val user = users.find { it.id == id }
            if (user != null) Result.success(user)
            else Result.failure(Exception("User not found"))
        }
    }
    
    fun addUser(user: User) {
        users.add(user)
    }
}

// ViewModel test
@HiltAndroidTest
@RunWith(AndroidJUnit4::class)
class UserViewModelTest {
    
    @get:Rule
    var hiltRule = HiltAndroidRule(this)
    
    @Inject
    lateinit var repository: UserRepository
    
    private lateinit var viewModel: UserViewModel
    
    @Before
    fun setup() {
        hiltRule.inject()
        viewModel = UserViewModel(repository)
        
        // Setup fake data
        val fakeRepo = repository as FakeUserRepository
        fakeRepo.addUser(User("1", "John Doe"))
    }
    
    @Test
    fun `loadUser should update state with user data`() = runTest {
        viewModel.loadUser("1")
        
        val state = viewModel.uiState.value
        assertThat(state).isInstanceOf(UiState.Success::class.java)
        assertThat((state as UiState.Success).data.name).isEqualTo("John Doe")
    }
    
    @Test
    fun `loadUser should handle error`() = runTest {
        val fakeRepo = repository as FakeUserRepository
        fakeRepo.shouldThrowError = true
        
        viewModel.loadUser("1")
        
        val state = viewModel.uiState.value
        assertThat(state).isInstanceOf(UiState.Error::class.java)
    }
}

// Integration test with real dependencies
@HiltAndroidTest
@RunWith(AndroidJUnit4::class)
class UserRepositoryIntegrationTest {
    
    @get:Rule
    var hiltRule = HiltAndroidRule(this)
    
    @Inject
    lateinit var database: AppDatabase
    
    @Inject
    lateinit var repository: UserRepository
    
    @Before
    fun setup() {
        hiltRule.inject()
    }
    
    @After
    fun tearDown() {
        database.close()
    }
    
    @Test
    fun `repository should cache user locally`() = runTest {
        val user = User("1", "John Doe")
        
        repository.saveUser(user)
        
        val cachedUser = database.userDao().getUser("1")
        assertThat(cachedUser).isEqualTo(user)
    }
}

// Custom test module for specific test case
@Module
@TestInstallIn(
    components = [SingletonComponent::class],
    replaces = [NetworkModule::class]
)
object CustomTestNetworkModule {
    
    @Provides
    @Singleton
    fun provideTestApiService(): ApiService {
        return mockk {
            coEvery { getUser(any()) } returns User("1", "Test User")
            coEvery { getUsers() } returns listOf(
                User("1", "User 1"),
                User("2", "User 2")
            )
        }
    }
}
```

**Intent of Architecture DI Questions:**
These scenarios test:
- Understanding of DI in multi-module architecture
- Ability to design clean, testable dependency graphs
- Knowledge of qualifiers and conditional bindings
- Practical testing strategies with DI
- Real-world problem-solving with dependency management

---

## References

- [Dagger Basics - Android Developers](https://developer.android.com/training/dependency-injection/dagger-basics)
- [Using Dagger in Android Apps - Android Developers](https://developer.android.com/training/dependency-injection/dagger-android)
- [Using Dagger in Multi-Module Apps - Android Developers](https://developer.android.com/training/dependency-injection/dagger-multi-module)
- [Hilt Documentation - Android Developers](https://developer.android.com/training/dependency-injection/hilt-android)
- [Koin Documentation](https://insert-koin.io/)
- [Dagger GitHub](https://github.com/google/dagger)
- [Hilt GitHub](https://github.com/google/dagger/tree/master/java/dagger/hilt)

---

## Quick Reference

| Concept | Dagger | Hilt | Koin |
|---------|--------|------|------|
| **Application Setup** | Manual component creation | `@HiltAndroidApp` | `startKoin { }` |
| **Inject Activity** | `@ContributesAndroidInjector` | `@AndroidEntryPoint` | `by inject()` |
| **Module Definition** | `@Module` class | `@Module` + `@InstallIn` | `module { }` |
| **Provide Dependency** | `@Provides` method | `@Provides` method | `single { }` / `factory { }` |
| **Scope Annotation** | Custom scopes | Predefined scopes | `scope<Type> { }` |
| **ViewModel** | Custom factory | `@HiltViewModel` | `viewModel { }` |
| **Constructor Injection** | `@Inject constructor` | `@Inject constructor` | Constructor parameters |

---

## Key Takeaways

1. **Dagger 2.0** provides full control but requires more boilerplate
2. **Hilt** simplifies Dagger for Android with standard components and predefined scopes
3. **Koin** offers lightweight DI with Kotlin DSL and no code generation
4. **Choose based on**: Project size, team expertise, build time constraints, type safety requirements
5. **Best practice**: Prefer constructor injection, scope appropriately, organize modules by feature
6. **Multibinding** enables extensible architectures with sets and maps of dependencies
7. **Qualifiers** distinguish between multiple bindings of the same type
8. **Assisted Injection** combines dependency injection with runtime parameters
9. **Testing** is easier with DI - use test modules and mocks effectively
10. For new Android projects, **Hilt is the recommended approach**

---

## Interview Preparation Checklist

✅ Understand the difference between Dagger, Hilt, and Koin
✅ Know when to use @Inject, @Provides, and @Binds
✅ Understand scopes and component hierarchy
✅ Practice implementing multi-module DI structure
✅ Know how to test code with DI (mock injection)
✅ Understand multibinding and qualifiers
✅ Can explain trade-offs between compile-time vs runtime DI
✅ Know how to migrate from Dagger to Hilt
✅ Understand assisted injection for runtime parameters
✅ Can design DI architecture for complex apps

---

*Last Updated: December 2025*

*Version 2.0 - Enhanced with Advanced Topics, Real-World Patterns, and Architecture Scenarios*


