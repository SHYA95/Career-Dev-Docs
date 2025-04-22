# Kotlin Multiplatform (KMP) with MVI Architecture and Repository Pattern

##  🚀 Overview
This document outlines the implementation of Model-View-Intent (MVI) architecture in a Kotlin Multiplatform (KMP) project, along with the Repository pattern and Revision use cases. This architecture enables sharing business logic across platforms while maintaining platform-specific UI implementations.

## 🔑 Key Concepts

### MVI Architecture
MVI is a unidirectional data flow architecture with these core components:
- **Model**: Immutable state representation
- **View**: UI that renders the state
- **Intent**: User actions that trigger state changes
- **Reducer**: Pure function that produces new state
- **Effect**: Side effects like navigation or notifications

### Repository Pattern
A clean API abstraction for data access that:
- Separates data sources from business logic
- Provides a single entry point for data operations
- Handles caching and data synchronization

### Revision Use Case
A specific business logic implementation that:
- Manages document revisions
- Tracks changes to documents
- Provides history and rollback capabilities

## 📊 Project Structure

```
project/
├── shared/                      # Multiplatform module
│   ├── src/
│   │   ├── commonMain/          # Common code for all platforms
│   │   │   ├── domain/
│   │   │   │   ├── model/       # Business models
│   │   │   │   ├── repository/  # Repository interfaces
│   │   │   │   └── usecase/     # Business logic
│   │   │   ├── presentation/
│   │   │   │   ├── mvi/         # MVI components
│   │   │   │   └── viewmodel/   # ViewModels
│   │   │   └── data/
│   │   │       ├── repository/  # Repository implementations
│   │   │       ├── source/      # Data sources
│   │   │       └── mapper/      # Data mappers
│   │   ├── androidMain/         # Android-specific code
│   │   └── iosMain/             # iOS-specific code
│   └── build.gradle.kts         # Multiplatform configuration
├── androidApp/                  # Android application
└── iosApp/                      # iOS application
```

## 💡 Implementation Examples

### MVI Components

```kotlin
// In commonMain/presentation/mvi/model
data class DocumentState(
    val isLoading: Boolean = false,
    val document: Document? = null,
    val revisions: List<Revision> = emptyList(),
    val error: String? = null
)

// In commonMain/presentation/mvi/intent
sealed class DocumentIntent {
    data class LoadDocument(val id: String) : DocumentIntent()
    data class ReviseDocument(val documentId: String, val changes: DocumentChanges) : DocumentIntent()
    data class RevisionCreated(val revision: Revision) : DocumentIntent()
    data class Error(val message: String) : DocumentIntent()
}

// In commonMain/presentation/mvi/effect
sealed class DocumentEffect {
    data class Navigate(val route: String) : DocumentEffect()
    data class ShowToast(val message: String) : DocumentEffect()
}
```

### MVI Store

```kotlin
// In commonMain/presentation/mvi
class MviStore<State, Intent, Effect>(
    initialState: State,
    private val reducer: (State, Intent) -> Pair<State, Set<Effect>>,
    scope: CoroutineScope
) {
    private val _state = MutableStateFlow(initialState)
    val state: StateFlow<State> = _state.asStateFlow()
    
    private val _effects = MutableSharedFlow<Effect>()
    val effects: SharedFlow<Effect> = _effects.asSharedFlow()
    
    private val intents = MutableSharedFlow<Intent>()
    
    init {
        scope.launch {
            intents.collect { intent ->
                val (newState, newEffects) = reducer(state.value, intent)
                _state.value = newState
                newEffects.forEach { effect ->
                    _effects.emit(effect)
                }
            }
        }
    }
    
    fun dispatch(intent: Intent) {
        scope.launch {
            intents.emit(intent)
        }
    }
}
```

### Repository Pattern

