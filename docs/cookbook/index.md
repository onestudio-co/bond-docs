# Recipes

## Introduction

This cookbook provides practical, copy-and-paste solutions for common mobile app patterns using Bond. Each recipe is a complete, tested implementation that you can adapt to your specific needs.

## Data Patterns

### Infinite Scroll with Pagination

Implement smooth infinite scrolling with automatic loading:

```dart
class PostsController extends StateNotifier<PostsState> {
  final PostsRepository _repository;
  final ScrollController scrollController = ScrollController();
  
  PostsController(this._repository) : super(PostsState.initial()) {
    scrollController.addListener(_onScroll);
    loadInitial();
  }

  void _onScroll() {
    if (scrollController.position.pixels >= 
        scrollController.position.maxScrollExtent - 200) {
      loadMore();
    }
  }

  Future<void> loadInitial() async {
    state = state.copyWith(isLoading: true, error: null);
    
    try {
      final result = await _repository.getPosts(page: 1);
      state = state.copyWith(
        posts: result.items,
        currentPage: 1,
        hasMore: result.hasMore,
        isLoading: false,
      );
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoading: false);
    }
  }

  Future<void> loadMore() async {
    if (state.isLoadingMore || !state.hasMore) return;
    
    state = state.copyWith(isLoadingMore: true);
    
    try {
      final result = await _repository.getPosts(page: state.currentPage + 1);
      state = state.copyWith(
        posts: [...state.posts, ...result.items],
        currentPage: state.currentPage + 1,
        hasMore: result.hasMore,
        isLoadingMore: false,
      );
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoadingMore: false);
    }
  }

  @override
  void dispose() {
    scrollController.dispose();
    super.dispose();
  }
}

// Usage in widget
class PostsList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final controller = ref.watch(postsControllerProvider.notifier);
    final state = ref.watch(postsControllerProvider);
    
    return ListView.builder(
      controller: controller.scrollController,
      itemCount: state.posts.length + (state.hasMore ? 1 : 0),
      itemBuilder: (context, index) {
        if (index >= state.posts.length) {
          return Center(child: CircularProgressIndicator());
        }
        
        return PostCard(post: state.posts[index]);
      },
    );
  }
}
```

### Optimistic Updates

Update UI immediately, sync with server later:

```dart
class PostsController extends StateNotifier<PostsState> {
  Future<void> likePost(String postId) async {
    final postIndex = state.posts.indexWhere((p) => p.id == postId);
    if (postIndex == -1) return;

    final originalPost = state.posts[postIndex];
    
    // Optimistic update
    final optimisticPost = originalPost.copyWith(
      isLiked: !originalPost.isLiked,
      likesCount: originalPost.isLiked 
          ? originalPost.likesCount - 1 
          : originalPost.likesCount + 1,
    );
    
    final newPosts = [...state.posts];
    newPosts[postIndex] = optimisticPost;
    state = state.copyWith(posts: newPosts);

    try {
      // Sync with server
      final updatedPost = await _repository.likePost(postId);
      
      // Update with server response
      final finalPosts = [...state.posts];
      finalPosts[postIndex] = updatedPost;
      state = state.copyWith(posts: finalPosts);
    } catch (e) {
      // Revert on error
      final revertedPosts = [...state.posts];
      revertedPosts[postIndex] = originalPost;
      state = state.copyWith(posts: revertedPosts);
      
      _showError('Failed to like post');
    }
  }
}
```

### File Upload with Progress

Handle file uploads with real-time progress:

