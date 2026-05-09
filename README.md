EnglishFor2Day - Rebuild of Mobile App.

Complete architectural rebuild and modernisation of the EnglishFor2Day educational app – made with Flutter.

This project was designed with the following major limitations of the legacy application in mind: very network-dependent rendering, high RAM usage while parsing rich-text, and a non-responsive user interface.

## 🎯 The Problem & Purpose
EnglishFor2Day has a huge audience, such as students with cell phone networks that may sometimes be unstable, and with low cost Android smartphones. The legacy application had issues with bloated memory usage when loading large amounts of grammar tables, and crashed or hung if the Internet connection was lost during screen transitions.

**The Solution:**
I refactored the application with a Feature-First Clean Architecture using a strict Offline-First Data Pipeline. The app now stores responses from the API in a local NoSQL database (Hive). When the network fails, the repository layer automatically switches back to the cache, without disrupting the student's reading experience.

## 🎨 The "Strategy Pattern" UI Engine.
The UI engine using the "Strategy Pattern". I designed a global Design System Controller to meet the legacy user experience, while maintaining the look and feel of modern design. The app will include 2 full visual layers which run independently of one another, both with all the same business logic.

### 1. The Classic Theme
A faithful copy of the original app, with solid colours, typical Material drop shadow, and standard list views. Geared towards those who like to be precisely familiar.

### 2. The 3D Glassmorphism Theme
A very high end, touch-driven interface, developed from scratch. 
Mechanical Physics: Buttons are not flat, instead they use `Matrix4.translationValues` to actually push into the Z axis when they are tapped, which more closely resembles a mechanical keyboard.
On old phones, real-time `BackdropFilter` blurs consume battery life as a result of Zero-Overhead Glass. To get the frosted-glass effect, I used Aurora Mesh (static radial gradients) and calibrated the opacity layers to produce the effect of a foggy glass window, while keeping the GPU almost completely unused.
3D Page Curls: Reading grammar rules: Making regular lists of grammar rules into a real book.

![UI Showcase](https://github.com/user-attachments/assets/e43e09a1-a8e9-4f08-9814-638fa92f51d2)

## 🏗️ Architecture & Tech Stack

*   **Framework:** Flutter / Dart
*   **Architecture:** Feature-First Clean Architecture (Domain, Data, Presentation)
*   **State Management:** GetX (Strictly scoped via local UI constructors, pure Rx reactive states)
*   **Dependency Injection:** `get_it` (Factory & Singleton scoping)
*   **Routing:** `go_router` (Nested routing trees for deep-linking)
*   **Local Storage:** Hive (NoSQL) & `SharedPreferencesAsync`
*   **Networking:** `http` with custom Generic Interceptors

### Folder Structure
```text
lib/
├── core/
│   ├── errors/           # Failure and Exception mapping
│   ├── network/          # HTTP wrapper and connection checkers
│   ├── theme/            # AppColors and DesignSystemController
│   ├── utils/            # FixedResponsive scaling engine
│   └── widgets/          # Global UI components (Overflow Menu, Mesh Backgrounds)
├── features/
│   ├── dashboard/        
│   ├── form_verbs/       # Dynamic tabular data rendering
│   ├── right_forms/      
│   ├── small_talks/      
│   ├── spoken_rules/     # Heavy HTML parsing and 3D pagination
│   └── video_lectures/   # Native YouTube player integration
├── service_locator.dart  # Dependency Injection wiring
└── main.dart
```
## ⚡ Core Optimizations & Edge Case Handling

### Asynchronous HTML Parsing:
The API returns heavily nested HTML strings with inline CSS. Parsing this on the main UI thread during a route transition caused heavy frame drops. I abstracted the text sanitization into a background Isolate (compute), allowing the screens to render instantly and populate the text milliseconds later.

### Memory Lifecycle Management:
By decoupling the Router from the State Management package, I gained strict control over the RAM. Controllers are injected via get_it only when their specific GoRoute is active. Open streams are manually closed in the dispose cycle, ensuring the garbage collector reclaims 100% of the memory when the user exits a feature.

### Responsive Scaling (Shortest-Side Math):
Standard MediaQuery width implementations explode when a user rotates their phone to landscape. I wrote a custom extension that calculates UI scale based on the shortestSide of the device's PlatformDispatcher. This guarantees the app looks perfectly proportioned on an iPad, but ignores rotation scaling on a standard mobile device.

### Dynamic Table Layouts:
API responses for vocabulary and verbs frequently lacked specific fields (e.g., missing Antonyms or Images). Instead of throwing RenderFlex errors or leaving blank gaps, the UI components dynamically collapse and redraw their constraints to maintain a clean, unbroken layout.