```kotlin
// In commonMain/domain/repository
interface DocumentRepository {
    suspend fun getDocument(id: String): Result<Document>
    suspend fun updateDocument(document: Document): Result<Document>
    suspend fun getAllDocuments(): Flow<List<Document>>
}

// In commonMain/domain/repository
interface RevisionRepository {
    suspend fun saveRevision(revision: Revision): Result<Revision>
    suspend fun getRevisions(documentId: String): Result<List<Revision>>
    suspend fun getRevision(id: String): Result<Revision>
}

// In commonMain/data/repository
class DocumentRepositoryImpl(
    private val remoteDataSource: DocumentRemoteDataSource,
    private val localDataSource: DocumentLocalDataSource
) : DocumentRepository {
    override suspend fun getDocument(id: String): Result<Document> {
        return try {
            // Try local first
            localDataSource.getDocument(id).let { localDoc ->
                if (localDoc != null) {
                    Result.success(localDoc)
                } else {
                    // Fallback to remote
                    val remoteDoc = remoteDataSource.fetchDocument(id)
                    localDataSource.saveDocument(remoteDoc)
                    Result.success(remoteDoc)
                }
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    // Other implementation methods...
}
```

### Revision Use Case

```kotlin
// In commonMain/domain/usecase
class ReviseDocumentUseCase(
    private val documentRepository: DocumentRepository,
    private val revisionRepository: RevisionRepository
) {
    suspend operator fun invoke(documentId: String, changes: DocumentChanges): Result<Revision> {
        return try {
            // 1. Get the current document
            val document = documentRepository.getDocument(documentId)
                .getOrElse { return Result.failure(it) }
            
            // 2. Create a revision
            val revision = Revision(
                id = UUID.randomUUID().toString(),
                documentId = documentId,
                changes = changes,
                timestamp = Clock.System.now()
            )
            
            // 3. Save the revision
            revisionRepository.saveRevision(revision)
                .getOrElse { return Result.failure(it) }
            
            // 4. Update the document with changes
            val updatedDocument = document.applyChanges(changes)
            documentRepository.updateDocument(updatedDocument)
                .getOrElse { return Result.failure(it) }
            
            Result.success(revision)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### ViewModel

```kotlin
// In commonMain/presentation/viewmodel
class DocumentViewModel(
    private val getDocumentUseCase: GetDocumentUseCase,
    private val reviseDocumentUseCase: ReviseDocumentUseCase,
    private val getRevisionsUseCase: GetRevisionsUseCase,
    coroutineScope: CoroutineScope
) {
    private val store = MviStore<DocumentState, DocumentIntent, DocumentEffect>(
        initialState = DocumentState(isLoading = true),
        reducer = ::reduce,
        scope = coroutineScope
    )
    
    val state = store.state
    val effects = store.effects
    
    fun dispatch(intent: DocumentIntent) {
        store.dispatch(intent)
    }
    
    private fun reduce(state: DocumentState, intent: DocumentIntent): Pair<DocumentState, Set<DocumentEffect>> {
        return when (intent) {
            is DocumentIntent.LoadDocument -> {
                coroutineScope.launch {
                    getDocumentUseCase(intent.id).fold(
                        onSuccess = { document ->
                            getRevisionsUseCase(intent.id).fold(
                                onSuccess = { revisions ->
                                    dispatch(DocumentIntent.DocumentLoaded(document, revisions))
                                },
                                onFailure = { error ->
                                    dispatch(DocumentIntent.Error(error.message ?: "Failed to load revisions"))
                                }
                            )
                        },
                        onFailure = { error ->
                            dispatch(DocumentIntent.Error(error.message ?: "Failed to load document"))
                        }
                    )
                }
                state.copy(isLoading = true) to emptySet()
            }
            
            is DocumentIntent.ReviseDocument -> {
                coroutineScope.launch {
                    reviseDocumentUseCase(intent.documentId, intent.changes).fold(
                        onSuccess = { revision ->
                            dispatch(DocumentIntent.RevisionCreated(revision))
                        },
                        onFailure = { error ->
                            dispatch(DocumentIntent.Error(error.message ?: "Failed to revise document"))
                        }
                    )
                }
                state.copy(isLoading = true) to emptySet()
            }
            
            is DocumentIntent.RevisionCreated -> {
                val updatedRevisions = state.revisions + intent.revision
                state.copy(
                    isLoading = false,
                    revisions = updatedRevisions
                ) to setOf(DocumentEffect.ShowToast("Revision created successfully"))
            }
            
            is DocumentIntent.Error -> {
                state.copy(
                    isLoading = false,
                    error = intent.message
                ) to setOf(DocumentEffect.ShowToast(intent.message))
            }
            
            // Handle other intents...
        }
    }
}
```

## 🔄 Platform Integration

### Android Integration

```kotlin
// In androidApp
@Composable
fun DocumentScreen(viewModel: DocumentViewModel) {
    val state by viewModel.state.collectAsState()
    
    LaunchedEffect(Unit) {
        viewModel.effects.collect { effect ->
            when (effect) {
                is DocumentEffect.ShowToast -> {
                    Toast.makeText(LocalContext.current, effect.message, Toast.LENGTH_SHORT).show()
                }
                is DocumentEffect.Navigate -> {
                    // Handle navigation
                }
            }
        }
    }
    
    when {
        state.isLoading -> {
            CircularProgressIndicator()
        }
        state.error != null -> {
            Text("Error: ${state.error}")
        }
        state.document != null -> {
            DocumentContent(
                document = state.document!!,
                revisions = state.revisions,
                onRevise = { changes ->
                    viewModel.dispatch(DocumentIntent.ReviseDocument(state.document!!.id, changes))
                }
            )
        }
    }
}
```

### iOS Integration

```swift
// In iosApp
import SwiftUI
import shared