```dart
class FileUploadService {
  final BondFire _bondFire;
  
  Stream<UploadProgress> uploadFile(File file, String endpoint) async* {
    final fileName = path.basename(file.path);
    final formData = FormData.fromMap({
      'file': await MultipartFile.fromFile(file.path, filename: fileName),
    });

    yield UploadProgress.started();

    try {
      final response = await _bondFire
          .post<FileUploadResponse>(endpoint)
          .body(formData)
          .onSendProgress((sent, total) {
            final progress = sent / total;
            // This will be emitted through the stream
          })
          .factory(FileUploadResponse.fromJson)
          .execute();

      yield UploadProgress.completed(response);
    } catch (e) {
      yield UploadProgress.failed(e.toString());
    }
  }
}

// Usage in controller
class ProfileController extends StateNotifier<ProfileState> {
  Future<void> uploadAvatar(File imageFile) async {
    await for (final progress in _fileUploadService.uploadFile(imageFile, '/upload/avatar')) {
      switch (progress.status) {
        case UploadStatus.started:
          state = state.copyWith(isUploading: true, uploadProgress: 0);
          break;
        case UploadStatus.progress:
          state = state.copyWith(uploadProgress: progress.percentage);
          break;
        case UploadStatus.completed:
          state = state.copyWith(
            isUploading: false,
            uploadProgress: 100,
            user: state.user.copyWith(avatarUrl: progress.result!.url),
          );
          break;
        case UploadStatus.failed:
          state = state.copyWith(
            isUploading: false,
            uploadProgress: 0,
            error: progress.error,
          );
          break;
      }
    }
  }
}
```

## UI Patterns

### Pull to Refresh

Implement pull-to-refresh with cache invalidation:

```dart
class RefreshableList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final controller = ref.watch(postsControllerProvider.notifier);
    final state = ref.watch(postsControllerProvider);
    
    return RefreshIndicator(
      onRefresh: () async {
        await controller.refresh();
      },
      child: ListView.builder(
        itemCount: state.posts.length,
        itemBuilder: (context, index) {
          return PostCard(post: state.posts[index]);
        },
      ),
    );
  }
}

// In controller
Future<void> refresh() async {
  // Clear cache
  await Cache.forget('posts_page_1');
  
  // Reload data
  await loadInitial();
}
```

### Search with Debouncing

Implement efficient search with debounced queries:

```dart
class SearchController extends StateNotifier<SearchState> {
  final SearchRepository _repository;
  Timer? _debounceTimer;
  
  SearchController(this._repository) : super(SearchState.initial());

  void search(String query) {
    _debounceTimer?.cancel();
    
    if (query.isEmpty) {
      state = state.copyWith(query: '', results: [], isLoading: false);
      return;
    }
    
    state = state.copyWith(query: query, isLoading: true);
    
    _debounceTimer = Timer(Duration(milliseconds: 500), () async {
      try {
        final results = await _repository.search(query);
        state = state.copyWith(
          results: results,
          isLoading: false,
          error: null,
        );
      } catch (e) {
        state = state.copyWith(
          error: e.toString(),
          isLoading: false,
        );
      }
    });
  }
}
```

### Offline Support

Handle offline scenarios gracefully:

```dart
class OfflineAwareRepository {
  final ApiService _apiService;
  final Cache _cache;
  final ConnectivityService _connectivity;
  
  Future<List<Post>> getPosts() async {
    final isOnline = await _connectivity.isConnected();
    
    if (isOnline) {
      try {
        final posts = await _apiService.getPosts();
        await _cache.put('posts', posts, Duration(hours: 24));
        return posts;
      } catch (e) {
        // Fallback to cache if network fails
        final cached = await _cache.get<List<Post>>('posts');
        if (cached != null) {
          return cached;
        }
        rethrow;
      }
    } else {
      // Offline - use cache only
      final cached = await _cache.get<List<Post>>('posts');
      if (cached != null) {
        return cached;
      }
      throw OfflineException('No cached data available');
    }
  }
}
```

## Advanced Patterns

### Deep Links to Features

Handle complex deep linking scenarios:

```dart
class DeepLinkHandler {
  static Future<void> handleNotificationTap(Map<String, dynamic> data) async {
    final type = data['type'] as String;
    
    switch (type) {
      case 'new_message':
        final chatId = data['chat_id'] as String;
        await AppRouter.pushChatDetail(chatId);
        break;
        
      case 'friend_request':
        final userId = data['user_id'] as String;
        await AppRouter.pushUserProfile(userId);
        break;
        
      case 'post_like':
        final postId = data['post_id'] as String;
        await AppRouter.pushPostDetail(postId);
        break;
        
      default:
        await AppRouter.pushHome();
    }
  }
}
```

### Force Update Dialog

Implement force update when app version is outdated:

