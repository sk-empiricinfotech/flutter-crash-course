# ⚡ Flutter Crash Course — Recap Edition
### For developers who know the basics and want to revise fast.

---

## Table of Contents

1. [Flutter Overview](#1-flutter-overview)
2. [Project Structure](#2-project-structure)
3. [UI Building Essentials](#3-ui-building-essentials)
4. [Navigation & Routing](#4-navigation--routing)
5. [State Management](#5-state-management)
6. [Forms & User Input](#6-forms--user-input)
7. [Networking & APIs](#7-networking--apis)
8. [Async Programming in Flutter](#8-async-programming-in-flutter)
9. [Lists & Performance](#9-lists--performance)
10. [Theming & Styling](#10-theming--styling)
11. [Packages & Plugins](#11-packages--plugins)
12. [Debugging & Performance Tips](#12-debugging--performance-tips)
13. [Architecture Basics](#13-architecture-basics)
14. [Mini Project — News Reader App](#14-mini-project--news-reader-app)

---

## 1. Flutter Overview

### How Flutter Works

Flutter renders its own UI using the **Skia / Impeller** graphics engine — it doesn't use native OS widgets. Everything on screen is a **widget**, and Flutter rebuilds only what changed.

```
Your Dart Code
     ↓
Widget Tree (what you define)
     ↓
Element Tree (Flutter's internal representation)
     ↓
Render Tree (layout + painting)
     ↓
Skia / Impeller (GPU rendering)
```

The `build()` method describes what the UI looks like at any point in time. Flutter calls it whenever the state changes. Keep it **pure** — no side effects, no heavy logic inside `build()`.

---

### Stateless vs Stateful Widgets

```dart
// StatelessWidget — no internal state, rebuild only from outside
class GreetingCard extends StatelessWidget {
  final String name;
  const GreetingCard({super.key, required this.name});

  @override
  Widget build(BuildContext context) {
    return Text('Hello, $name!');
  }
}

// StatefulWidget — manages its own state internally
class Counter extends StatefulWidget {
  const Counter({super.key});
  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: () => setState(() => _count++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

| | StatelessWidget | StatefulWidget |
|---|---|---|
| Has internal state | ❌ | ✅ |
| Rebuilt by | Parent widget | Parent or `setState()` |
| Use when | Display only | Interactive, changing UI |

> 💡 **Quick Tip:** Prefer `StatelessWidget` by default. Reach for `StatefulWidget` only when the widget truly owns mutable state.

> ⚠️ **Common Mistake:** Putting business logic or API calls directly in `build()`. Keep `build()` clean — it may be called many times per second.

---

## 2. Project Structure

### Standard Layout

```
my_flutter_app/
├── android/                  ← Android native project
├── ios/                      ← iOS native project
├── lib/
│   ├── main.dart             ← Entry point
│   ├── app.dart              ← MaterialApp / root widget
│   ├── core/
│   │   ├── constants/        ← Colors, strings, dimensions
│   │   ├── theme/            ← ThemeData
│   │   └── utils/            ← Helper functions
│   ├── features/
│   │   ├── home/
│   │   │   ├── screens/      ← UI screens
│   │   │   ├── widgets/      ← Feature-specific widgets
│   │   │   └── controller/   ← State / logic
│   │   └── auth/
│   │       ├── screens/
│   │       └── controller/
│   ├── data/
│   │   ├── models/           ← Data classes
│   │   ├── repositories/     ← Data access layer
│   │   └── services/         ← API, database services
│   └── shared/
│       └── widgets/          ← Reusable widgets
├── test/                     ← Unit and widget tests
└── pubspec.yaml
```

---

### main.dart

```dart
import 'package:flutter/material.dart';
import 'app.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized(); // Required before async setup
  runApp(const MyApp());
}
```

### app.dart

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My App',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.light,
      darkTheme: AppTheme.dark,
      initialRoute: '/',
      routes: AppRoutes.routes,
    );
  }
}
```

---

### pubspec.yaml Essentials

```yaml
name: my_flutter_app
description: A Flutter application.
version: 1.0.0+1

environment:
  sdk: ^3.0.0

dependencies:
  flutter:
    sdk: flutter
  http: ^1.2.0           # HTTP requests
  provider: ^6.1.1       # State management
  go_router: ^13.0.0     # Navigation
  cached_network_image: ^3.3.1  # Image caching
  shared_preferences: ^2.2.2    # Local storage

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0

flutter:
  uses-material-design: true
  assets:
    - assets/images/
    - assets/icons/
  fonts:
    - family: Poppins
      fonts:
        - asset: assets/fonts/Poppins-Regular.ttf
        - asset: assets/fonts/Poppins-Bold.ttf
          weight: 700
```

> 💡 **Quick Tip:** After editing `pubspec.yaml`, always run `flutter pub get`. Indentation is critical — YAML is whitespace-sensitive.

---

## 3. UI Building Essentials

### Core Layout Widgets

```dart
// Container — box model for a single child
Container(
  width: 200,
  height: 100,
  margin: const EdgeInsets.all(16),
  padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
  decoration: BoxDecoration(
    color: Colors.blue.shade50,
    borderRadius: BorderRadius.circular(12),
    boxShadow: [
      BoxShadow(color: Colors.black12, blurRadius: 8, offset: Offset(0, 4)),
    ],
  ),
  child: const Text('Hello'),
),

// Row — horizontal layout
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween, // horizontal axis
  crossAxisAlignment: CrossAxisAlignment.center,      // vertical axis
  children: [
    const Icon(Icons.star),
    const Text('Rating'),
    const Icon(Icons.arrow_forward_ios, size: 16),
  ],
),

// Column — vertical layout
Column(
  mainAxisAlignment: MainAxisAlignment.center,    // vertical axis
  crossAxisAlignment: CrossAxisAlignment.start,   // horizontal axis
  children: [
    const Text('Title', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
    const SizedBox(height: 8),
    const Text('Subtitle', style: TextStyle(color: Colors.grey)),
  ],
),

// Stack — overlapping widgets (like CSS position: absolute)
Stack(
  alignment: Alignment.bottomRight,
  children: [
    Image.network('https://example.com/photo.jpg'),
    Container(
      padding: const EdgeInsets.all(8),
      color: Colors.black54,
      child: const Text('Caption', style: TextStyle(color: Colors.white)),
    ),
  ],
),
```

---

### Padding, Margin, Expanded, Flexible

```dart
// Padding — adds space inside the widget
Padding(
  padding: const EdgeInsets.symmetric(horizontal: 16),
  child: Text('Padded text'),
),

// Margin — use Container's margin property
Container(margin: const EdgeInsets.only(top: 24), child: Text('Has margin')),

// Expanded — fills remaining space, forces the child to fill
Row(
  children: [
    const Icon(Icons.search),
    Expanded(                          // Takes all remaining width
      child: TextField(
        decoration: InputDecoration(hintText: 'Search...'),
      ),
    ),
    const Icon(Icons.mic),
  ],
),

// Flexible — takes available space but child can be smaller
Row(
  children: [
    Flexible(flex: 2, child: Container(color: Colors.red, height: 50)),
    Flexible(flex: 1, child: Container(color: Colors.blue, height: 50)),
  ],
),
```

| Widget | Behaviour |
|---|---|
| `Expanded` | Forces child to fill available space |
| `Flexible` | Gives child available space; child can be smaller |
| `Spacer` | Empty flexible space between widgets |
| `SizedBox` | Fixed size box or spacing |

---

### Layout Constraints — The Golden Rule

> In Flutter: **constraints go down, sizes go up, parent decides position.**

```dart
// ConstrainedBox — impose min/max constraints
ConstrainedBox(
  constraints: const BoxConstraints(minHeight: 100, maxWidth: 300),
  child: Container(color: Colors.amber),
),

// SizedBox.expand — fills parent completely
SizedBox.expand(
  child: Container(color: Colors.teal),
),

// FractionallySizedBox — percentage of parent
FractionallySizedBox(
  widthFactor: 0.8,   // 80% of parent width
  child: ElevatedButton(onPressed: () {}, child: Text('Wide Button')),
),
```

> ⚠️ **Common Mistake:** Wrapping a `Column` inside a `SingleChildScrollView` inside another `Column` without setting `shrinkWrap: true` or proper constraints — causes "unbounded height" errors.

---

## 4. Navigation & Routing

### Navigator — Push & Pop

```dart
// Push a new screen
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const DetailScreen(id: 42)),
);

// Push and remove all previous routes (e.g., after login)
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (_) => const HomeScreen()),
  (route) => false,
);

// Pop back
Navigator.pop(context);

// Pop with a return value
Navigator.pop(context, 'result_data');

// Await the result from a pushed screen
final result = await Navigator.push(context, ...);
print(result); // 'result_data'
```

---

### Named Routes

```dart
// In MaterialApp
MaterialApp(
  initialRoute: '/',
  routes: {
    '/':          (context) => const HomeScreen(),
    '/detail':    (context) => const DetailScreen(),
    '/settings':  (context) => const SettingsScreen(),
  },
);

// Navigate
Navigator.pushNamed(context, '/detail');

// Navigate with arguments
Navigator.pushNamed(context, '/detail', arguments: {'id': 5, 'title': 'Post'});

// Receive arguments in destination screen
final args = ModalRoute.of(context)!.settings.arguments as Map<String, dynamic>;
print(args['id']); // 5
```

---

### GoRouter (Recommended for Production)

```dart
// pubspec.yaml: go_router: ^13.0.0

final router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: (context, state) => const HomeScreen()),
    GoRoute(
      path: '/post/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return PostDetailScreen(id: id);
      },
    ),
  ],
);

// In MaterialApp
MaterialApp.router(routerConfig: router);

// Navigate
context.go('/post/42');
context.push('/settings');
context.pop();
```

> 💡 **Quick Tip:** Use `GoRouter` for apps with deep links, web support, or nested navigation. Named routes are fine for simple apps.

---

### Passing Data Between Screens

```dart
// Option 1 — Constructor (recommended for simple data)
Navigator.push(context, MaterialPageRoute(
  builder: (_) => ProductScreen(product: selectedProduct),
));

// Option 2 — Arguments with GoRouter
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = int.parse(state.pathParameters['id']!);
    return ProductScreen(id: id);
  },
)

// Option 3 — State management (for complex/shared data)
// Store selected product in Provider/Riverpod and read it on next screen
```

---

## 5. State Management

### setState — For Simple Local State

```dart
class LikeButton extends StatefulWidget {
  const LikeButton({super.key});
  @override
  State<LikeButton> createState() => _LikeButtonState();
}

class _LikeButtonState extends State<LikeButton> {
  bool _isLiked = false;
  int _count = 120;

  void _toggle() {
    setState(() {
      _isLiked = !_isLiked;
      _count += _isLiked ? 1 : -1;
    });
  }

  @override
  Widget build(BuildContext context) {
    return IconButton(
      icon: Icon(_isLiked ? Icons.favorite : Icons.favorite_border),
      color: _isLiked ? Colors.red : Colors.grey,
      onPressed: _toggle,
    );
  }
}
```

> ⚠️ **Common Mistake:** Calling `setState()` after the widget is disposed. Always check `if (mounted) setState(...)` when calling after an async gap.

---

### Provider — Simple, Scalable

```dart
// 1. Add dependency: provider: ^6.1.1

// 2. Create a ChangeNotifier
class CartProvider extends ChangeNotifier {
  final List<String> _items = [];
  List<String> get items => _items;
  int get count => _items.length;

  void addItem(String item) {
    _items.add(item);
    notifyListeners(); // Tells all listeners to rebuild
  }

  void removeItem(String item) {
    _items.remove(item);
    notifyListeners();
  }
}

// 3. Provide it at the top of the tree
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => CartProvider(),
      child: const MyApp(),
    ),
  );
}

// 4a. Read (no rebuild needed)
final cart = context.read<CartProvider>();
cart.addItem('Apple');

// 4b. Watch (rebuilds when notified)
final count = context.watch<CartProvider>().count;

// 4c. Select (rebuilds only when selected value changes)
final itemCount = context.select<CartProvider, int>((c) => c.count);
```

---

### Riverpod — Modern, Testable

```dart
// 1. Add: flutter_riverpod: ^2.5.0
// 2. Wrap app in ProviderScope

void main() {
  runApp(const ProviderScope(child: MyApp()));
}

// 3. Define a provider
final counterProvider = StateNotifierProvider<CounterNotifier, int>((ref) {
  return CounterNotifier();
});

class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);
  void increment() => state++;
  void decrement() => state--;
}

// 4. Use in widget — extend ConsumerWidget instead of StatelessWidget
class CounterScreen extends ConsumerWidget {
  const CounterScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Column(
      children: [
        Text('$count'),
        ElevatedButton(
          onPressed: () => ref.read(counterProvider.notifier).increment(),
          child: const Text('+'),
        ),
      ],
    );
  }
}
```

---

### Bloc — For Complex, Event-Driven State

```dart
// Events
abstract class AuthEvent {}
class LoginRequested extends AuthEvent {
  final String email, password;
  LoginRequested(this.email, this.password);
}

// States
abstract class AuthState {}
class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}
class AuthSuccess extends AuthState { final String token; AuthSuccess(this.token); }
class AuthFailure extends AuthState { final String error; AuthFailure(this.error); }

// Bloc
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(AuthInitial()) {
    on<LoginRequested>((event, emit) async {
      emit(AuthLoading());
      try {
        final token = await authRepo.login(event.email, event.password);
        emit(AuthSuccess(token));
      } catch (e) {
        emit(AuthFailure(e.toString()));
      }
    });
  }
}

// UI
BlocBuilder<AuthBloc, AuthState>(
  builder: (context, state) {
    if (state is AuthLoading) return const CircularProgressIndicator();
    if (state is AuthSuccess) return Text('Welcome!');
    if (state is AuthFailure) return Text('Error: ${state.error}');
    return LoginForm();
  },
),
```

---

### When to Use What

| Solution | Best For | Complexity |
|---|---|---|
| `setState` | Local, single-widget state | ⭐ Simple |
| `Provider` | App-wide state, beginner-friendly | ⭐⭐ Medium |
| `Riverpod` | Scalable apps, testability, modern | ⭐⭐⭐ Medium-High |
| `Bloc` | Complex flows, strict separation, teams | ⭐⭐⭐⭐ High |

---

## 6. Forms & User Input

### TextField Basics

```dart
class SearchBar extends StatefulWidget {
  const SearchBar({super.key});
  @override
  State<SearchBar> createState() => _SearchBarState();
}

class _SearchBarState extends State<SearchBar> {
  final TextEditingController _controller = TextEditingController();

  @override
  void dispose() {
    _controller.dispose(); // Always dispose controllers!
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: _controller,
      keyboardType: TextInputType.text,
      textInputAction: TextInputAction.search,
      decoration: InputDecoration(
        hintText: 'Search...',
        prefixIcon: const Icon(Icons.search),
        suffixIcon: IconButton(
          icon: const Icon(Icons.clear),
          onPressed: () => _controller.clear(),
        ),
        border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
        filled: true,
        fillColor: Colors.grey.shade100,
      ),
      onSubmitted: (value) => print('Search: $value'),
      onChanged: (value) => print('Typing: $value'),
    );
  }
}
```

---

### Form with Validation

```dart
class LoginForm extends StatefulWidget {
  const LoginForm({super.key});
  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _obscurePassword = true;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  void _submit() {
    if (_formKey.currentState!.validate()) {
      // Form is valid — proceed
      print('Email: ${_emailController.text}');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            controller: _emailController,
            keyboardType: TextInputType.emailAddress,
            decoration: const InputDecoration(
              labelText: 'Email',
              prefixIcon: Icon(Icons.email_outlined),
            ),
            validator: (value) {
              if (value == null || value.isEmpty) return 'Email is required';
              if (!value.contains('@')) return 'Enter a valid email';
              return null; // null means valid
            },
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _passwordController,
            obscureText: _obscurePassword,
            decoration: InputDecoration(
              labelText: 'Password',
              prefixIcon: const Icon(Icons.lock_outline),
              suffixIcon: IconButton(
                icon: Icon(_obscurePassword ? Icons.visibility : Icons.visibility_off),
                onPressed: () => setState(() => _obscurePassword = !_obscurePassword),
              ),
            ),
            validator: (value) {
              if (value == null || value.length < 6) return 'Min 6 characters';
              return null;
            },
          ),
          const SizedBox(height: 24),
          SizedBox(
            width: double.infinity,
            child: ElevatedButton(
              onPressed: _submit,
              child: const Text('Login'),
            ),
          ),
        ],
      ),
    );
  }
}
```

> ⚠️ **Common Mistakes:**
> - Forgetting `dispose()` on `TextEditingController` — causes memory leaks.
> - Using `TextEditingController` when you only need `onChanged` — use `onChanged` directly for simple cases.

---

## 7. Networking & APIs

### HTTP Requests

```dart
// pubspec.yaml: http: ^1.2.0
import 'package:http/http.dart' as http;
import 'dart:convert';

class ApiService {
  static const String _baseUrl = 'https://jsonplaceholder.typicode.com';

  // GET request
  Future<List<Post>> fetchPosts() async {
    final response = await http.get(
      Uri.parse('$_baseUrl/posts'),
      headers: {'Content-Type': 'application/json'},
    );

    if (response.statusCode == 200) {
      final List<dynamic> data = jsonDecode(response.body);
      return data.map((json) => Post.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load posts: ${response.statusCode}');
    }
  }

  // POST request
  Future<Post> createPost({required String title, required String body}) async {
    final response = await http.post(
      Uri.parse('$_baseUrl/posts'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({'title': title, 'body': body, 'userId': 1}),
    );

    if (response.statusCode == 201) {
      return Post.fromJson(jsonDecode(response.body));
    } else {
      throw Exception('Failed to create post');
    }
  }
}
```

---

### JSON Parsing (Model Class)

```dart
class Post {
  final int id;
  final int userId;
  final String title;
  final String body;

  const Post({
    required this.id,
    required this.userId,
    required this.title,
    required this.body,
  });

  // From JSON — for parsing API response
  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id:     json['id']     as int,
      userId: json['userId'] as int,
      title:  json['title']  as String,
      body:   json['body']   as String,
    );
  }

  // To JSON — for sending data
  Map<String, dynamic> toJson() {
    return {'id': id, 'userId': userId, 'title': title, 'body': body};
  }

  @override
  String toString() => 'Post(id: $id, title: $title)';
}
```

> 💡 **Quick Tip:** For large projects, use `json_serializable` or `freezed` packages to auto-generate `fromJson` / `toJson` code. Avoids manual errors and saves time.

---

### Error Handling for APIs

```dart
// Custom exceptions
class ApiException implements Exception {
  final int statusCode;
  final String message;
  ApiException(this.statusCode, this.message);
  @override
  String toString() => 'ApiException [$statusCode]: $message';
}

class NetworkException implements Exception {
  final String message;
  NetworkException(this.message);
}

// Repository with proper error handling
class PostRepository {
  final ApiService _api;
  PostRepository(this._api);

  Future<List<Post>> getPosts() async {
    try {
      return await _api.fetchPosts();
    } on SocketException {
      throw NetworkException('No internet connection.');
    } on TimeoutException {
      throw NetworkException('Request timed out.');
    } on ApiException {
      rethrow;
    } catch (e) {
      throw Exception('Unexpected error: $e');
    }
  }
}
```

---

## 8. Async Programming in Flutter

### FutureBuilder

Use when you have a **one-time** async operation (fetch data once and display).

```dart
class PostListScreen extends StatelessWidget {
  final PostRepository _repo = PostRepository(ApiService());

  PostListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Posts')),
      body: FutureBuilder<List<Post>>(
        future: _repo.getPosts(),
        builder: (context, snapshot) {
          // Still loading
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          // Error
          if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.error_outline, size: 48, color: Colors.red),
                  const SizedBox(height: 8),
                  Text('Error: ${snapshot.error}'),
                  ElevatedButton(
                    onPressed: () => (context as Element).markNeedsBuild(),
                    child: const Text('Retry'),
                  ),
                ],
              ),
            );
          }
          // Data available
          final posts = snapshot.data!;
          return ListView.builder(
            itemCount: posts.length,
            itemBuilder: (context, index) => ListTile(
              title: Text(posts[index].title),
              subtitle: Text('Post #${posts[index].id}'),
            ),
          );
        },
      ),
    );
  }
}
```

> ⚠️ **Common Mistake:** Calling the async function directly inside `FutureBuilder`'s `future` parameter within `build()`. This creates a new Future on every rebuild.

```dart
// ❌ Wrong — creates new Future every rebuild
FutureBuilder(future: fetchPosts(), ...)

// ✅ Correct — store Future in initState or a variable
late final Future<List<Post>> _postsFuture;

@override
void initState() {
  super.initState();
  _postsFuture = _repo.getPosts();
}

// Then use:
FutureBuilder(future: _postsFuture, ...)
```

---

### StreamBuilder

Use when you have **continuous / real-time data** (chat, timers, live feeds, Firebase).

```dart
// A simple stream example — ticking timer
Stream<int> timerStream() async* {
  int seconds = 0;
  while (true) {
    await Future.delayed(const Duration(seconds: 1));
    yield seconds++;
  }
}

class TimerWidget extends StatelessWidget {
  const TimerWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<int>(
      stream: timerStream(),
      initialData: 0,
      builder: (context, snapshot) {
        if (snapshot.hasError) return Text('Error: ${snapshot.error}');
        return Text(
          '${snapshot.data} seconds',
          style: const TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
        );
      },
    );
  }
}

// Real-world: Firebase Firestore stream
StreamBuilder<QuerySnapshot>(
  stream: FirebaseFirestore.instance.collection('messages').snapshots(),
  builder: (context, snapshot) {
    if (!snapshot.hasData) return const CircularProgressIndicator();
    final docs = snapshot.data!.docs;
    return ListView.builder(
      itemCount: docs.length,
      itemBuilder: (_, i) => ListTile(title: Text(docs[i]['text'])),
    );
  },
),
```

| | `FutureBuilder` | `StreamBuilder` |
|---|---|---|
| Data arrives | Once | Multiple times |
| Use for | HTTP fetch, one-time op | Firebase, WebSocket, timer |
| State resets | On rebuild | Persistent stream |

---

## 9. Lists & Performance

### ListView Variants

```dart
// ListView — for short, known lists
ListView(
  children: items.map((item) => ListTile(title: Text(item))).toList(),
),

// ListView.builder — for long/dynamic lists (lazy — recommended)
ListView.builder(
  itemCount: posts.length,
  itemBuilder: (context, index) {
    final post = posts[index];
    return PostCard(post: post);
  },
),

// ListView.separated — with dividers
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (_, __) => const Divider(height: 1),
  itemBuilder: (context, index) => ListTile(title: Text(items[index])),
),

// ListView.custom — full control with slivers
```

---

### GridView

```dart
// GridView.builder — lazy, for large grids (recommended)
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,          // 2 columns
    crossAxisSpacing: 12,
    mainAxisSpacing: 12,
    childAspectRatio: 3 / 4,    // width:height ratio
  ),
  itemCount: products.length,
  itemBuilder: (context, index) => ProductCard(product: products[index]),
),

// GridView.extent — auto-calculates columns based on max item width
GridView.extent(
  maxCrossAxisExtent: 180,  // Each item max 180px wide
  children: items.map((i) => ItemCard(item: i)).toList(),
),
```

---

### Performance Tips

```dart
// 1. Use const constructors wherever possible
const Text('Static Label')      // ✅ Not rebuilt unnecessarily
Text('Static Label')            // ❌ Rebuilt every time parent rebuilds

// 2. Extract widgets into separate classes (not just methods)
// ❌ Method — always rebuilt with parent
Widget _buildHeader() => const Text('Header');

// ✅ Class — can be const, independently managed
class AppHeader extends StatelessWidget {
  const AppHeader({super.key});
  @override
  Widget build(BuildContext context) => const Text('Header');
}

// 3. Use keys to preserve state during list reorders
ListView.builder(
  itemBuilder: (_, i) => PostCard(
    key: ValueKey(posts[i].id), // Helps Flutter identify items
    post: posts[i],
  ),
),

// 4. Cache network images
// pubspec.yaml: cached_network_image: ^3.3.1
CachedNetworkImage(
  imageUrl: 'https://example.com/photo.jpg',
  placeholder: (_, __) => const CircularProgressIndicator(),
  errorWidget: (_, __, ___) => const Icon(Icons.broken_image),
),

// 5. Avoid rebuilding entire lists — use AnimatedList or specific state solutions
```

> 💡 **Quick Tips:**
> - Use `ListView.builder` instead of `ListView` for any list longer than ~10 items.
> - Never use `shrinkWrap: true` inside a `ListView` inside a `ScrollView` — it defeats lazy loading. Use `Slivers` instead.
> - Use `RepaintBoundary` to isolate frequently-updating widgets (animations, timers) from the rest of the tree.

---

## 10. Theming & Styling

### ThemeData Setup

```dart
// lib/core/theme/app_theme.dart
import 'package:flutter/material.dart';

class AppTheme {
  static const Color _primaryColor = Color(0xFF6C63FF);
  static const Color _secondaryColor = Color(0xFF03DAC6);

  static ThemeData get light => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: _primaryColor,
      secondary: _secondaryColor,
      brightness: Brightness.light,
    ),
    textTheme: const TextTheme(
      headlineLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
      headlineMedium: TextStyle(fontSize: 24, fontWeight: FontWeight.w600),
      bodyLarge: TextStyle(fontSize: 16),
      bodyMedium: TextStyle(fontSize: 14),
    ),
    appBarTheme: const AppBarTheme(
      centerTitle: true,
      elevation: 0,
      scrolledUnderElevation: 1,
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        minimumSize: const Size(double.infinity, 52),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
      ),
    ),
    inputDecorationTheme: InputDecorationTheme(
      border: OutlineInputBorder(borderRadius: BorderRadius.circular(10)),
      contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
    ),
  );

  static ThemeData get dark => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: _primaryColor,
      brightness: Brightness.dark,
    ),
  );
}
```

---

### Using Theme in Widgets

```dart
// Read theme values — no hardcoded colors in widgets!
@override
Widget build(BuildContext context) {
  final theme = Theme.of(context);
  final colors = theme.colorScheme;
  final textTheme = theme.textTheme;

  return Container(
    color: colors.surface,
    child: Text(
      'Hello',
      style: textTheme.headlineMedium?.copyWith(color: colors.primary),
    ),
  );
}
```

---

### Dark Mode Toggle

```dart
// Using ValueNotifier for simple toggle (or use Provider/Riverpod)
final themeNotifier = ValueNotifier<ThemeMode>(ThemeMode.system);

// In MaterialApp
ValueListenableBuilder<ThemeMode>(
  valueListenable: themeNotifier,
  builder: (context, mode, _) {
    return MaterialApp(
      themeMode: mode,
      theme: AppTheme.light,
      darkTheme: AppTheme.dark,
      home: const HomeScreen(),
    );
  },
),

// Toggle from a settings screen
Switch(
  value: themeNotifier.value == ThemeMode.dark,
  onChanged: (isDark) {
    themeNotifier.value = isDark ? ThemeMode.dark : ThemeMode.light;
  },
),
```

> 💡 **Quick Tip:** Never hardcode colors like `Colors.blue` in widgets. Always read from `Theme.of(context).colorScheme`. This makes dark mode and rebranding effortless.

---

## 11. Packages & Plugins

### Finding & Adding Packages

```bash
# Search on pub.dev or via CLI
flutter pub search http

# Add a package
flutter pub add http

# Add a dev dependency
flutter pub add --dev flutter_lints

# Update all packages
flutter pub upgrade

# Show outdated packages
flutter pub outdated
```

---

### Essential Packages Reference

| Category | Package | Use Case |
|---|---|---|
| HTTP | `http` / `dio` | REST API calls |
| State | `provider`, `riverpod`, `flutter_bloc` | State management |
| Navigation | `go_router` | Declarative routing |
| Local DB | `hive`, `sqflite`, `isar` | Local storage |
| Firebase | `firebase_core`, `cloud_firestore` | Backend |
| Images | `cached_network_image` | Cached image loading |
| Serialization | `json_serializable`, `freezed` | JSON models |
| DI | `get_it` | Dependency injection |
| Env | `flutter_dotenv` | Environment variables |
| UI | `shimmer`, `lottie`, `flutter_svg` | Animations & assets |

---

### Using Dio (Better HTTP client)

```dart
// pubspec.yaml: dio: ^5.4.0
import 'package:dio/dio.dart';

class DioClient {
  late final Dio _dio;

  DioClient() {
    _dio = Dio(BaseOptions(
      baseUrl: 'https://api.example.com',
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 10),
      headers: {'Content-Type': 'application/json'},
    ));

    // Interceptor — e.g., attach auth token to every request
    _dio.interceptors.add(InterceptorsWrapper(
      onRequest: (options, handler) {
        options.headers['Authorization'] = 'Bearer $authToken';
        return handler.next(options);
      },
      onError: (error, handler) {
        if (error.response?.statusCode == 401) {
          // Handle token expiry — refresh or logout
        }
        return handler.next(error);
      },
    ));
  }

  Future<List<Post>> fetchPosts() async {
    final response = await _dio.get('/posts');
    return (response.data as List).map((j) => Post.fromJson(j)).toList();
  }
}
```

---

## 12. Debugging & Performance Tips

### Common Issues & Fixes

| Issue | Cause | Fix |
|---|---|---|
| "Unbounded height" | Column inside Column / ListView without constraints | Add `shrinkWrap`, use `Expanded`, or fix parent constraints |
| Jank / dropped frames | Heavy work in `build()` | Move logic to isolates or async functions |
| `setState()` after dispose | Async callback runs after widget removed | Check `if (mounted)` before `setState()` |
| Memory leak | Controllers not disposed | Always `dispose()` in `State.dispose()` |
| Infinite FutureBuilder loop | Future created inside `build()` | Store Future in `initState()` |
| White screen on startup | Error before `runApp` | Check `WidgetsFlutterBinding.ensureInitialized()` |

---

### Flutter DevTools

```bash
# Run with DevTools
flutter run --profile      # For performance profiling (not debug mode)
flutter run --release      # For production testing

# Open DevTools
flutter pub global activate devtools
flutter pub global run devtools
```

**Key DevTools Panels:**

- **Widget Inspector** — Visualise widget tree, check layouts, find overflow issues
- **Performance** — Find jank, track frame build times, identify expensive rebuilds
- **Memory** — Spot memory leaks, track object allocations
- **Network** — Monitor HTTP requests and responses
- **CPU Profiler** — Identify slow Dart code

---

### Useful Debug Utilities

```dart
// Print widget rebuild count (dev only)
import 'package:flutter/foundation.dart';

// Slow animations to 0.1x speed for debugging
timeDilation = 5.0; // In MaterialApp or a debug button

// Debug painting — show layout boundaries
import 'package:flutter/rendering.dart';
debugPaintSizeEnabled = true;
debugPaintLayerBordersEnabled = true;

// Log to console with formatting
debugPrint('Data: $data'); // Better than print() — respects log limits

// Assert — runs only in debug mode
assert(user != null, 'User must not be null at this point');
```

---

## 13. Architecture Basics

### Clean Architecture — Simplified

```
UI Layer          →  Screens, Widgets (only display and user events)
Logic Layer       →  Controllers, ViewModels, Blocs (state & business rules)
Data Layer        →  Repositories, Services, Models (data access & APIs)
```

```
lib/
├── features/
│   └── posts/
│       ├── screens/
│       │   └── posts_screen.dart        ← UI only
│       ├── widgets/
│       │   └── post_card.dart
│       ├── controller/
│       │   └── posts_controller.dart    ← Logic, holds state
│       └── data/
│           ├── post_model.dart          ← Data class
│           ├── post_repository.dart     ← Decides where data comes from
│           └── post_service.dart        ← Raw API calls
```

---

### Practical Example

```dart
// ─── Model ────────────────────────────────────────────────────────
class Post {
  final int id;
  final String title;
  final String body;
  const Post({required this.id, required this.title, required this.body});
  factory Post.fromJson(Map<String, dynamic> j) =>
      Post(id: j['id'], title: j['title'], body: j['body']);
}

// ─── Service (raw API) ────────────────────────────────────────────
class PostService {
  final http.Client _client;
  PostService(this._client);

  Future<List<Map<String, dynamic>>> fetchRawPosts() async {
    final res = await _client.get(Uri.parse('https://jsonplaceholder.typicode.com/posts'));
    if (res.statusCode != 200) throw Exception('HTTP ${res.statusCode}');
    return List<Map<String, dynamic>>.from(jsonDecode(res.body));
  }
}

// ─── Repository (data decision) ───────────────────────────────────
class PostRepository {
  final PostService _service;
  PostRepository(this._service);

  Future<List<Post>> getPosts() async {
    final raw = await _service.fetchRawPosts();
    return raw.map(Post.fromJson).toList();
  }
}

// ─── Controller (state logic) ─────────────────────────────────────
class PostsController extends ChangeNotifier {
  final PostRepository _repo;
  PostsController(this._repo);

  List<Post> posts = [];
  bool isLoading = false;
  String? error;

  Future<void> loadPosts() async {
    isLoading = true;
    error = null;
    notifyListeners();

    try {
      posts = await _repo.getPosts();
    } catch (e) {
      error = e.toString();
    } finally {
      isLoading = false;
      notifyListeners();
    }
  }
}

// ─── Screen (UI only) ─────────────────────────────────────────────
class PostsScreen extends StatelessWidget {
  const PostsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final ctrl = context.watch<PostsController>();

    if (ctrl.isLoading) return const Center(child: CircularProgressIndicator());
    if (ctrl.error != null) return Center(child: Text(ctrl.error!));

    return ListView.builder(
      itemCount: ctrl.posts.length,
      itemBuilder: (_, i) => ListTile(title: Text(ctrl.posts[i].title)),
    );
  }
}
```

> 💡 **Architecture Golden Rules:**
> - UI knows about Controller — not about Repository or Service.
> - Repository knows about Service — not about UI.
> - Models know about nothing — they are pure data.
> - Dependency flows one way — always downward.

---

## 14. Mini Project — News Reader App

A complete mini-app combining: API fetch · list display · navigation · state management · error handling.

---

### Project Structure

```
lib/
├── main.dart
├── app.dart
├── features/
│   └── news/
│       ├── models/article_model.dart
│       ├── services/news_service.dart
│       ├── repositories/news_repository.dart
│       ├── controller/news_controller.dart
│       └── screens/
│           ├── news_list_screen.dart
│           └── article_detail_screen.dart
└── shared/widgets/
    ├── error_view.dart
    └── loading_view.dart
```

---

### Article Model

```dart
// lib/features/news/models/article_model.dart
class Article {
  final String id;
  final String title;
  final String description;
  final String imageUrl;
  final String source;
  final String publishedAt;
  final String url;

  const Article({
    required this.id,
    required this.title,
    required this.description,
    required this.imageUrl,
    required this.source,
    required this.publishedAt,
    required this.url,
  });

  factory Article.fromJson(Map<String, dynamic> json) {
    return Article(
      id:          json['id']?.toString() ?? '',
      title:       json['title'] ?? 'No title',
      description: json['description'] ?? '',
      imageUrl:    json['urlToImage'] ?? '',
      source:      json['source']?['name'] ?? 'Unknown',
      publishedAt: json['publishedAt'] ?? '',
      url:         json['url'] ?? '',
    );
  }
}
```

---

### News Service & Repository

```dart
// lib/features/news/services/news_service.dart
import 'package:http/http.dart' as http;
import 'dart:convert';

class NewsService {
  // Using a free mock API (replace with real NewsAPI key)
  static const String _baseUrl = 'https://newsapi.org/v2';
  static const String _apiKey  = 'YOUR_API_KEY';

  Future<List<Map<String, dynamic>>> fetchTopHeadlines() async {
    final uri = Uri.parse('$_baseUrl/top-headlines?country=us&apiKey=$_apiKey');
    final response = await http.get(uri).timeout(const Duration(seconds: 10));

    if (response.statusCode == 200) {
      final data = jsonDecode(response.body);
      return List<Map<String, dynamic>>.from(data['articles']);
    } else {
      throw Exception('API Error: ${response.statusCode}');
    }
  }
}

// lib/features/news/repositories/news_repository.dart
class NewsRepository {
  final NewsService _service;
  NewsRepository(this._service);

  Future<List<Article>> getTopHeadlines() async {
    final raw = await _service.fetchTopHeadlines();
    return raw.map(Article.fromJson).toList();
  }
}
```

---

### News Controller

```dart
// lib/features/news/controller/news_controller.dart
import 'package:flutter/material.dart';

class NewsController extends ChangeNotifier {
  final NewsRepository _repo;
  NewsController(this._repo);

  List<Article> articles = [];
  bool isLoading = false;
  String? errorMessage;

  Future<void> loadNews() async {
    isLoading = true;
    errorMessage = null;
    notifyListeners();

    try {
      articles = await _repo.getTopHeadlines();
    } on SocketException {
      errorMessage = 'No internet connection.';
    } catch (e) {
      errorMessage = 'Something went wrong. Please try again.';
    } finally {
      isLoading = false;
      notifyListeners();
    }
  }

  void refresh() => loadNews();
}
```

---

### News List Screen

```dart
// lib/features/news/screens/news_list_screen.dart
class NewsListScreen extends StatefulWidget {
  const NewsListScreen({super.key});
  @override
  State<NewsListScreen> createState() => _NewsListScreenState();
}

class _NewsListScreenState extends State<NewsListScreen> {
  @override
  void initState() {
    super.initState();
    // Load news once when screen opens
    WidgetsBinding.instance.addPostFrameCallback((_) {
      context.read<NewsController>().loadNews();
    });
  }

  @override
  Widget build(BuildContext context) {
    final ctrl = context.watch<NewsController>();

    return Scaffold(
      appBar: AppBar(
        title: const Text('Top Headlines'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: ctrl.refresh,
          ),
        ],
      ),
      body: _buildBody(ctrl),
    );
  }

  Widget _buildBody(NewsController ctrl) {
    if (ctrl.isLoading) {
      return const Center(child: CircularProgressIndicator());
    }
    if (ctrl.errorMessage != null) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.wifi_off, size: 64, color: Colors.grey),
            const SizedBox(height: 16),
            Text(ctrl.errorMessage!, style: const TextStyle(fontSize: 16)),
            const SizedBox(height: 16),
            ElevatedButton.icon(
              onPressed: ctrl.refresh,
              icon: const Icon(Icons.refresh),
              label: const Text('Retry'),
            ),
          ],
        ),
      );
    }
    if (ctrl.articles.isEmpty) {
      return const Center(child: Text('No articles found.'));
    }
    return RefreshIndicator(
      onRefresh: ctrl.loadNews,
      child: ListView.builder(
        padding: const EdgeInsets.all(12),
        itemCount: ctrl.articles.length,
        itemBuilder: (context, index) {
          final article = ctrl.articles[index];
          return ArticleCard(
            article: article,
            onTap: () => Navigator.push(
              context,
              MaterialPageRoute(
                builder: (_) => ArticleDetailScreen(article: article),
              ),
            ),
          );
        },
      ),
    );
  }
}

// Article Card Widget
class ArticleCard extends StatelessWidget {
  final Article article;
  final VoidCallback onTap;

  const ArticleCard({super.key, required this.article, required this.onTap});

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);

    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      clipBehavior: Clip.antiAlias,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
      child: InkWell(
        onTap: onTap,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            if (article.imageUrl.isNotEmpty)
              CachedNetworkImage(
                imageUrl: article.imageUrl,
                height: 180,
                width: double.infinity,
                fit: BoxFit.cover,
                placeholder: (_, __) => Container(
                  height: 180,
                  color: theme.colorScheme.surfaceVariant,
                  child: const Center(child: CircularProgressIndicator()),
                ),
                errorWidget: (_, __, ___) => Container(
                  height: 180,
                  color: theme.colorScheme.surfaceVariant,
                  child: const Icon(Icons.broken_image, size: 48),
                ),
              ),
            Padding(
              padding: const EdgeInsets.all(12),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(
                    children: [
                      Chip(
                        label: Text(article.source,
                            style: const TextStyle(fontSize: 11)),
                        padding: EdgeInsets.zero,
                      ),
                      const Spacer(),
                      Text(
                        _formatDate(article.publishedAt),
                        style: theme.textTheme.bodySmall,
                      ),
                    ],
                  ),
                  const SizedBox(height: 8),
                  Text(
                    article.title,
                    style: theme.textTheme.titleMedium
                        ?.copyWith(fontWeight: FontWeight.bold),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                  if (article.description.isNotEmpty) ...[
                    const SizedBox(height: 6),
                    Text(
                      article.description,
                      style: theme.textTheme.bodyMedium
                          ?.copyWith(color: Colors.grey.shade600),
                      maxLines: 2,
                      overflow: TextOverflow.ellipsis,
                    ),
                  ],
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  String _formatDate(String iso) {
    try {
      final date = DateTime.parse(iso);
      return '${date.day}/${date.month}/${date.year}';
    } catch (_) {
      return '';
    }
  }
}
```

---

### Article Detail Screen

```dart
// lib/features/news/screens/article_detail_screen.dart
class ArticleDetailScreen extends StatelessWidget {
  final Article article;
  const ArticleDetailScreen({super.key, required this.article});

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);

    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            expandedHeight: 250,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              background: article.imageUrl.isNotEmpty
                  ? CachedNetworkImage(
                      imageUrl: article.imageUrl,
                      fit: BoxFit.cover,
                    )
                  : Container(color: theme.colorScheme.primary),
            ),
          ),
          SliverToBoxAdapter(
            child: Padding(
              padding: const EdgeInsets.all(20),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(
                    children: [
                      const Icon(Icons.source_outlined, size: 16),
                      const SizedBox(width: 4),
                      Text(article.source,
                          style: TextStyle(color: theme.colorScheme.primary)),
                      const Spacer(),
                      Text(article.publishedAt.substring(0, 10),
                          style: theme.textTheme.bodySmall),
                    ],
                  ),
                  const SizedBox(height: 16),
                  Text(article.title,
                      style: theme.textTheme.headlineSmall
                          ?.copyWith(fontWeight: FontWeight.bold)),
                  const SizedBox(height: 12),
                  const Divider(),
                  const SizedBox(height: 12),
                  Text(article.description,
                      style: theme.textTheme.bodyLarge
                          ?.copyWith(height: 1.6)),
                  const SizedBox(height: 24),
                  SizedBox(
                    width: double.infinity,
                    child: OutlinedButton.icon(
                      icon: const Icon(Icons.open_in_browser),
                      label: const Text('Read Full Article'),
                      onPressed: () {
                        // Use url_launcher package to open article.url
                      },
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

### Wiring It All Up

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:http/http.dart' as http;

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const MyApp());
}

// lib/app.dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => NewsController(
        NewsRepository(
          NewsService(),
        ),
      ),
      child: MaterialApp(
        title: 'News Reader',
        debugShowCheckedModeBanner: false,
        theme: AppTheme.light,
        darkTheme: AppTheme.dark,
        home: const NewsListScreen(),
      ),
    );
  }
}
```

---

### What This Mini Project Covers

| Concept | Where |
|---|---|
| Model / `fromJson` | `Article` class |
| Repository pattern | `NewsRepository` |
| Service layer | `NewsService` |
| `ChangeNotifier` + Provider | `NewsController` |
| `context.watch` / `context.read` | `NewsListScreen` |
| `initState` + `addPostFrameCallback` | Initial data load |
| Loading / error / empty states | `_buildBody()` |
| `ListView.builder` | Article list |
| `Navigator.push` with data | Card → Detail |
| `RefreshIndicator` | Pull to refresh |
| `CustomScrollView` + `SliverAppBar` | Detail screen |
| `CachedNetworkImage` | Article images |
| Theming | `AppTheme` usage throughout |

---

## ⚡ Quick Reference Cheatsheet

```dart
// Widget types
StatelessWidget   // No state
StatefulWidget    // Has mutable state

// State update
setState(() { ... });         // Rebuild this widget
notifyListeners();            // Rebuild all Provider listeners

// Navigation
Navigator.push(context, MaterialPageRoute(builder: (_) => Screen()));
Navigator.pop(context);
context.go('/route');         // GoRouter

// Layout
Expanded(child: widget)       // Fill remaining space
Flexible(child: widget)       // Take available space, shrink if needed
SizedBox(height: 16)          // Spacing
SizedBox.expand(child: ...)   // Fill entire parent

// Theme
Theme.of(context).colorScheme.primary
Theme.of(context).textTheme.bodyLarge

// Async widgets
FutureBuilder<T>(future: ..., builder: (ctx, snapshot) { ... })
StreamBuilder<T>(stream: ..., builder: (ctx, snapshot) { ... })

// Snapshot states
snapshot.connectionState == ConnectionState.waiting  // Loading
snapshot.hasError                                    // Error
snapshot.hasData                                     // Success

// Provider
context.read<T>()     // Read without listen (in callbacks)
context.watch<T>()    // Read with listen (in build)
context.select<T, R>((t) => t.value)  // Selective listen

// Dispose
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}
```

---

## 🔜 Next Steps

| What to Learn | Why |
|---|---|
| **GoRouter** | Deep links, web, nested navigation |
| **Riverpod 2.x** | More powerful and testable than Provider |
| **Hive / Isar** | Fast local database for offline apps |
| **Flutter Testing** | Unit, widget, integration tests |
| **Firebase** | Auth, Firestore, push notifications |
| **Animations** | `AnimatedContainer`, `Hero`, `Rive` |
| **Platform Channels** | Native iOS/Android code interop |
| **Flutter Web** | Responsive layouts, web-specific APIs |
| **CI/CD** | Codemagic, GitHub Actions for builds |

---

> **Keep building. The best Flutter developers aren't those who memorised the docs — they're those who shipped things, debugged weird layout issues at midnight, and kept going. 🚀**