struct DocumentView: View {
    @ObservedObject private var viewModel: DocumentViewModelWrapper
    
    init(documentId: String) {
        self.viewModel = DocumentViewModelWrapper(documentId: documentId)
    }
    
    var body: some View {
        let state = viewModel.state
        
        return Group {
            if state.isLoading {
                ProgressView()
            } else if let error = state.error {
                Text("Error: \(error)")
            } else if let document = state.document {
                DocumentContentView(
                    document: document,
                    revisions: state.revisions,
                    onRevise: { changes in
                        viewModel.reviseDocument(documentId: document.id, changes: changes)
                    }
                )
            }
        }
        .onAppear {
            viewModel.observeEffects()
        }
    }
}

class DocumentViewModelWrapper: ObservableObject {
    private let viewModel: DocumentViewModel
    @Published var state: DocumentState = DocumentState(isLoading: true)
    
    init(documentId: String) {
        self.viewModel = DocumentViewModel(
            getDocumentUseCase: GetDocumentUseCase(/* dependencies */),
            reviseDocumentUseCase: ReviseDocumentUseCase(/* dependencies */),
            getRevisionsUseCase: GetRevisionsUseCase(/* dependencies */),
            coroutineScope: CoroutineScopeProvider.shared.scope
        )
        
        // Observe state changes
        viewModel.state.watch { [weak self] state in
            self?.state = state
        }
        
        // Load document
        viewModel.dispatch(intent: DocumentIntent.LoadDocument(id: documentId))
    }
    
    func reviseDocument(documentId: String, changes: DocumentChanges) {
        viewModel.dispatch(intent: DocumentIntent.ReviseDocument(
            documentId: documentId,
            changes: changes
        ))
    }
    
    func observeEffects() {
        viewModel.effects.watch { effect in
            if let showToast = effect as? DocumentEffect.ShowToast {
                // Show toast on iOS
            } else if let navigate = effect as? DocumentEffect.Navigate {
                // Handle navigation
            }
        }
    }
}
```

## 🌟 Benefits

- **Code Sharing**: Business logic, repositories, and use cases shared across platforms
- **Separation of Concerns**: Clear boundaries between data, domain, and presentation layers
- **Testability**: Pure functions and immutable state make testing easier
- **Predictable State Management**: Unidirectional data flow ensures predictable state changes
- **Maintainability**: Well-defined architecture makes the codebase easier to maintain

## 🛠️ Best Practices

1. Keep platform-specific code to a minimum
2. Use dependency injection for better testability
3. Handle errors consistently across the application
4. Implement proper caching strategies in repositories
5. Use coroutines and flows for asynchronous operations
6. Create small, focused use cases for business logic
7. Make state immutable to prevent unexpected changes

## 🔗 Additional Resources
https://medium.com/swlh/mvi-architecture-with-android-fcde123e3c4a