```dart
class AppUpdateService {
  final RemoteConfigService _remoteConfig;
  
  Future<void> checkForUpdates() async {
    final minVersion = _remoteConfig.getString('min_app_version');
    final currentVersion = AppConfig.version;
    
    if (_isVersionOutdated(currentVersion, minVersion)) {
      await _showForceUpdateDialog();
    }
  }
  
  bool _isVersionOutdated(String current, String minimum) {
    final currentParts = current.split('.').map(int.parse).toList();
    final minimumParts = minimum.split('.').map(int.parse).toList();
    
    for (int i = 0; i < 3; i++) {
      if (currentParts[i] < minimumParts[i]) return true;
      if (currentParts[i] > minimumParts[i]) return false;
    }
    
    return false;
  }
  
  Future<void> _showForceUpdateDialog() async {
    await showDialog(
      context: AppRouter.navigator.context,
      barrierDismissible: false,
      builder: (context) => AlertDialog(
        title: Text(context.l10n.updateRequiredTitle),
        content: Text(context.l10n.updateRequiredMessage),
        actions: [
          ElevatedButton(
            onPressed: () => _openAppStore(),
            child: Text(context.l10n.updateButton),
          ),
        ],
      ),
    );
  }
}
```

### Feature Flags

Implement runtime feature toggles:

```dart
class FeatureFlagService {
  final RemoteConfigService _remoteConfig;
  final Cache _cache;
  
  Future<bool> isFeatureEnabled(String featureName) async {
    // Check cache first
    final cached = await _cache.get<bool>('feature_$featureName');
    if (cached != null) return cached;
    
    // Fetch from remote config
    final enabled = _remoteConfig.getBool('feature_$featureName');
    
    // Cache for offline access
    await _cache.put('feature_$featureName', enabled, Duration(hours: 1));
    
    return enabled;
  }
  
  Widget buildFeatureGate({
    required String featureName,
    required Widget child,
    Widget? fallback,
  }) {
    return FutureBuilder<bool>(
      future: isFeatureEnabled(featureName),
      builder: (context, snapshot) {
        if (snapshot.data == true) {
          return child;
        }
        return fallback ?? SizedBox.shrink();
      },
    );
  }
}

// Usage
FeatureFlagService().buildFeatureGate(
  featureName: 'social_sharing',
  child: ShareButton(post: post),
  fallback: SizedBox.shrink(),
)
```

## Testing Recipes

### Mock API Responses

```dart
class MockApiService extends UsersApiService {
  @override
  Future<ListResponse<User>> getUsers({int page = 1, int limit = 20}) async {
    await Future.delayed(Duration(milliseconds: 500)); // Simulate network delay
    
    return ListResponse<User>(
      data: List.generate(limit, (index) => 
        User(
          id: '${(page - 1) * limit + index + 1}',
          name: 'User ${(page - 1) * limit + index + 1}',
          email: 'user${(page - 1) * limit + index + 1}@example.com',
        ),
      ),
      meta: PaginationMeta(
        currentPage: page,
        totalPages: 10,
        total: 200,
      ),
    );
  }
}
```

### Golden Test Setup

```dart
void main() {
  group('Golden Tests', () {
    testWidgets('PostCard should match golden file', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          theme: AppTheme.lightTheme,
          home: Material(
            child: PostCard(post: testPost),
          ),
        ),
      );
      
      await expectLater(
        find.byType(PostCard),
        matchesGoldenFile('post_card_light.png'),
      );
    });
    
    testWidgets('PostCard dark theme should match golden file', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          theme: AppTheme.darkTheme,
          home: Material(
            child: PostCard(post: testPost),
          ),
        ),
      );
      
      await expectLater(
        find.byType(PostCard),
        matchesGoldenFile('post_card_dark.png'),
      );
    });
  });
}
```

## Performance Recipes

### Image Caching and Optimization

```dart
class OptimizedImageWidget extends StatelessWidget {
  final String imageUrl;
  final double? width;
  final double? height;
  
  const OptimizedImageWidget({
    Key? key,
    required this.imageUrl,
    this.width,
    this.height,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return CachedNetworkImage(
      imageUrl: _getOptimizedUrl(),
      width: width,
      height: height,
      fit: BoxFit.cover,
      placeholder: (context, url) => Container(
        width: width,
        height: height,
        color: context.colors.surfaceVariant,
        child: Center(
          child: CircularProgressIndicator(),
        ),
      ),
      errorWidget: (context, url, error) => Container(
        width: width,
        height: height,
        color: context.colors.errorContainer,
        child: Icon(
          Icons.error,
          color: context.colors.onErrorContainer,
        ),
      ),
    );
  }
  
  String _getOptimizedUrl() {
    final uri = Uri.parse(imageUrl);
    final params = Map<String, String>.from(uri.queryParameters);
    
    // Add optimization parameters
    if (width != null) params['w'] = width!.round().toString();
    if (height != null) params['h'] = height!.round().toString();
    params['q'] = '80'; // Quality
    params['f'] = 'webp'; // Format
    
    return uri.replace(queryParameters: params).toString();
  }
}
```

### Memory Management

```dart
class MemoryEfficientListView extends StatefulWidget {
  final List<Post> posts;
  
  const MemoryEfficientListView({Key? key, required this.posts}) : super(key: key);

  @override
  _MemoryEfficientListViewState createState() => _MemoryEfficientListViewState();
}

class _MemoryEfficientListViewState extends State<MemoryEfficientListView> {
  final Map<int, Widget> _cachedWidgets = {};
  
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: widget.posts.length,
      itemBuilder: (context, index) {
        // Cache widgets to avoid rebuilding
        return _cachedWidgets[index] ??= PostCard(
          post: widget.posts[index],
          key: ValueKey(widget.posts[index].id),
        );
      },
    );
  }
  
  @override
  void didUpdateWidget(MemoryEfficientListView oldWidget) {
    super.didUpdateWidget(oldWidget);
    
    // Clear cache if posts changed
    if (widget.posts != oldWidget.posts) {
      _cachedWidgets.clear();
    }
  }
}
```

## Security Recipes

### Secure Token Storage

```dart
class SecureTokenStorage implements TokenStorage {
  static const _accessTokenKey = 'access_token';
  static const _refreshTokenKey = 'refresh_token';
  
  @override
  Future<String?> getAccessToken() async {
    try {
      return await FlutterSecureStorage().read(key: _accessTokenKey);
    } catch (e) {
      return null;
    }
  }
  
  @override
  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
  }) async {
    await Future.wait([
      FlutterSecureStorage().write(key: _accessTokenKey, value: accessToken),
      FlutterSecureStorage().write(key: _refreshTokenKey, value: refreshToken),
    ]);
  }
  
  @override
  Future<void> clearTokens() async {
    await Future.wait([
      FlutterSecureStorage().delete(key: _accessTokenKey),
      FlutterSecureStorage().delete(key: _refreshTokenKey),
    ]);
  }
}
```

### Input Sanitization

```dart
class InputSanitizer {
  static String sanitizeHtml(String input) {
    return input
        .replaceAll(RegExp(r'<[^>]*>'), '') // Remove HTML tags
        .replaceAll(RegExp(r'&[^;]+;'), '') // Remove HTML entities
        .trim();
  }
  
  static String sanitizeFileName(String input) {
    return input
        .replaceAll(RegExp(r'[<>:"/\\|?*]'), '_') // Replace invalid chars
        .replaceAll(RegExp(r'\s+'), '_') // Replace spaces
        .toLowerCase();
  }
  
  static String sanitizeUrl(String input) {
    try {
      final uri = Uri.parse(input);
      return uri.toString();
    } catch (e) {
      throw ArgumentError('Invalid URL: $input');
    }
  }
}
```

## Next Steps

Explore more advanced topics:

- [Advanced Patterns](../advanced-topics/index.md) - Complex architectural patterns
- [Performance Optimization](../advanced-topics/index.md) - Make your app faster
- [Security Best Practices](../advanced-topics/index.md) - Secure your application
- [Testing Strategies](../faq/index.md) - Comprehensive testing approaches

## Contributing Recipes

Have a useful pattern? Contribute it to the cookbook:

1. Fork the documentation repository
2. Add your recipe with complete code examples
3. Include tests and explanations
4. Submit a pull request

Good recipes are:
- Complete and runnable
- Well-documented with explanations
- Include error handling
- Provide test examples
- Follow Bond conventions
