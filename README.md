 (cd "$(git rev-parse --show-toplevel)" && git apply --3way <<'EOF' 
diff --git a/.gitignore b/.gitignore
new file mode 100644
index 0000000000000000000000000000000000000000..e722e73f979c825e5737573fe49708da978d722f
--- /dev/null
+++ b/.gitignore
@@ -0,0 +1,12 @@
+.dart_tool/
+.packages
+.pub-cache/
+.pub/
+build/
+.flutter-plugins
+.flutter-plugins-dependencies
+.metadata
+.idea/
+.vscode/
+android/.gradle/
+android/local.properties
diff --git a/README.md b/README.md
new file mode 100644
index 0000000000000000000000000000000000000000..db167cf99655f980fc980ae499a6bcea72d3aa92
--- /dev/null
+++ b/README.md
@@ -0,0 +1,108 @@
+# Kriolu AI Dictionary (Flutter)
+
+Kriolu AI Dictionary is an offline-first Flutter mobile app to help learners study Cape Verdean Creole (Kriolu) through:
+
+- Offline dictionary search
+- Word detail with examples and speech
+- Favorites saved locally
+- AI learning mode (flashcards, quiz, conversation practice)
+- AI chat teacher (rule-based starter implementation)
+
+## Tech Stack
+
+- Flutter + Material Design 3
+- Provider (state management)
+- SQFlite (local persistence for favorites)
+- `flutter_tts` (text-to-speech robot)
+
+## Project Structure
+
+```text
+lib/
+├── ai/
+├── database/
+├── models/
+├── providers/
+├── screens/
+├── services/
+└── widgets/
+assets/
+├── audio/
+├── data/
+└── images/
+```
+
+## Features Implemented
+
+1. **Dictionary (offline)**
+   - Loads `assets/data/dictionary.json`.
+   - Includes 500 sample entries.
+   - Data model contains:
+     - `creole_word`
+     - `portuguese_translation`
+     - `english_translation`
+     - `example_creole`
+     - `example_portuguese`
+     - `audio`
+
+2. **Real-time Search**
+   - Filters by Kriolu, Portuguese, and English terms.
+
+3. **Word Detail Page**
+   - Word, translations, examples, audio controls, favorite toggle.
+
+4. **Speaking Robot (TTS)**
+   - Uses `flutter_tts`.
+   - Pronounces words and example sentences.
+   - Speech speed control.
+
+5. **Favorites**
+   - Stored locally using SQLite.
+
+6. **AI Learning Mode**
+   - Flashcards
+   - Quiz
+   - Conversation practice prompts
+
+7. **AI Chat Teacher**
+   - Starter offline conversational helper with grammar patterns and phrase guidance.
+
+8. **Bottom Navigation**
+   - Dictionary, Favorites, Learn, AI Chat.
+
+## Assets
+
+- Dictionary data: `assets/data/dictionary.json`
+- Audio folder for pronunciations: `assets/audio/`
+- Splash image: `assets/images/splash_logo.png`
+- App icon: `assets/icons/app_icon.png`
+
+## Run Instructions
+
+1. Install Flutter SDK and Android Studio.
+2. Run:
+   ```bash
+   flutter pub get
+   flutter pub run flutter_launcher_icons
+   flutter pub run flutter_native_splash:create
+   flutter run
+   ```
+
+## Release / Play Store Readiness Checklist
+
+- [x] Material-based production app structure
+- [x] Offline dictionary and local favorites
+- [x] App icon config (`flutter_launcher_icons`)
+- [x] Splash screen config (`flutter_native_splash`)
+- [x] Release profile with ProGuard enabled (`android/app/build.gradle`)
+- [ ] Replace placeholder icon/splash assets with branded final artwork
+- [ ] Configure Play Console signing keystore (`key.properties` + signingConfigs)
+- [ ] Add privacy policy URL and store listing metadata
+
+## Scaling Notes for 10,000+ Entries
+
+- Move to indexed local database table for dictionary entries.
+- Add debounced search and prefix indexes.
+- Consider FTS (full-text search) in SQLite.
+- Paginate or virtualize heavy lists.
+- Cache normalized search tokens.
diff --git a/analysis_options.yaml b/analysis_options.yaml
new file mode 100644
index 0000000000000000000000000000000000000000..24779714094651d8af10f608a9d0fa4be322c5a1
--- /dev/null
+++ b/analysis_options.yaml
@@ -0,0 +1,6 @@
+include: package:flutter_lints/flutter.yaml
+
+linter:
+  rules:
+    prefer_single_quotes: true
+    directives_ordering: true
diff --git a/android/app/build.gradle b/android/app/build.gradle
new file mode 100644
index 0000000000000000000000000000000000000000..060eb47be2543fc818e5fa18cdf50711968ba3f5
--- /dev/null
+++ b/android/app/build.gradle
@@ -0,0 +1,34 @@
+plugins {
+    id 'com.android.application'
+    id 'kotlin-android'
+}
+
+android {
+    namespace 'com.kriolu.ai.dictionary'
+    compileSdk 34
+
+    defaultConfig {
+        applicationId 'com.kriolu.ai.dictionary'
+        minSdk 23
+        targetSdk 34
+        versionCode 1
+        versionName '1.0.0'
+    }
+
+    buildTypes {
+        release {
+            minifyEnabled true
+            shrinkResources true
+            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
+        }
+    }
+
+    compileOptions {
+        sourceCompatibility JavaVersion.VERSION_17
+        targetCompatibility JavaVersion.VERSION_17
+    }
+
+    kotlinOptions {
+        jvmTarget = '17'
+    }
+}
diff --git a/android/app/proguard-rules.pro b/android/app/proguard-rules.pro
new file mode 100644
index 0000000000000000000000000000000000000000..ca34e358614c8144c9213533cb6455a833543931
--- /dev/null
+++ b/android/app/proguard-rules.pro
@@ -0,0 +1,4 @@
+# Keep Flutter engine and plugin classes when shrinking in release mode.
+-keep class io.flutter.** { *; }
+-keep class io.flutter.plugins.** { *; }
+-dontwarn io.flutter.embedding.**
diff --git a/android/build.gradle b/android/build.gradle
new file mode 100644
index 0000000000000000000000000000000000000000..0fa091847bfe69437bb93daac16a5498b521154d
--- /dev/null
+++ b/android/build.gradle
@@ -0,0 +1,17 @@
+buildscript {
+    repositories {
+        google()
+        mavenCentral()
+    }
+    dependencies {
+        classpath 'com.android.tools.build:gradle:8.4.1'
+        classpath 'org.jetbrains.kotlin:kotlin-gradle-plugin:1.9.24'
+    }
+}
+
+allprojects {
+    repositories {
+        google()
+        mavenCentral()
+    }
+}
diff --git a/android/gradle.properties b/android/gradle.properties
new file mode 100644
index 0000000000000000000000000000000000000000..3b5b324f6e3f162acfb54a7c423f4566fe04bae1
--- /dev/null
+++ b/android/gradle.properties
@@ -0,0 +1,3 @@
+org.gradle.jvmargs=-Xmx4G -XX:+HeapDumpOnOutOfMemoryError
+android.useAndroidX=true
+android.enableJetifier=true
diff --git a/android/settings.gradle b/android/settings.gradle
new file mode 100644
index 0000000000000000000000000000000000000000..e61b4aba4c09353c8a2ab625efbd39336110612a
--- /dev/null
+++ b/android/settings.gradle
@@ -0,0 +1,10 @@
+pluginManagement {
+    repositories {
+        google()
+        mavenCentral()
+        gradlePluginPortal()
+    }
+}
+
+rootProject.name = 'kriolu_ai_dictionary'
+include ':app'
diff --git a/assets/audio/.gitkeep b/assets/audio/.gitkeep
new file mode 100644
index 0000000000000000000000000000000000000000..e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
diff --git a/assets/data/dictionary.json b/assets/data/dictionary.json
new file mode 100644
index 0000000000000000000000000000000000000000..7a703dd163bb5953545db90f15b777c114367e5e
--- /dev/null
+++ b/assets/data/dictionary.json
@@ -0,0 +1,4002 @@
+[
+  {
+    "creole_word": "kume",
+    "portuguese_translation": "comer",
+    "english_translation": "eat",
+    "example_creole": "N ta kume pan.",
+    "example_portuguese": "Eu como pão.",
+    "audio": "assets/audio/kume.mp3"
+  },
+  {
+    "creole_word": "bai_001",
+    "portuguese_translation": "ir 1",
+    "english_translation": "go 1",
+    "example_creole": "N ta bai tudu dia 1.",
+    "example_portuguese": "Eu ir todos os dias 1.",
+    "audio": "assets/audio/bai_001.mp3"
+  },
+  {
+    "creole_word": "odja_002",
+    "portuguese_translation": "ver 2",
+    "english_translation": "see 2",
+    "example_creole": "N ta odja tudu dia 2.",
+    "example_portuguese": "Eu ver todos os dias 2.",
+    "audio": "assets/audio/odja_002.mp3"
+  },
+  {
+    "creole_word": "fla_003",
+    "portuguese_translation": "falar 3",
+    "english_translation": "speak 3",
+    "example_creole": "N ta fla tudu dia 3.",
+    "example_portuguese": "Eu falar todos os dias 3.",
+    "audio": "assets/audio/fla_003.mp3"
+  },
+  {
+    "creole_word": "kanta_004",
+    "portuguese_translation": "cantar 4",
+    "english_translation": "sing 4",
+    "example_creole": "N ta kanta tudu dia 4.",
+    "example_portuguese": "Eu cantar todos os dias 4.",
+    "audio": "assets/audio/kanta_004.mp3"
+  },
+  {
+    "creole_word": "durmi_005",
+    "portuguese_translation": "dormir 5",
+    "english_translation": "sleep 5",
+    "example_creole": "N ta durmi tudu dia 5.",
+    "example_portuguese": "Eu dormir todos os dias 5.",
+    "audio": "assets/audio/durmi_005.mp3"
+  },
+  {
+    "creole_word": "studia_006",
+    "portuguese_translation": "estudar 6",
+    "english_translation": "study 6",
+    "example_creole": "N ta studia tudu dia 6.",
+    "example_portuguese": "Eu estudar todos os dias 6.",
+    "audio": "assets/audio/studia_006.mp3"
+  },
+  {
+    "creole_word": "trabadja_007",
+    "portuguese_translation": "trabalhar 7",
+    "english_translation": "work 7",
+    "example_creole": "N ta trabadja tudu dia 7.",
+    "example_portuguese": "Eu trabalhar todos os dias 7.",
+    "audio": "assets/audio/trabadja_007.mp3"
+  },
+  {
+    "creole_word": "kore_008",
+    "portuguese_translation": "correr 8",
+    "english_translation": "run 8",
+    "example_creole": "N ta kore tudu dia 8.",
+    "example_portuguese": "Eu correr todos os dias 8.",
+    "audio": "assets/audio/kore_008.mp3"
+  },
+  {
+    "creole_word": "skreve_009",
+    "portuguese_translation": "escrever 9",
+    "english_translation": "write 9",
+    "example_creole": "N ta skreve tudu dia 9.",
+    "example_portuguese": "Eu escrever todos os dias 9.",
+    "audio": "assets/audio/skreve_009.mp3"
+  },
+  {
+    "creole_word": "bebe_010",
+    "portuguese_translation": "beber 10",
+    "english_translation": "drink 10",
+    "example_creole": "N ta bebe tudu dia 10.",
+    "example_portuguese": "Eu beber todos os dias 10.",
+    "audio": "assets/audio/bebe_010.mp3"
+  },
+  {
+    "creole_word": "bai_011",
+    "portuguese_translation": "ir 11",
+    "english_translation": "go 11",
+    "example_creole": "N ta bai tudu dia 11.",
+    "example_portuguese": "Eu ir todos os dias 11.",
+    "audio": "assets/audio/bai_011.mp3"
+  },
+  {
+    "creole_word": "odja_012",
+    "portuguese_translation": "ver 12",
+    "english_translation": "see 12",
+    "example_creole": "N ta odja tudu dia 12.",
+    "example_portuguese": "Eu ver todos os dias 12.",
+    "audio": "assets/audio/odja_012.mp3"
+  },
+  {
+    "creole_word": "fla_013",
+    "portuguese_translation": "falar 13",
+    "english_translation": "speak 13",
+    "example_creole": "N ta fla tudu dia 13.",
+    "example_portuguese": "Eu falar todos os dias 13.",
+    "audio": "assets/audio/fla_013.mp3"
+  },
+  {
+    "creole_word": "kanta_014",
+    "portuguese_translation": "cantar 14",
+    "english_translation": "sing 14",
+    "example_creole": "N ta kanta tudu dia 14.",
+    "example_portuguese": "Eu cantar todos os dias 14.",
+    "audio": "assets/audio/kanta_014.mp3"
+  },
+  {
+    "creole_word": "durmi_015",
+    "portuguese_translation": "dormir 15",
+    "english_translation": "sleep 15",
+    "example_creole": "N ta durmi tudu dia 15.",
+    "example_portuguese": "Eu dormir todos os dias 15.",
+    "audio": "assets/audio/durmi_015.mp3"
+  },
+  {
+    "creole_word": "studia_016",
+    "portuguese_translation": "estudar 16",
+    "english_translation": "study 16",
+    "example_creole": "N ta studia tudu dia 16.",
+    "example_portuguese": "Eu estudar todos os dias 16.",
+    "audio": "assets/audio/studia_016.mp3"
+  },
+  {
+    "creole_word": "trabadja_017",
+    "portuguese_translation": "trabalhar 17",
+    "english_translation": "work 17",
+    "example_creole": "N ta trabadja tudu dia 17.",
+    "example_portuguese": "Eu trabalhar todos os dias 17.",
+    "audio": "assets/audio/trabadja_017.mp3"
+  },
+  {
+    "creole_word": "kore_018",
+    "portuguese_translation": "correr 18",
+    "english_translation": "run 18",
+    "example_creole": "N ta kore tudu dia 18.",
+    "example_portuguese": "Eu correr todos os dias 18.",
+    "audio": "assets/audio/kore_018.mp3"
+  },
+  {
+    "creole_word": "skreve_019",
+    "portuguese_translation": "escrever 19",
+    "english_translation": "write 19",
+    "example_creole": "N ta skreve tudu dia 19.",
+    "example_portuguese": "Eu escrever todos os dias 19.",
+    "audio": "assets/audio/skreve_019.mp3"
+  },
+  {
+    "creole_word": "bebe_020",
+    "portuguese_translation": "beber 20",
+    "english_translation": "drink 20",
+    "example_creole": "N ta bebe tudu dia 20.",
+    "example_portuguese": "Eu beber todos os dias 20.",
+    "audio": "assets/audio/bebe_020.mp3"
+  },
+  {
+    "creole_word": "bai_021",
+    "portuguese_translation": "ir 21",
+    "english_translation": "go 21",
+    "example_creole": "N ta bai tudu dia 21.",
+    "example_portuguese": "Eu ir todos os dias 21.",
+    "audio": "assets/audio/bai_021.mp3"
+  },
+  {
+    "creole_word": "odja_022",
+    "portuguese_translation": "ver 22",
+    "english_translation": "see 22",
+    "example_creole": "N ta odja tudu dia 22.",
+    "example_portuguese": "Eu ver todos os dias 22.",
+    "audio": "assets/audio/odja_022.mp3"
+  },
+  {
+    "creole_word": "fla_023",
+    "portuguese_translation": "falar 23",
+    "english_translation": "speak 23",
+    "example_creole": "N ta fla tudu dia 23.",
+    "example_portuguese": "Eu falar todos os dias 23.",
+    "audio": "assets/audio/fla_023.mp3"
+  },
+  {
+    "creole_word": "kanta_024",
+    "portuguese_translation": "cantar 24",
+    "english_translation": "sing 24",
+    "example_creole": "N ta kanta tudu dia 24.",
+    "example_portuguese": "Eu cantar todos os dias 24.",
+    "audio": "assets/audio/kanta_024.mp3"
+  },
+  {
+    "creole_word": "durmi_025",
+    "portuguese_translation": "dormir 25",
+    "english_translation": "sleep 25",
+    "example_creole": "N ta durmi tudu dia 25.",
+    "example_portuguese": "Eu dormir todos os dias 25.",
+    "audio": "assets/audio/durmi_025.mp3"
+  },
+  {
+    "creole_word": "studia_026",
+    "portuguese_translation": "estudar 26",
+    "english_translation": "study 26",
+    "example_creole": "N ta studia tudu dia 26.",
+    "example_portuguese": "Eu estudar todos os dias 26.",
+    "audio": "assets/audio/studia_026.mp3"
+  },
+  {
+    "creole_word": "trabadja_027",
+    "portuguese_translation": "trabalhar 27",
+    "english_translation": "work 27",
+    "example_creole": "N ta trabadja tudu dia 27.",
+    "example_portuguese": "Eu trabalhar todos os dias 27.",
+    "audio": "assets/audio/trabadja_027.mp3"
+  },
+  {
+    "creole_word": "kore_028",
+    "portuguese_translation": "correr 28",
+    "english_translation": "run 28",
+    "example_creole": "N ta kore tudu dia 28.",
+    "example_portuguese": "Eu correr todos os dias 28.",
+    "audio": "assets/audio/kore_028.mp3"
+  },
+  {
+    "creole_word": "skreve_029",
+    "portuguese_translation": "escrever 29",
+    "english_translation": "write 29",
+    "example_creole": "N ta skreve tudu dia 29.",
+    "example_portuguese": "Eu escrever todos os dias 29.",
+    "audio": "assets/audio/skreve_029.mp3"
+  },
+  {
+    "creole_word": "bebe_030",
+    "portuguese_translation": "beber 30",
+    "english_translation": "drink 30",
+    "example_creole": "N ta bebe tudu dia 30.",
+    "example_portuguese": "Eu beber todos os dias 30.",
+    "audio": "assets/audio/bebe_030.mp3"
+  },
+  {
+    "creole_word": "bai_031",
+    "portuguese_translation": "ir 31",
+    "english_translation": "go 31",
+    "example_creole": "N ta bai tudu dia 31.",
+    "example_portuguese": "Eu ir todos os dias 31.",
+    "audio": "assets/audio/bai_031.mp3"
+  },
+  {
+    "creole_word": "odja_032",
+    "portuguese_translation": "ver 32",
+    "english_translation": "see 32",
+    "example_creole": "N ta odja tudu dia 32.",
+    "example_portuguese": "Eu ver todos os dias 32.",
+    "audio": "assets/audio/odja_032.mp3"
+  },
+  {
+    "creole_word": "fla_033",
+    "portuguese_translation": "falar 33",
+    "english_translation": "speak 33",
+    "example_creole": "N ta fla tudu dia 33.",
+    "example_portuguese": "Eu falar todos os dias 33.",
+    "audio": "assets/audio/fla_033.mp3"
+  },
+  {
+    "creole_word": "kanta_034",
+    "portuguese_translation": "cantar 34",
+    "english_translation": "sing 34",
+    "example_creole": "N ta kanta tudu dia 34.",
+    "example_portuguese": "Eu cantar todos os dias 34.",
+    "audio": "assets/audio/kanta_034.mp3"
+  },
+  {
+    "creole_word": "durmi_035",
+    "portuguese_translation": "dormir 35",
+    "english_translation": "sleep 35",
+    "example_creole": "N ta durmi tudu dia 35.",
+    "example_portuguese": "Eu dormir todos os dias 35.",
+    "audio": "assets/audio/durmi_035.mp3"
+  },
+  {
+    "creole_word": "studia_036",
+    "portuguese_translation": "estudar 36",
+    "english_translation": "study 36",
+    "example_creole": "N ta studia tudu dia 36.",
+    "example_portuguese": "Eu estudar todos os dias 36.",
+    "audio": "assets/audio/studia_036.mp3"
+  },
+  {
+    "creole_word": "trabadja_037",
+    "portuguese_translation": "trabalhar 37",
+    "english_translation": "work 37",
+    "example_creole": "N ta trabadja tudu dia 37.",
+    "example_portuguese": "Eu trabalhar todos os dias 37.",
+    "audio": "assets/audio/trabadja_037.mp3"
+  },
+  {
+    "creole_word": "kore_038",
+    "portuguese_translation": "correr 38",
+    "english_translation": "run 38",
+    "example_creole": "N ta kore tudu dia 38.",
+    "example_portuguese": "Eu correr todos os dias 38.",
+    "audio": "assets/audio/kore_038.mp3"
+  },
+  {
+    "creole_word": "skreve_039",
+    "portuguese_translation": "escrever 39",
+    "english_translation": "write 39",
+    "example_creole": "N ta skreve tudu dia 39.",
+    "example_portuguese": "Eu escrever todos os dias 39.",
+    "audio": "assets/audio/skreve_039.mp3"
+  },
+  {
+    "creole_word": "bebe_040",
+    "portuguese_translation": "beber 40",
+    "english_translation": "drink 40",
+    "example_creole": "N ta bebe tudu dia 40.",
+    "example_portuguese": "Eu beber todos os dias 40.",
+    "audio": "assets/audio/bebe_040.mp3"
+  },
+  {
+    "creole_word": "bai_041",
+    "portuguese_translation": "ir 41",
+    "english_translation": "go 41",
+    "example_creole": "N ta bai tudu dia 41.",
+    "example_portuguese": "Eu ir todos os dias 41.",
+    "audio": "assets/audio/bai_041.mp3"
+  },
+  {
+    "creole_word": "odja_042",
+    "portuguese_translation": "ver 42",
+    "english_translation": "see 42",
+    "example_creole": "N ta odja tudu dia 42.",
+    "example_portuguese": "Eu ver todos os dias 42.",
+    "audio": "assets/audio/odja_042.mp3"
+  },
+  {
+    "creole_word": "fla_043",
+    "portuguese_translation": "falar 43",
+    "english_translation": "speak 43",
+    "example_creole": "N ta fla tudu dia 43.",
+    "example_portuguese": "Eu falar todos os dias 43.",
+    "audio": "assets/audio/fla_043.mp3"
+  },
+  {
+    "creole_word": "kanta_044",
+    "portuguese_translation": "cantar 44",
+    "english_translation": "sing 44",
+    "example_creole": "N ta kanta tudu dia 44.",
+    "example_portuguese": "Eu cantar todos os dias 44.",
+    "audio": "assets/audio/kanta_044.mp3"
+  },
+  {
+    "creole_word": "durmi_045",
+    "portuguese_translation": "dormir 45",
+    "english_translation": "sleep 45",
+    "example_creole": "N ta durmi tudu dia 45.",
+    "example_portuguese": "Eu dormir todos os dias 45.",
+    "audio": "assets/audio/durmi_045.mp3"
+  },
+  {
+    "creole_word": "studia_046",
+    "portuguese_translation": "estudar 46",
+    "english_translation": "study 46",
+    "example_creole": "N ta studia tudu dia 46.",
+    "example_portuguese": "Eu estudar todos os dias 46.",
+    "audio": "assets/audio/studia_046.mp3"
+  },
+  {
+    "creole_word": "trabadja_047",
+    "portuguese_translation": "trabalhar 47",
+    "english_translation": "work 47",
+    "example_creole": "N ta trabadja tudu dia 47.",
+    "example_portuguese": "Eu trabalhar todos os dias 47.",
+    "audio": "assets/audio/trabadja_047.mp3"
+  },
+  {
+    "creole_word": "kore_048",
+    "portuguese_translation": "correr 48",
+    "english_translation": "run 48",
+    "example_creole": "N ta kore tudu dia 48.",
+    "example_portuguese": "Eu correr todos os dias 48.",
+    "audio": "assets/audio/kore_048.mp3"
+  },
+  {
+    "creole_word": "skreve_049",
+    "portuguese_translation": "escrever 49",
+    "english_translation": "write 49",
+    "example_creole": "N ta skreve tudu dia 49.",
+    "example_portuguese": "Eu escrever todos os dias 49.",
+    "audio": "assets/audio/skreve_049.mp3"
+  },
+  {
+    "creole_word": "bebe_050",
+    "portuguese_translation": "beber 50",
+    "english_translation": "drink 50",
+    "example_creole": "N ta bebe tudu dia 50.",
+    "example_portuguese": "Eu beber todos os dias 50.",
+    "audio": "assets/audio/bebe_050.mp3"
+  },
+  {
+    "creole_word": "bai_051",
+    "portuguese_translation": "ir 51",
+    "english_translation": "go 51",
+    "example_creole": "N ta bai tudu dia 51.",
+    "example_portuguese": "Eu ir todos os dias 51.",
+    "audio": "assets/audio/bai_051.mp3"
+  },
+  {
+    "creole_word": "odja_052",
+    "portuguese_translation": "ver 52",
+    "english_translation": "see 52",
+    "example_creole": "N ta odja tudu dia 52.",
+    "example_portuguese": "Eu ver todos os dias 52.",
+    "audio": "assets/audio/odja_052.mp3"
+  },
+  {
+    "creole_word": "fla_053",
+    "portuguese_translation": "falar 53",
+    "english_translation": "speak 53",
+    "example_creole": "N ta fla tudu dia 53.",
+    "example_portuguese": "Eu falar todos os dias 53.",
+    "audio": "assets/audio/fla_053.mp3"
+  },
+  {
+    "creole_word": "kanta_054",
+    "portuguese_translation": "cantar 54",
+    "english_translation": "sing 54",
+    "example_creole": "N ta kanta tudu dia 54.",
+    "example_portuguese": "Eu cantar todos os dias 54.",
+    "audio": "assets/audio/kanta_054.mp3"
+  },
+  {
+    "creole_word": "durmi_055",
+    "portuguese_translation": "dormir 55",
+    "english_translation": "sleep 55",
+    "example_creole": "N ta durmi tudu dia 55.",
+    "example_portuguese": "Eu dormir todos os dias 55.",
+    "audio": "assets/audio/durmi_055.mp3"
+  },
+  {
+    "creole_word": "studia_056",
+    "portuguese_translation": "estudar 56",
+    "english_translation": "study 56",
+    "example_creole": "N ta studia tudu dia 56.",
+    "example_portuguese": "Eu estudar todos os dias 56.",
+    "audio": "assets/audio/studia_056.mp3"
+  },
+  {
+    "creole_word": "trabadja_057",
+    "portuguese_translation": "trabalhar 57",
+    "english_translation": "work 57",
+    "example_creole": "N ta trabadja tudu dia 57.",
+    "example_portuguese": "Eu trabalhar todos os dias 57.",
+    "audio": "assets/audio/trabadja_057.mp3"
+  },
+  {
+    "creole_word": "kore_058",
+    "portuguese_translation": "correr 58",
+    "english_translation": "run 58",
+    "example_creole": "N ta kore tudu dia 58.",
+    "example_portuguese": "Eu correr todos os dias 58.",
+    "audio": "assets/audio/kore_058.mp3"
+  },
+  {
+    "creole_word": "skreve_059",
+    "portuguese_translation": "escrever 59",
+    "english_translation": "write 59",
+    "example_creole": "N ta skreve tudu dia 59.",
+    "example_portuguese": "Eu escrever todos os dias 59.",
+    "audio": "assets/audio/skreve_059.mp3"
+  },
+  {
+    "creole_word": "bebe_060",
+    "portuguese_translation": "beber 60",
+    "english_translation": "drink 60",
+    "example_creole": "N ta bebe tudu dia 60.",
+    "example_portuguese": "Eu beber todos os dias 60.",
+    "audio": "assets/audio/bebe_060.mp3"
+  },
+  {
+    "creole_word": "bai_061",
+    "portuguese_translation": "ir 61",
+    "english_translation": "go 61",
+    "example_creole": "N ta bai tudu dia 61.",
+    "example_portuguese": "Eu ir todos os dias 61.",
+    "audio": "assets/audio/bai_061.mp3"
+  },
+  {
+    "creole_word": "odja_062",
+    "portuguese_translation": "ver 62",
+    "english_translation": "see 62",
+    "example_creole": "N ta odja tudu dia 62.",
+    "example_portuguese": "Eu ver todos os dias 62.",
+    "audio": "assets/audio/odja_062.mp3"
+  },
+  {
+    "creole_word": "fla_063",
+    "portuguese_translation": "falar 63",
+    "english_translation": "speak 63",
+    "example_creole": "N ta fla tudu dia 63.",
+    "example_portuguese": "Eu falar todos os dias 63.",
+    "audio": "assets/audio/fla_063.mp3"
+  },
+  {
+    "creole_word": "kanta_064",
+    "portuguese_translation": "cantar 64",
+    "english_translation": "sing 64",
+    "example_creole": "N ta kanta tudu dia 64.",
+    "example_portuguese": "Eu cantar todos os dias 64.",
+    "audio": "assets/audio/kanta_064.mp3"
+  },
+  {
+    "creole_word": "durmi_065",
+    "portuguese_translation": "dormir 65",
+    "english_translation": "sleep 65",
+    "example_creole": "N ta durmi tudu dia 65.",
+    "example_portuguese": "Eu dormir todos os dias 65.",
+    "audio": "assets/audio/durmi_065.mp3"
+  },
+  {
+    "creole_word": "studia_066",
+    "portuguese_translation": "estudar 66",
+    "english_translation": "study 66",
+    "example_creole": "N ta studia tudu dia 66.",
+    "example_portuguese": "Eu estudar todos os dias 66.",
+    "audio": "assets/audio/studia_066.mp3"
+  },
+  {
+    "creole_word": "trabadja_067",
+    "portuguese_translation": "trabalhar 67",
+    "english_translation": "work 67",
+    "example_creole": "N ta trabadja tudu dia 67.",
+    "example_portuguese": "Eu trabalhar todos os dias 67.",
+    "audio": "assets/audio/trabadja_067.mp3"
+  },
+  {
+    "creole_word": "kore_068",
+    "portuguese_translation": "correr 68",
+    "english_translation": "run 68",
+    "example_creole": "N ta kore tudu dia 68.",
+    "example_portuguese": "Eu correr todos os dias 68.",
+    "audio": "assets/audio/kore_068.mp3"
+  },
+  {
+    "creole_word": "skreve_069",
+    "portuguese_translation": "escrever 69",
+    "english_translation": "write 69",
+    "example_creole": "N ta skreve tudu dia 69.",
+    "example_portuguese": "Eu escrever todos os dias 69.",
+    "audio": "assets/audio/skreve_069.mp3"
+  },
+  {
+    "creole_word": "bebe_070",
+    "portuguese_translation": "beber 70",
+    "english_translation": "drink 70",
+    "example_creole": "N ta bebe tudu dia 70.",
+    "example_portuguese": "Eu beber todos os dias 70.",
+    "audio": "assets/audio/bebe_070.mp3"
+  },
+  {
+    "creole_word": "bai_071",
+    "portuguese_translation": "ir 71",
+    "english_translation": "go 71",
+    "example_creole": "N ta bai tudu dia 71.",
+    "example_portuguese": "Eu ir todos os dias 71.",
+    "audio": "assets/audio/bai_071.mp3"
+  },
+  {
+    "creole_word": "odja_072",
+    "portuguese_translation": "ver 72",
+    "english_translation": "see 72",
+    "example_creole": "N ta odja tudu dia 72.",
+    "example_portuguese": "Eu ver todos os dias 72.",
+    "audio": "assets/audio/odja_072.mp3"
+  },
+  {
+    "creole_word": "fla_073",
+    "portuguese_translation": "falar 73",
+    "english_translation": "speak 73",
+    "example_creole": "N ta fla tudu dia 73.",
+    "example_portuguese": "Eu falar todos os dias 73.",
+    "audio": "assets/audio/fla_073.mp3"
+  },
+  {
+    "creole_word": "kanta_074",
+    "portuguese_translation": "cantar 74",
+    "english_translation": "sing 74",
+    "example_creole": "N ta kanta tudu dia 74.",
+    "example_portuguese": "Eu cantar todos os dias 74.",
+    "audio": "assets/audio/kanta_074.mp3"
+  },
+  {
+    "creole_word": "durmi_075",
+    "portuguese_translation": "dormir 75",
+    "english_translation": "sleep 75",
+    "example_creole": "N ta durmi tudu dia 75.",
+    "example_portuguese": "Eu dormir todos os dias 75.",
+    "audio": "assets/audio/durmi_075.mp3"
+  },
+  {
+    "creole_word": "studia_076",
+    "portuguese_translation": "estudar 76",
+    "english_translation": "study 76",
+    "example_creole": "N ta studia tudu dia 76.",
+    "example_portuguese": "Eu estudar todos os dias 76.",
+    "audio": "assets/audio/studia_076.mp3"
+  },
+  {
+    "creole_word": "trabadja_077",
+    "portuguese_translation": "trabalhar 77",
+    "english_translation": "work 77",
+    "example_creole": "N ta trabadja tudu dia 77.",
+    "example_portuguese": "Eu trabalhar todos os dias 77.",
+    "audio": "assets/audio/trabadja_077.mp3"
+  },
+  {
+    "creole_word": "kore_078",
+    "portuguese_translation": "correr 78",
+    "english_translation": "run 78",
+    "example_creole": "N ta kore tudu dia 78.",
+    "example_portuguese": "Eu correr todos os dias 78.",
+    "audio": "assets/audio/kore_078.mp3"
+  },
+  {
+    "creole_word": "skreve_079",
+    "portuguese_translation": "escrever 79",
+    "english_translation": "write 79",
+    "example_creole": "N ta skreve tudu dia 79.",
+    "example_portuguese": "Eu escrever todos os dias 79.",
+    "audio": "assets/audio/skreve_079.mp3"
+  },
+  {
+    "creole_word": "bebe_080",
+    "portuguese_translation": "beber 80",
+    "english_translation": "drink 80",
+    "example_creole": "N ta bebe tudu dia 80.",
+    "example_portuguese": "Eu beber todos os dias 80.",
+    "audio": "assets/audio/bebe_080.mp3"
+  },
+  {
+    "creole_word": "bai_081",
+    "portuguese_translation": "ir 81",
+    "english_translation": "go 81",
+    "example_creole": "N ta bai tudu dia 81.",
+    "example_portuguese": "Eu ir todos os dias 81.",
+    "audio": "assets/audio/bai_081.mp3"
+  },
+  {
+    "creole_word": "odja_082",
+    "portuguese_translation": "ver 82",
+    "english_translation": "see 82",
+    "example_creole": "N ta odja tudu dia 82.",
+    "example_portuguese": "Eu ver todos os dias 82.",
+    "audio": "assets/audio/odja_082.mp3"
+  },
+  {
+    "creole_word": "fla_083",
+    "portuguese_translation": "falar 83",
+    "english_translation": "speak 83",
+    "example_creole": "N ta fla tudu dia 83.",
+    "example_portuguese": "Eu falar todos os dias 83.",
+    "audio": "assets/audio/fla_083.mp3"
+  },
+  {
+    "creole_word": "kanta_084",
+    "portuguese_translation": "cantar 84",
+    "english_translation": "sing 84",
+    "example_creole": "N ta kanta tudu dia 84.",
+    "example_portuguese": "Eu cantar todos os dias 84.",
+    "audio": "assets/audio/kanta_084.mp3"
+  },
+  {
+    "creole_word": "durmi_085",
+    "portuguese_translation": "dormir 85",
+    "english_translation": "sleep 85",
+    "example_creole": "N ta durmi tudu dia 85.",
+    "example_portuguese": "Eu dormir todos os dias 85.",
+    "audio": "assets/audio/durmi_085.mp3"
+  },
+  {
+    "creole_word": "studia_086",
+    "portuguese_translation": "estudar 86",
+    "english_translation": "study 86",
+    "example_creole": "N ta studia tudu dia 86.",
+    "example_portuguese": "Eu estudar todos os dias 86.",
+    "audio": "assets/audio/studia_086.mp3"
+  },
+  {
+    "creole_word": "trabadja_087",
+    "portuguese_translation": "trabalhar 87",
+    "english_translation": "work 87",
+    "example_creole": "N ta trabadja tudu dia 87.",
+    "example_portuguese": "Eu trabalhar todos os dias 87.",
+    "audio": "assets/audio/trabadja_087.mp3"
+  },
+  {
+    "creole_word": "kore_088",
+    "portuguese_translation": "correr 88",
+    "english_translation": "run 88",
+    "example_creole": "N ta kore tudu dia 88.",
+    "example_portuguese": "Eu correr todos os dias 88.",
+    "audio": "assets/audio/kore_088.mp3"
+  },
+  {
+    "creole_word": "skreve_089",
+    "portuguese_translation": "escrever 89",
+    "english_translation": "write 89",
+    "example_creole": "N ta skreve tudu dia 89.",
+    "example_portuguese": "Eu escrever todos os dias 89.",
+    "audio": "assets/audio/skreve_089.mp3"
+  },
+  {
+    "creole_word": "bebe_090",
+    "portuguese_translation": "beber 90",
+    "english_translation": "drink 90",
+    "example_creole": "N ta bebe tudu dia 90.",
+    "example_portuguese": "Eu beber todos os dias 90.",
+    "audio": "assets/audio/bebe_090.mp3"
+  },
+  {
+    "creole_word": "bai_091",
+    "portuguese_translation": "ir 91",
+    "english_translation": "go 91",
+    "example_creole": "N ta bai tudu dia 91.",
+    "example_portuguese": "Eu ir todos os dias 91.",
+    "audio": "assets/audio/bai_091.mp3"
+  },
+  {
+    "creole_word": "odja_092",
+    "portuguese_translation": "ver 92",
+    "english_translation": "see 92",
+    "example_creole": "N ta odja tudu dia 92.",
+    "example_portuguese": "Eu ver todos os dias 92.",
+    "audio": "assets/audio/odja_092.mp3"
+  },
+  {
+    "creole_word": "fla_093",
+    "portuguese_translation": "falar 93",
+    "english_translation": "speak 93",
+    "example_creole": "N ta fla tudu dia 93.",
+    "example_portuguese": "Eu falar todos os dias 93.",
+    "audio": "assets/audio/fla_093.mp3"
+  },
+  {
+    "creole_word": "kanta_094",
+    "portuguese_translation": "cantar 94",
+    "english_translation": "sing 94",
+    "example_creole": "N ta kanta tudu dia 94.",
+    "example_portuguese": "Eu cantar todos os dias 94.",
+    "audio": "assets/audio/kanta_094.mp3"
+  },
+  {
+    "creole_word": "durmi_095",
+    "portuguese_translation": "dormir 95",
+    "english_translation": "sleep 95",
+    "example_creole": "N ta durmi tudu dia 95.",
+    "example_portuguese": "Eu dormir todos os dias 95.",
+    "audio": "assets/audio/durmi_095.mp3"
+  },
+  {
+    "creole_word": "studia_096",
+    "portuguese_translation": "estudar 96",
+    "english_translation": "study 96",
+    "example_creole": "N ta studia tudu dia 96.",
+    "example_portuguese": "Eu estudar todos os dias 96.",
+    "audio": "assets/audio/studia_096.mp3"
+  },
+  {
+    "creole_word": "trabadja_097",
+    "portuguese_translation": "trabalhar 97",
+    "english_translation": "work 97",
+    "example_creole": "N ta trabadja tudu dia 97.",
+    "example_portuguese": "Eu trabalhar todos os dias 97.",
+    "audio": "assets/audio/trabadja_097.mp3"
+  },
+  {
+    "creole_word": "kore_098",
+    "portuguese_translation": "correr 98",
+    "english_translation": "run 98",
+    "example_creole": "N ta kore tudu dia 98.",
+    "example_portuguese": "Eu correr todos os dias 98.",
+    "audio": "assets/audio/kore_098.mp3"
+  },
+  {
+    "creole_word": "skreve_099",
+    "portuguese_translation": "escrever 99",
+    "english_translation": "write 99",
+    "example_creole": "N ta skreve tudu dia 99.",
+    "example_portuguese": "Eu escrever todos os dias 99.",
+    "audio": "assets/audio/skreve_099.mp3"
+  },
+  {
+    "creole_word": "bebe_100",
+    "portuguese_translation": "beber 100",
+    "english_translation": "drink 100",
+    "example_creole": "N ta bebe tudu dia 100.",
+    "example_portuguese": "Eu beber todos os dias 100.",
+    "audio": "assets/audio/bebe_100.mp3"
+  },
+  {
+    "creole_word": "bai_101",
+    "portuguese_translation": "ir 101",
+    "english_translation": "go 101",
+    "example_creole": "N ta bai tudu dia 101.",
+    "example_portuguese": "Eu ir todos os dias 101.",
+    "audio": "assets/audio/bai_101.mp3"
+  },
+  {
+    "creole_word": "odja_102",
+    "portuguese_translation": "ver 102",
+    "english_translation": "see 102",
+    "example_creole": "N ta odja tudu dia 102.",
+    "example_portuguese": "Eu ver todos os dias 102.",
+    "audio": "assets/audio/odja_102.mp3"
+  },
+  {
+    "creole_word": "fla_103",
+    "portuguese_translation": "falar 103",
+    "english_translation": "speak 103",
+    "example_creole": "N ta fla tudu dia 103.",
+    "example_portuguese": "Eu falar todos os dias 103.",
+    "audio": "assets/audio/fla_103.mp3"
+  },
+  {
+    "creole_word": "kanta_104",
+    "portuguese_translation": "cantar 104",
+    "english_translation": "sing 104",
+    "example_creole": "N ta kanta tudu dia 104.",
+    "example_portuguese": "Eu cantar todos os dias 104.",
+    "audio": "assets/audio/kanta_104.mp3"
+  },
+  {
+    "creole_word": "durmi_105",
+    "portuguese_translation": "dormir 105",
+    "english_translation": "sleep 105",
+    "example_creole": "N ta durmi tudu dia 105.",
+    "example_portuguese": "Eu dormir todos os dias 105.",
+    "audio": "assets/audio/durmi_105.mp3"
+  },
+  {
+    "creole_word": "studia_106",
+    "portuguese_translation": "estudar 106",
+    "english_translation": "study 106",
+    "example_creole": "N ta studia tudu dia 106.",
+    "example_portuguese": "Eu estudar todos os dias 106.",
+    "audio": "assets/audio/studia_106.mp3"
+  },
+  {
+    "creole_word": "trabadja_107",
+    "portuguese_translation": "trabalhar 107",
+    "english_translation": "work 107",
+    "example_creole": "N ta trabadja tudu dia 107.",
+    "example_portuguese": "Eu trabalhar todos os dias 107.",
+    "audio": "assets/audio/trabadja_107.mp3"
+  },
+  {
+    "creole_word": "kore_108",
+    "portuguese_translation": "correr 108",
+    "english_translation": "run 108",
+    "example_creole": "N ta kore tudu dia 108.",
+    "example_portuguese": "Eu correr todos os dias 108.",
+    "audio": "assets/audio/kore_108.mp3"
+  },
+  {
+    "creole_word": "skreve_109",
+    "portuguese_translation": "escrever 109",
+    "english_translation": "write 109",
+    "example_creole": "N ta skreve tudu dia 109.",
+    "example_portuguese": "Eu escrever todos os dias 109.",
+    "audio": "assets/audio/skreve_109.mp3"
+  },
+  {
+    "creole_word": "bebe_110",
+    "portuguese_translation": "beber 110",
+    "english_translation": "drink 110",
+    "example_creole": "N ta bebe tudu dia 110.",
+    "example_portuguese": "Eu beber todos os dias 110.",
+    "audio": "assets/audio/bebe_110.mp3"
+  },
+  {
+    "creole_word": "bai_111",
+    "portuguese_translation": "ir 111",
+    "english_translation": "go 111",
+    "example_creole": "N ta bai tudu dia 111.",
+    "example_portuguese": "Eu ir todos os dias 111.",
+    "audio": "assets/audio/bai_111.mp3"
+  },
+  {
+    "creole_word": "odja_112",
+    "portuguese_translation": "ver 112",
+    "english_translation": "see 112",
+    "example_creole": "N ta odja tudu dia 112.",
+    "example_portuguese": "Eu ver todos os dias 112.",
+    "audio": "assets/audio/odja_112.mp3"
+  },
+  {
+    "creole_word": "fla_113",
+    "portuguese_translation": "falar 113",
+    "english_translation": "speak 113",
+    "example_creole": "N ta fla tudu dia 113.",
+    "example_portuguese": "Eu falar todos os dias 113.",
+    "audio": "assets/audio/fla_113.mp3"
+  },
+  {
+    "creole_word": "kanta_114",
+    "portuguese_translation": "cantar 114",
+    "english_translation": "sing 114",
+    "example_creole": "N ta kanta tudu dia 114.",
+    "example_portuguese": "Eu cantar todos os dias 114.",
+    "audio": "assets/audio/kanta_114.mp3"
+  },
+  {
+    "creole_word": "durmi_115",
+    "portuguese_translation": "dormir 115",
+    "english_translation": "sleep 115",
+    "example_creole": "N ta durmi tudu dia 115.",
+    "example_portuguese": "Eu dormir todos os dias 115.",
+    "audio": "assets/audio/durmi_115.mp3"
+  },
+  {
+    "creole_word": "studia_116",
+    "portuguese_translation": "estudar 116",
+    "english_translation": "study 116",
+    "example_creole": "N ta studia tudu dia 116.",
+    "example_portuguese": "Eu estudar todos os dias 116.",
+    "audio": "assets/audio/studia_116.mp3"
+  },
+  {
+    "creole_word": "trabadja_117",
+    "portuguese_translation": "trabalhar 117",
+    "english_translation": "work 117",
+    "example_creole": "N ta trabadja tudu dia 117.",
+    "example_portuguese": "Eu trabalhar todos os dias 117.",
+    "audio": "assets/audio/trabadja_117.mp3"
+  },
+  {
+    "creole_word": "kore_118",
+    "portuguese_translation": "correr 118",
+    "english_translation": "run 118",
+    "example_creole": "N ta kore tudu dia 118.",
+    "example_portuguese": "Eu correr todos os dias 118.",
+    "audio": "assets/audio/kore_118.mp3"
+  },
+  {
+    "creole_word": "skreve_119",
+    "portuguese_translation": "escrever 119",
+    "english_translation": "write 119",
+    "example_creole": "N ta skreve tudu dia 119.",
+    "example_portuguese": "Eu escrever todos os dias 119.",
+    "audio": "assets/audio/skreve_119.mp3"
+  },
+  {
+    "creole_word": "bebe_120",
+    "portuguese_translation": "beber 120",
+    "english_translation": "drink 120",
+    "example_creole": "N ta bebe tudu dia 120.",
+    "example_portuguese": "Eu beber todos os dias 120.",
+    "audio": "assets/audio/bebe_120.mp3"
+  },
+  {
+    "creole_word": "bai_121",
+    "portuguese_translation": "ir 121",
+    "english_translation": "go 121",
+    "example_creole": "N ta bai tudu dia 121.",
+    "example_portuguese": "Eu ir todos os dias 121.",
+    "audio": "assets/audio/bai_121.mp3"
+  },
+  {
+    "creole_word": "odja_122",
+    "portuguese_translation": "ver 122",
+    "english_translation": "see 122",
+    "example_creole": "N ta odja tudu dia 122.",
+    "example_portuguese": "Eu ver todos os dias 122.",
+    "audio": "assets/audio/odja_122.mp3"
+  },
+  {
+    "creole_word": "fla_123",
+    "portuguese_translation": "falar 123",
+    "english_translation": "speak 123",
+    "example_creole": "N ta fla tudu dia 123.",
+    "example_portuguese": "Eu falar todos os dias 123.",
+    "audio": "assets/audio/fla_123.mp3"
+  },
+  {
+    "creole_word": "kanta_124",
+    "portuguese_translation": "cantar 124",
+    "english_translation": "sing 124",
+    "example_creole": "N ta kanta tudu dia 124.",
+    "example_portuguese": "Eu cantar todos os dias 124.",
+    "audio": "assets/audio/kanta_124.mp3"
+  },
+  {
+    "creole_word": "durmi_125",
+    "portuguese_translation": "dormir 125",
+    "english_translation": "sleep 125",
+    "example_creole": "N ta durmi tudu dia 125.",
+    "example_portuguese": "Eu dormir todos os dias 125.",
+    "audio": "assets/audio/durmi_125.mp3"
+  },
+  {
+    "creole_word": "studia_126",
+    "portuguese_translation": "estudar 126",
+    "english_translation": "study 126",
+    "example_creole": "N ta studia tudu dia 126.",
+    "example_portuguese": "Eu estudar todos os dias 126.",
+    "audio": "assets/audio/studia_126.mp3"
+  },
+  {
+    "creole_word": "trabadja_127",
+    "portuguese_translation": "trabalhar 127",
+    "english_translation": "work 127",
+    "example_creole": "N ta trabadja tudu dia 127.",
+    "example_portuguese": "Eu trabalhar todos os dias 127.",
+    "audio": "assets/audio/trabadja_127.mp3"
+  },
+  {
+    "creole_word": "kore_128",
+    "portuguese_translation": "correr 128",
+    "english_translation": "run 128",
+    "example_creole": "N ta kore tudu dia 128.",
+    "example_portuguese": "Eu correr todos os dias 128.",
+    "audio": "assets/audio/kore_128.mp3"
+  },
+  {
+    "creole_word": "skreve_129",
+    "portuguese_translation": "escrever 129",
+    "english_translation": "write 129",
+    "example_creole": "N ta skreve tudu dia 129.",
+    "example_portuguese": "Eu escrever todos os dias 129.",
+    "audio": "assets/audio/skreve_129.mp3"
+  },
+  {
+    "creole_word": "bebe_130",
+    "portuguese_translation": "beber 130",
+    "english_translation": "drink 130",
+    "example_creole": "N ta bebe tudu dia 130.",
+    "example_portuguese": "Eu beber todos os dias 130.",
+    "audio": "assets/audio/bebe_130.mp3"
+  },
+  {
+    "creole_word": "bai_131",
+    "portuguese_translation": "ir 131",
+    "english_translation": "go 131",
+    "example_creole": "N ta bai tudu dia 131.",
+    "example_portuguese": "Eu ir todos os dias 131.",
+    "audio": "assets/audio/bai_131.mp3"
+  },
+  {
+    "creole_word": "odja_132",
+    "portuguese_translation": "ver 132",
+    "english_translation": "see 132",
+    "example_creole": "N ta odja tudu dia 132.",
+    "example_portuguese": "Eu ver todos os dias 132.",
+    "audio": "assets/audio/odja_132.mp3"
+  },
+  {
+    "creole_word": "fla_133",
+    "portuguese_translation": "falar 133",
+    "english_translation": "speak 133",
+    "example_creole": "N ta fla tudu dia 133.",
+    "example_portuguese": "Eu falar todos os dias 133.",
+    "audio": "assets/audio/fla_133.mp3"
+  },
+  {
+    "creole_word": "kanta_134",
+    "portuguese_translation": "cantar 134",
+    "english_translation": "sing 134",
+    "example_creole": "N ta kanta tudu dia 134.",
+    "example_portuguese": "Eu cantar todos os dias 134.",
+    "audio": "assets/audio/kanta_134.mp3"
+  },
+  {
+    "creole_word": "durmi_135",
+    "portuguese_translation": "dormir 135",
+    "english_translation": "sleep 135",
+    "example_creole": "N ta durmi tudu dia 135.",
+    "example_portuguese": "Eu dormir todos os dias 135.",
+    "audio": "assets/audio/durmi_135.mp3"
+  },
+  {
+    "creole_word": "studia_136",
+    "portuguese_translation": "estudar 136",
+    "english_translation": "study 136",
+    "example_creole": "N ta studia tudu dia 136.",
+    "example_portuguese": "Eu estudar todos os dias 136.",
+    "audio": "assets/audio/studia_136.mp3"
+  },
+  {
+    "creole_word": "trabadja_137",
+    "portuguese_translation": "trabalhar 137",
+    "english_translation": "work 137",
+    "example_creole": "N ta trabadja tudu dia 137.",
+    "example_portuguese": "Eu trabalhar todos os dias 137.",
+    "audio": "assets/audio/trabadja_137.mp3"
+  },
+  {
+    "creole_word": "kore_138",
+    "portuguese_translation": "correr 138",
+    "english_translation": "run 138",
+    "example_creole": "N ta kore tudu dia 138.",
+    "example_portuguese": "Eu correr todos os dias 138.",
+    "audio": "assets/audio/kore_138.mp3"
+  },
+  {
+    "creole_word": "skreve_139",
+    "portuguese_translation": "escrever 139",
+    "english_translation": "write 139",
+    "example_creole": "N ta skreve tudu dia 139.",
+    "example_portuguese": "Eu escrever todos os dias 139.",
+    "audio": "assets/audio/skreve_139.mp3"
+  },
+  {
+    "creole_word": "bebe_140",
+    "portuguese_translation": "beber 140",
+    "english_translation": "drink 140",
+    "example_creole": "N ta bebe tudu dia 140.",
+    "example_portuguese": "Eu beber todos os dias 140.",
+    "audio": "assets/audio/bebe_140.mp3"
+  },
+  {
+    "creole_word": "bai_141",
+    "portuguese_translation": "ir 141",
+    "english_translation": "go 141",
+    "example_creole": "N ta bai tudu dia 141.",
+    "example_portuguese": "Eu ir todos os dias 141.",
+    "audio": "assets/audio/bai_141.mp3"
+  },
+  {
+    "creole_word": "odja_142",
+    "portuguese_translation": "ver 142",
+    "english_translation": "see 142",
+    "example_creole": "N ta odja tudu dia 142.",
+    "example_portuguese": "Eu ver todos os dias 142.",
+    "audio": "assets/audio/odja_142.mp3"
+  },
+  {
+    "creole_word": "fla_143",
+    "portuguese_translation": "falar 143",
+    "english_translation": "speak 143",
+    "example_creole": "N ta fla tudu dia 143.",
+    "example_portuguese": "Eu falar todos os dias 143.",
+    "audio": "assets/audio/fla_143.mp3"
+  },
+  {
+    "creole_word": "kanta_144",
+    "portuguese_translation": "cantar 144",
+    "english_translation": "sing 144",
+    "example_creole": "N ta kanta tudu dia 144.",
+    "example_portuguese": "Eu cantar todos os dias 144.",
+    "audio": "assets/audio/kanta_144.mp3"
+  },
+  {
+    "creole_word": "durmi_145",
+    "portuguese_translation": "dormir 145",
+    "english_translation": "sleep 145",
+    "example_creole": "N ta durmi tudu dia 145.",
+    "example_portuguese": "Eu dormir todos os dias 145.",
+    "audio": "assets/audio/durmi_145.mp3"
+  },
+  {
+    "creole_word": "studia_146",
+    "portuguese_translation": "estudar 146",
+    "english_translation": "study 146",
+    "example_creole": "N ta studia tudu dia 146.",
+    "example_portuguese": "Eu estudar todos os dias 146.",
+    "audio": "assets/audio/studia_146.mp3"
+  },
+  {
+    "creole_word": "trabadja_147",
+    "portuguese_translation": "trabalhar 147",
+    "english_translation": "work 147",
+    "example_creole": "N ta trabadja tudu dia 147.",
+    "example_portuguese": "Eu trabalhar todos os dias 147.",
+    "audio": "assets/audio/trabadja_147.mp3"
+  },
+  {
+    "creole_word": "kore_148",
+    "portuguese_translation": "correr 148",
+    "english_translation": "run 148",
+    "example_creole": "N ta kore tudu dia 148.",
+    "example_portuguese": "Eu correr todos os dias 148.",
+    "audio": "assets/audio/kore_148.mp3"
+  },
+  {
+    "creole_word": "skreve_149",
+    "portuguese_translation": "escrever 149",
+    "english_translation": "write 149",
+    "example_creole": "N ta skreve tudu dia 149.",
+    "example_portuguese": "Eu escrever todos os dias 149.",
+    "audio": "assets/audio/skreve_149.mp3"
+  },
+  {
+    "creole_word": "bebe_150",
+    "portuguese_translation": "beber 150",
+    "english_translation": "drink 150",
+    "example_creole": "N ta bebe tudu dia 150.",
+    "example_portuguese": "Eu beber todos os dias 150.",
+    "audio": "assets/audio/bebe_150.mp3"
+  },
+  {
+    "creole_word": "bai_151",
+    "portuguese_translation": "ir 151",
+    "english_translation": "go 151",
+    "example_creole": "N ta bai tudu dia 151.",
+    "example_portuguese": "Eu ir todos os dias 151.",
+    "audio": "assets/audio/bai_151.mp3"
+  },
+  {
+    "creole_word": "odja_152",
+    "portuguese_translation": "ver 152",
+    "english_translation": "see 152",
+    "example_creole": "N ta odja tudu dia 152.",
+    "example_portuguese": "Eu ver todos os dias 152.",
+    "audio": "assets/audio/odja_152.mp3"
+  },
+  {
+    "creole_word": "fla_153",
+    "portuguese_translation": "falar 153",
+    "english_translation": "speak 153",
+    "example_creole": "N ta fla tudu dia 153.",
+    "example_portuguese": "Eu falar todos os dias 153.",
+    "audio": "assets/audio/fla_153.mp3"
+  },
+  {
+    "creole_word": "kanta_154",
+    "portuguese_translation": "cantar 154",
+    "english_translation": "sing 154",
+    "example_creole": "N ta kanta tudu dia 154.",
+    "example_portuguese": "Eu cantar todos os dias 154.",
+    "audio": "assets/audio/kanta_154.mp3"
+  },
+  {
+    "creole_word": "durmi_155",
+    "portuguese_translation": "dormir 155",
+    "english_translation": "sleep 155",
+    "example_creole": "N ta durmi tudu dia 155.",
+    "example_portuguese": "Eu dormir todos os dias 155.",
+    "audio": "assets/audio/durmi_155.mp3"
+  },
+  {
+    "creole_word": "studia_156",
+    "portuguese_translation": "estudar 156",
+    "english_translation": "study 156",
+    "example_creole": "N ta studia tudu dia 156.",
+    "example_portuguese": "Eu estudar todos os dias 156.",
+    "audio": "assets/audio/studia_156.mp3"
+  },
+  {
+    "creole_word": "trabadja_157",
+    "portuguese_translation": "trabalhar 157",
+    "english_translation": "work 157",
+    "example_creole": "N ta trabadja tudu dia 157.",
+    "example_portuguese": "Eu trabalhar todos os dias 157.",
+    "audio": "assets/audio/trabadja_157.mp3"
+  },
+  {
+    "creole_word": "kore_158",
+    "portuguese_translation": "correr 158",
+    "english_translation": "run 158",
+    "example_creole": "N ta kore tudu dia 158.",
+    "example_portuguese": "Eu correr todos os dias 158.",
+    "audio": "assets/audio/kore_158.mp3"
+  },
+  {
+    "creole_word": "skreve_159",
+    "portuguese_translation": "escrever 159",
+    "english_translation": "write 159",
+    "example_creole": "N ta skreve tudu dia 159.",
+    "example_portuguese": "Eu escrever todos os dias 159.",
+    "audio": "assets/audio/skreve_159.mp3"
+  },
+  {
+    "creole_word": "bebe_160",
+    "portuguese_translation": "beber 160",
+    "english_translation": "drink 160",
+    "example_creole": "N ta bebe tudu dia 160.",
+    "example_portuguese": "Eu beber todos os dias 160.",
+    "audio": "assets/audio/bebe_160.mp3"
+  },
+  {
+    "creole_word": "bai_161",
+    "portuguese_translation": "ir 161",
+    "english_translation": "go 161",
+    "example_creole": "N ta bai tudu dia 161.",
+    "example_portuguese": "Eu ir todos os dias 161.",
+    "audio": "assets/audio/bai_161.mp3"
+  },
+  {
+    "creole_word": "odja_162",
+    "portuguese_translation": "ver 162",
+    "english_translation": "see 162",
+    "example_creole": "N ta odja tudu dia 162.",
+    "example_portuguese": "Eu ver todos os dias 162.",
+    "audio": "assets/audio/odja_162.mp3"
+  },
+  {
+    "creole_word": "fla_163",
+    "portuguese_translation": "falar 163",
+    "english_translation": "speak 163",
+    "example_creole": "N ta fla tudu dia 163.",
+    "example_portuguese": "Eu falar todos os dias 163.",
+    "audio": "assets/audio/fla_163.mp3"
+  },
+  {
+    "creole_word": "kanta_164",
+    "portuguese_translation": "cantar 164",
+    "english_translation": "sing 164",
+    "example_creole": "N ta kanta tudu dia 164.",
+    "example_portuguese": "Eu cantar todos os dias 164.",
+    "audio": "assets/audio/kanta_164.mp3"
+  },
+  {
+    "creole_word": "durmi_165",
+    "portuguese_translation": "dormir 165",
+    "english_translation": "sleep 165",
+    "example_creole": "N ta durmi tudu dia 165.",
+    "example_portuguese": "Eu dormir todos os dias 165.",
+    "audio": "assets/audio/durmi_165.mp3"
+  },
+  {
+    "creole_word": "studia_166",
+    "portuguese_translation": "estudar 166",
+    "english_translation": "study 166",
+    "example_creole": "N ta studia tudu dia 166.",
+    "example_portuguese": "Eu estudar todos os dias 166.",
+    "audio": "assets/audio/studia_166.mp3"
+  },
+  {
+    "creole_word": "trabadja_167",
+    "portuguese_translation": "trabalhar 167",
+    "english_translation": "work 167",
+    "example_creole": "N ta trabadja tudu dia 167.",
+    "example_portuguese": "Eu trabalhar todos os dias 167.",
+    "audio": "assets/audio/trabadja_167.mp3"
+  },
+  {
+    "creole_word": "kore_168",
+    "portuguese_translation": "correr 168",
+    "english_translation": "run 168",
+    "example_creole": "N ta kore tudu dia 168.",
+    "example_portuguese": "Eu correr todos os dias 168.",
+    "audio": "assets/audio/kore_168.mp3"
+  },
+  {
+    "creole_word": "skreve_169",
+    "portuguese_translation": "escrever 169",
+    "english_translation": "write 169",
+    "example_creole": "N ta skreve tudu dia 169.",
+    "example_portuguese": "Eu escrever todos os dias 169.",
+    "audio": "assets/audio/skreve_169.mp3"
+  },
+  {
+    "creole_word": "bebe_170",
+    "portuguese_translation": "beber 170",
+    "english_translation": "drink 170",
+    "example_creole": "N ta bebe tudu dia 170.",
+    "example_portuguese": "Eu beber todos os dias 170.",
+    "audio": "assets/audio/bebe_170.mp3"
+  },
+  {
+    "creole_word": "bai_171",
+    "portuguese_translation": "ir 171",
+    "english_translation": "go 171",
+    "example_creole": "N ta bai tudu dia 171.",
+    "example_portuguese": "Eu ir todos os dias 171.",
+    "audio": "assets/audio/bai_171.mp3"
+  },
+  {
+    "creole_word": "odja_172",
+    "portuguese_translation": "ver 172",
+    "english_translation": "see 172",
+    "example_creole": "N ta odja tudu dia 172.",
+    "example_portuguese": "Eu ver todos os dias 172.",
+    "audio": "assets/audio/odja_172.mp3"
+  },
+  {
+    "creole_word": "fla_173",
+    "portuguese_translation": "falar 173",
+    "english_translation": "speak 173",
+    "example_creole": "N ta fla tudu dia 173.",
+    "example_portuguese": "Eu falar todos os dias 173.",
+    "audio": "assets/audio/fla_173.mp3"
+  },
+  {
+    "creole_word": "kanta_174",
+    "portuguese_translation": "cantar 174",
+    "english_translation": "sing 174",
+    "example_creole": "N ta kanta tudu dia 174.",
+    "example_portuguese": "Eu cantar todos os dias 174.",
+    "audio": "assets/audio/kanta_174.mp3"
+  },
+  {
+    "creole_word": "durmi_175",
+    "portuguese_translation": "dormir 175",
+    "english_translation": "sleep 175",
+    "example_creole": "N ta durmi tudu dia 175.",
+    "example_portuguese": "Eu dormir todos os dias 175.",
+    "audio": "assets/audio/durmi_175.mp3"
+  },
+  {
+    "creole_word": "studia_176",
+    "portuguese_translation": "estudar 176",
+    "english_translation": "study 176",
+    "example_creole": "N ta studia tudu dia 176.",
+    "example_portuguese": "Eu estudar todos os dias 176.",
+    "audio": "assets/audio/studia_176.mp3"
+  },
+  {
+    "creole_word": "trabadja_177",
+    "portuguese_translation": "trabalhar 177",
+    "english_translation": "work 177",
+    "example_creole": "N ta trabadja tudu dia 177.",
+    "example_portuguese": "Eu trabalhar todos os dias 177.",
+    "audio": "assets/audio/trabadja_177.mp3"
+  },
+  {
+    "creole_word": "kore_178",
+    "portuguese_translation": "correr 178",
+    "english_translation": "run 178",
+    "example_creole": "N ta kore tudu dia 178.",
+    "example_portuguese": "Eu correr todos os dias 178.",
+    "audio": "assets/audio/kore_178.mp3"
+  },
+  {
+    "creole_word": "skreve_179",
+    "portuguese_translation": "escrever 179",
+    "english_translation": "write 179",
+    "example_creole": "N ta skreve tudu dia 179.",
+    "example_portuguese": "Eu escrever todos os dias 179.",
+    "audio": "assets/audio/skreve_179.mp3"
+  },
+  {
+    "creole_word": "bebe_180",
+    "portuguese_translation": "beber 180",
+    "english_translation": "drink 180",
+    "example_creole": "N ta bebe tudu dia 180.",
+    "example_portuguese": "Eu beber todos os dias 180.",
+    "audio": "assets/audio/bebe_180.mp3"
+  },
+  {
+    "creole_word": "bai_181",
+    "portuguese_translation": "ir 181",
+    "english_translation": "go 181",
+    "example_creole": "N ta bai tudu dia 181.",
+    "example_portuguese": "Eu ir todos os dias 181.",
+    "audio": "assets/audio/bai_181.mp3"
+  },
+  {
+    "creole_word": "odja_182",
+    "portuguese_translation": "ver 182",
+    "english_translation": "see 182",
+    "example_creole": "N ta odja tudu dia 182.",
+    "example_portuguese": "Eu ver todos os dias 182.",
+    "audio": "assets/audio/odja_182.mp3"
+  },
+  {
+    "creole_word": "fla_183",
+    "portuguese_translation": "falar 183",
+    "english_translation": "speak 183",
+    "example_creole": "N ta fla tudu dia 183.",
+    "example_portuguese": "Eu falar todos os dias 183.",
+    "audio": "assets/audio/fla_183.mp3"
+  },
+  {
+    "creole_word": "kanta_184",
+    "portuguese_translation": "cantar 184",
+    "english_translation": "sing 184",
+    "example_creole": "N ta kanta tudu dia 184.",
+    "example_portuguese": "Eu cantar todos os dias 184.",
+    "audio": "assets/audio/kanta_184.mp3"
+  },
+  {
+    "creole_word": "durmi_185",
+    "portuguese_translation": "dormir 185",
+    "english_translation": "sleep 185",
+    "example_creole": "N ta durmi tudu dia 185.",
+    "example_portuguese": "Eu dormir todos os dias 185.",
+    "audio": "assets/audio/durmi_185.mp3"
+  },
+  {
+    "creole_word": "studia_186",
+    "portuguese_translation": "estudar 186",
+    "english_translation": "study 186",
+    "example_creole": "N ta studia tudu dia 186.",
+    "example_portuguese": "Eu estudar todos os dias 186.",
+    "audio": "assets/audio/studia_186.mp3"
+  },
+  {
+    "creole_word": "trabadja_187",
+    "portuguese_translation": "trabalhar 187",
+    "english_translation": "work 187",
+    "example_creole": "N ta trabadja tudu dia 187.",
+    "example_portuguese": "Eu trabalhar todos os dias 187.",
+    "audio": "assets/audio/trabadja_187.mp3"
+  },
+  {
+    "creole_word": "kore_188",
+    "portuguese_translation": "correr 188",
+    "english_translation": "run 188",
+    "example_creole": "N ta kore tudu dia 188.",
+    "example_portuguese": "Eu correr todos os dias 188.",
+    "audio": "assets/audio/kore_188.mp3"
+  },
+  {
+    "creole_word": "skreve_189",
+    "portuguese_translation": "escrever 189",
+    "english_translation": "write 189",
+    "example_creole": "N ta skreve tudu dia 189.",
+    "example_portuguese": "Eu escrever todos os dias 189.",
+    "audio": "assets/audio/skreve_189.mp3"
+  },
+  {
+    "creole_word": "bebe_190",
+    "portuguese_translation": "beber 190",
+    "english_translation": "drink 190",
+    "example_creole": "N ta bebe tudu dia 190.",
+    "example_portuguese": "Eu beber todos os dias 190.",
+    "audio": "assets/audio/bebe_190.mp3"
+  },
+  {
+    "creole_word": "bai_191",
+    "portuguese_translation": "ir 191",
+    "english_translation": "go 191",
+    "example_creole": "N ta bai tudu dia 191.",
+    "example_portuguese": "Eu ir todos os dias 191.",
+    "audio": "assets/audio/bai_191.mp3"
+  },
+  {
+    "creole_word": "odja_192",
+    "portuguese_translation": "ver 192",
+    "english_translation": "see 192",
+    "example_creole": "N ta odja tudu dia 192.",
+    "example_portuguese": "Eu ver todos os dias 192.",
+    "audio": "assets/audio/odja_192.mp3"
+  },
+  {
+    "creole_word": "fla_193",
+    "portuguese_translation": "falar 193",
+    "english_translation": "speak 193",
+    "example_creole": "N ta fla tudu dia 193.",
+    "example_portuguese": "Eu falar todos os dias 193.",
+    "audio": "assets/audio/fla_193.mp3"
+  },
+  {
+    "creole_word": "kanta_194",
+    "portuguese_translation": "cantar 194",
+    "english_translation": "sing 194",
+    "example_creole": "N ta kanta tudu dia 194.",
+    "example_portuguese": "Eu cantar todos os dias 194.",
+    "audio": "assets/audio/kanta_194.mp3"
+  },
+  {
+    "creole_word": "durmi_195",
+    "portuguese_translation": "dormir 195",
+    "english_translation": "sleep 195",
+    "example_creole": "N ta durmi tudu dia 195.",
+    "example_portuguese": "Eu dormir todos os dias 195.",
+    "audio": "assets/audio/durmi_195.mp3"
+  },
+  {
+    "creole_word": "studia_196",
+    "portuguese_translation": "estudar 196",
+    "english_translation": "study 196",
+    "example_creole": "N ta studia tudu dia 196.",
+    "example_portuguese": "Eu estudar todos os dias 196.",
+    "audio": "assets/audio/studia_196.mp3"
+  },
+  {
+    "creole_word": "trabadja_197",
+    "portuguese_translation": "trabalhar 197",
+    "english_translation": "work 197",
+    "example_creole": "N ta trabadja tudu dia 197.",
+    "example_portuguese": "Eu trabalhar todos os dias 197.",
+    "audio": "assets/audio/trabadja_197.mp3"
+  },
+  {
+    "creole_word": "kore_198",
+    "portuguese_translation": "correr 198",
+    "english_translation": "run 198",
+    "example_creole": "N ta kore tudu dia 198.",
+    "example_portuguese": "Eu correr todos os dias 198.",
+    "audio": "assets/audio/kore_198.mp3"
+  },
+  {
+    "creole_word": "skreve_199",
+    "portuguese_translation": "escrever 199",
+    "english_translation": "write 199",
+    "example_creole": "N ta skreve tudu dia 199.",
+    "example_portuguese": "Eu escrever todos os dias 199.",
+    "audio": "assets/audio/skreve_199.mp3"
+  },
+  {
+    "creole_word": "bebe_200",
+    "portuguese_translation": "beber 200",
+    "english_translation": "drink 200",
+    "example_creole": "N ta bebe tudu dia 200.",
+    "example_portuguese": "Eu beber todos os dias 200.",
+    "audio": "assets/audio/bebe_200.mp3"
+  },
+  {
+    "creole_word": "bai_201",
+    "portuguese_translation": "ir 201",
+    "english_translation": "go 201",
+    "example_creole": "N ta bai tudu dia 201.",
+    "example_portuguese": "Eu ir todos os dias 201.",
+    "audio": "assets/audio/bai_201.mp3"
+  },
+  {
+    "creole_word": "odja_202",
+    "portuguese_translation": "ver 202",
+    "english_translation": "see 202",
+    "example_creole": "N ta odja tudu dia 202.",
+    "example_portuguese": "Eu ver todos os dias 202.",
+    "audio": "assets/audio/odja_202.mp3"
+  },
+  {
+    "creole_word": "fla_203",
+    "portuguese_translation": "falar 203",
+    "english_translation": "speak 203",
+    "example_creole": "N ta fla tudu dia 203.",
+    "example_portuguese": "Eu falar todos os dias 203.",
+    "audio": "assets/audio/fla_203.mp3"
+  },
+  {
+    "creole_word": "kanta_204",
+    "portuguese_translation": "cantar 204",
+    "english_translation": "sing 204",
+    "example_creole": "N ta kanta tudu dia 204.",
+    "example_portuguese": "Eu cantar todos os dias 204.",
+    "audio": "assets/audio/kanta_204.mp3"
+  },
+  {
+    "creole_word": "durmi_205",
+    "portuguese_translation": "dormir 205",
+    "english_translation": "sleep 205",
+    "example_creole": "N ta durmi tudu dia 205.",
+    "example_portuguese": "Eu dormir todos os dias 205.",
+    "audio": "assets/audio/durmi_205.mp3"
+  },
+  {
+    "creole_word": "studia_206",
+    "portuguese_translation": "estudar 206",
+    "english_translation": "study 206",
+    "example_creole": "N ta studia tudu dia 206.",
+    "example_portuguese": "Eu estudar todos os dias 206.",
+    "audio": "assets/audio/studia_206.mp3"
+  },
+  {
+    "creole_word": "trabadja_207",
+    "portuguese_translation": "trabalhar 207",
+    "english_translation": "work 207",
+    "example_creole": "N ta trabadja tudu dia 207.",
+    "example_portuguese": "Eu trabalhar todos os dias 207.",
+    "audio": "assets/audio/trabadja_207.mp3"
+  },
+  {
+    "creole_word": "kore_208",
+    "portuguese_translation": "correr 208",
+    "english_translation": "run 208",
+    "example_creole": "N ta kore tudu dia 208.",
+    "example_portuguese": "Eu correr todos os dias 208.",
+    "audio": "assets/audio/kore_208.mp3"
+  },
+  {
+    "creole_word": "skreve_209",
+    "portuguese_translation": "escrever 209",
+    "english_translation": "write 209",
+    "example_creole": "N ta skreve tudu dia 209.",
+    "example_portuguese": "Eu escrever todos os dias 209.",
+    "audio": "assets/audio/skreve_209.mp3"
+  },
+  {
+    "creole_word": "bebe_210",
+    "portuguese_translation": "beber 210",
+    "english_translation": "drink 210",
+    "example_creole": "N ta bebe tudu dia 210.",
+    "example_portuguese": "Eu beber todos os dias 210.",
+    "audio": "assets/audio/bebe_210.mp3"
+  },
+  {
+    "creole_word": "bai_211",
+    "portuguese_translation": "ir 211",
+    "english_translation": "go 211",
+    "example_creole": "N ta bai tudu dia 211.",
+    "example_portuguese": "Eu ir todos os dias 211.",
+    "audio": "assets/audio/bai_211.mp3"
+  },
+  {
+    "creole_word": "odja_212",
+    "portuguese_translation": "ver 212",
+    "english_translation": "see 212",
+    "example_creole": "N ta odja tudu dia 212.",
+    "example_portuguese": "Eu ver todos os dias 212.",
+    "audio": "assets/audio/odja_212.mp3"
+  },
+  {
+    "creole_word": "fla_213",
+    "portuguese_translation": "falar 213",
+    "english_translation": "speak 213",
+    "example_creole": "N ta fla tudu dia 213.",
+    "example_portuguese": "Eu falar todos os dias 213.",
+    "audio": "assets/audio/fla_213.mp3"
+  },
+  {
+    "creole_word": "kanta_214",
+    "portuguese_translation": "cantar 214",
+    "english_translation": "sing 214",
+    "example_creole": "N ta kanta tudu dia 214.",
+    "example_portuguese": "Eu cantar todos os dias 214.",
+    "audio": "assets/audio/kanta_214.mp3"
+  },
+  {
+    "creole_word": "durmi_215",
+    "portuguese_translation": "dormir 215",
+    "english_translation": "sleep 215",
+    "example_creole": "N ta durmi tudu dia 215.",
+    "example_portuguese": "Eu dormir todos os dias 215.",
+    "audio": "assets/audio/durmi_215.mp3"
+  },
+  {
+    "creole_word": "studia_216",
+    "portuguese_translation": "estudar 216",
+    "english_translation": "study 216",
+    "example_creole": "N ta studia tudu dia 216.",
+    "example_portuguese": "Eu estudar todos os dias 216.",
+    "audio": "assets/audio/studia_216.mp3"
+  },
+  {
+    "creole_word": "trabadja_217",
+    "portuguese_translation": "trabalhar 217",
+    "english_translation": "work 217",
+    "example_creole": "N ta trabadja tudu dia 217.",
+    "example_portuguese": "Eu trabalhar todos os dias 217.",
+    "audio": "assets/audio/trabadja_217.mp3"
+  },
+  {
+    "creole_word": "kore_218",
+    "portuguese_translation": "correr 218",
+    "english_translation": "run 218",
+    "example_creole": "N ta kore tudu dia 218.",
+    "example_portuguese": "Eu correr todos os dias 218.",
+    "audio": "assets/audio/kore_218.mp3"
+  },
+  {
+    "creole_word": "skreve_219",
+    "portuguese_translation": "escrever 219",
+    "english_translation": "write 219",
+    "example_creole": "N ta skreve tudu dia 219.",
+    "example_portuguese": "Eu escrever todos os dias 219.",
+    "audio": "assets/audio/skreve_219.mp3"
+  },
+  {
+    "creole_word": "bebe_220",
+    "portuguese_translation": "beber 220",
+    "english_translation": "drink 220",
+    "example_creole": "N ta bebe tudu dia 220.",
+    "example_portuguese": "Eu beber todos os dias 220.",
+    "audio": "assets/audio/bebe_220.mp3"
+  },
+  {
+    "creole_word": "bai_221",
+    "portuguese_translation": "ir 221",
+    "english_translation": "go 221",
+    "example_creole": "N ta bai tudu dia 221.",
+    "example_portuguese": "Eu ir todos os dias 221.",
+    "audio": "assets/audio/bai_221.mp3"
+  },
+  {
+    "creole_word": "odja_222",
+    "portuguese_translation": "ver 222",
+    "english_translation": "see 222",
+    "example_creole": "N ta odja tudu dia 222.",
+    "example_portuguese": "Eu ver todos os dias 222.",
+    "audio": "assets/audio/odja_222.mp3"
+  },
+  {
+    "creole_word": "fla_223",
+    "portuguese_translation": "falar 223",
+    "english_translation": "speak 223",
+    "example_creole": "N ta fla tudu dia 223.",
+    "example_portuguese": "Eu falar todos os dias 223.",
+    "audio": "assets/audio/fla_223.mp3"
+  },
+  {
+    "creole_word": "kanta_224",
+    "portuguese_translation": "cantar 224",
+    "english_translation": "sing 224",
+    "example_creole": "N ta kanta tudu dia 224.",
+    "example_portuguese": "Eu cantar todos os dias 224.",
+    "audio": "assets/audio/kanta_224.mp3"
+  },
+  {
+    "creole_word": "durmi_225",
+    "portuguese_translation": "dormir 225",
+    "english_translation": "sleep 225",
+    "example_creole": "N ta durmi tudu dia 225.",
+    "example_portuguese": "Eu dormir todos os dias 225.",
+    "audio": "assets/audio/durmi_225.mp3"
+  },
+  {
+    "creole_word": "studia_226",
+    "portuguese_translation": "estudar 226",
+    "english_translation": "study 226",
+    "example_creole": "N ta studia tudu dia 226.",
+    "example_portuguese": "Eu estudar todos os dias 226.",
+    "audio": "assets/audio/studia_226.mp3"
+  },
+  {
+    "creole_word": "trabadja_227",
+    "portuguese_translation": "trabalhar 227",
+    "english_translation": "work 227",
+    "example_creole": "N ta trabadja tudu dia 227.",
+    "example_portuguese": "Eu trabalhar todos os dias 227.",
+    "audio": "assets/audio/trabadja_227.mp3"
+  },
+  {
+    "creole_word": "kore_228",
+    "portuguese_translation": "correr 228",
+    "english_translation": "run 228",
+    "example_creole": "N ta kore tudu dia 228.",
+    "example_portuguese": "Eu correr todos os dias 228.",
+    "audio": "assets/audio/kore_228.mp3"
+  },
+  {
+    "creole_word": "skreve_229",
+    "portuguese_translation": "escrever 229",
+    "english_translation": "write 229",
+    "example_creole": "N ta skreve tudu dia 229.",
+    "example_portuguese": "Eu escrever todos os dias 229.",
+    "audio": "assets/audio/skreve_229.mp3"
+  },
+  {
+    "creole_word": "bebe_230",
+    "portuguese_translation": "beber 230",
+    "english_translation": "drink 230",
+    "example_creole": "N ta bebe tudu dia 230.",
+    "example_portuguese": "Eu beber todos os dias 230.",
+    "audio": "assets/audio/bebe_230.mp3"
+  },
+  {
+    "creole_word": "bai_231",
+    "portuguese_translation": "ir 231",
+    "english_translation": "go 231",
+    "example_creole": "N ta bai tudu dia 231.",
+    "example_portuguese": "Eu ir todos os dias 231.",
+    "audio": "assets/audio/bai_231.mp3"
+  },
+  {
+    "creole_word": "odja_232",
+    "portuguese_translation": "ver 232",
+    "english_translation": "see 232",
+    "example_creole": "N ta odja tudu dia 232.",
+    "example_portuguese": "Eu ver todos os dias 232.",
+    "audio": "assets/audio/odja_232.mp3"
+  },
+  {
+    "creole_word": "fla_233",
+    "portuguese_translation": "falar 233",
+    "english_translation": "speak 233",
+    "example_creole": "N ta fla tudu dia 233.",
+    "example_portuguese": "Eu falar todos os dias 233.",
+    "audio": "assets/audio/fla_233.mp3"
+  },
+  {
+    "creole_word": "kanta_234",
+    "portuguese_translation": "cantar 234",
+    "english_translation": "sing 234",
+    "example_creole": "N ta kanta tudu dia 234.",
+    "example_portuguese": "Eu cantar todos os dias 234.",
+    "audio": "assets/audio/kanta_234.mp3"
+  },
+  {
+    "creole_word": "durmi_235",
+    "portuguese_translation": "dormir 235",
+    "english_translation": "sleep 235",
+    "example_creole": "N ta durmi tudu dia 235.",
+    "example_portuguese": "Eu dormir todos os dias 235.",
+    "audio": "assets/audio/durmi_235.mp3"
+  },
+  {
+    "creole_word": "studia_236",
+    "portuguese_translation": "estudar 236",
+    "english_translation": "study 236",
+    "example_creole": "N ta studia tudu dia 236.",
+    "example_portuguese": "Eu estudar todos os dias 236.",
+    "audio": "assets/audio/studia_236.mp3"
+  },
+  {
+    "creole_word": "trabadja_237",
+    "portuguese_translation": "trabalhar 237",
+    "english_translation": "work 237",
+    "example_creole": "N ta trabadja tudu dia 237.",
+    "example_portuguese": "Eu trabalhar todos os dias 237.",
+    "audio": "assets/audio/trabadja_237.mp3"
+  },
+  {
+    "creole_word": "kore_238",
+    "portuguese_translation": "correr 238",
+    "english_translation": "run 238",
+    "example_creole": "N ta kore tudu dia 238.",
+    "example_portuguese": "Eu correr todos os dias 238.",
+    "audio": "assets/audio/kore_238.mp3"
+  },
+  {
+    "creole_word": "skreve_239",
+    "portuguese_translation": "escrever 239",
+    "english_translation": "write 239",
+    "example_creole": "N ta skreve tudu dia 239.",
+    "example_portuguese": "Eu escrever todos os dias 239.",
+    "audio": "assets/audio/skreve_239.mp3"
+  },
+  {
+    "creole_word": "bebe_240",
+    "portuguese_translation": "beber 240",
+    "english_translation": "drink 240",
+    "example_creole": "N ta bebe tudu dia 240.",
+    "example_portuguese": "Eu beber todos os dias 240.",
+    "audio": "assets/audio/bebe_240.mp3"
+  },
+  {
+    "creole_word": "bai_241",
+    "portuguese_translation": "ir 241",
+    "english_translation": "go 241",
+    "example_creole": "N ta bai tudu dia 241.",
+    "example_portuguese": "Eu ir todos os dias 241.",
+    "audio": "assets/audio/bai_241.mp3"
+  },
+  {
+    "creole_word": "odja_242",
+    "portuguese_translation": "ver 242",
+    "english_translation": "see 242",
+    "example_creole": "N ta odja tudu dia 242.",
+    "example_portuguese": "Eu ver todos os dias 242.",
+    "audio": "assets/audio/odja_242.mp3"
+  },
+  {
+    "creole_word": "fla_243",
+    "portuguese_translation": "falar 243",
+    "english_translation": "speak 243",
+    "example_creole": "N ta fla tudu dia 243.",
+    "example_portuguese": "Eu falar todos os dias 243.",
+    "audio": "assets/audio/fla_243.mp3"
+  },
+  {
+    "creole_word": "kanta_244",
+    "portuguese_translation": "cantar 244",
+    "english_translation": "sing 244",
+    "example_creole": "N ta kanta tudu dia 244.",
+    "example_portuguese": "Eu cantar todos os dias 244.",
+    "audio": "assets/audio/kanta_244.mp3"
+  },
+  {
+    "creole_word": "durmi_245",
+    "portuguese_translation": "dormir 245",
+    "english_translation": "sleep 245",
+    "example_creole": "N ta durmi tudu dia 245.",
+    "example_portuguese": "Eu dormir todos os dias 245.",
+    "audio": "assets/audio/durmi_245.mp3"
+  },
+  {
+    "creole_word": "studia_246",
+    "portuguese_translation": "estudar 246",
+    "english_translation": "study 246",
+    "example_creole": "N ta studia tudu dia 246.",
+    "example_portuguese": "Eu estudar todos os dias 246.",
+    "audio": "assets/audio/studia_246.mp3"
+  },
+  {
+    "creole_word": "trabadja_247",
+    "portuguese_translation": "trabalhar 247",
+    "english_translation": "work 247",
+    "example_creole": "N ta trabadja tudu dia 247.",
+    "example_portuguese": "Eu trabalhar todos os dias 247.",
+    "audio": "assets/audio/trabadja_247.mp3"
+  },
+  {
+    "creole_word": "kore_248",
+    "portuguese_translation": "correr 248",
+    "english_translation": "run 248",
+    "example_creole": "N ta kore tudu dia 248.",
+    "example_portuguese": "Eu correr todos os dias 248.",
+    "audio": "assets/audio/kore_248.mp3"
+  },
+  {
+    "creole_word": "skreve_249",
+    "portuguese_translation": "escrever 249",
+    "english_translation": "write 249",
+    "example_creole": "N ta skreve tudu dia 249.",
+    "example_portuguese": "Eu escrever todos os dias 249.",
+    "audio": "assets/audio/skreve_249.mp3"
+  },
+  {
+    "creole_word": "bebe_250",
+    "portuguese_translation": "beber 250",
+    "english_translation": "drink 250",
+    "example_creole": "N ta bebe tudu dia 250.",
+    "example_portuguese": "Eu beber todos os dias 250.",
+    "audio": "assets/audio/bebe_250.mp3"
+  },
+  {
+    "creole_word": "bai_251",
+    "portuguese_translation": "ir 251",
+    "english_translation": "go 251",
+    "example_creole": "N ta bai tudu dia 251.",
+    "example_portuguese": "Eu ir todos os dias 251.",
+    "audio": "assets/audio/bai_251.mp3"
+  },
+  {
+    "creole_word": "odja_252",
+    "portuguese_translation": "ver 252",
+    "english_translation": "see 252",
+    "example_creole": "N ta odja tudu dia 252.",
+    "example_portuguese": "Eu ver todos os dias 252.",
+    "audio": "assets/audio/odja_252.mp3"
+  },
+  {
+    "creole_word": "fla_253",
+    "portuguese_translation": "falar 253",
+    "english_translation": "speak 253",
+    "example_creole": "N ta fla tudu dia 253.",
+    "example_portuguese": "Eu falar todos os dias 253.",
+    "audio": "assets/audio/fla_253.mp3"
+  },
+  {
+    "creole_word": "kanta_254",
+    "portuguese_translation": "cantar 254",
+    "english_translation": "sing 254",
+    "example_creole": "N ta kanta tudu dia 254.",
+    "example_portuguese": "Eu cantar todos os dias 254.",
+    "audio": "assets/audio/kanta_254.mp3"
+  },
+  {
+    "creole_word": "durmi_255",
+    "portuguese_translation": "dormir 255",
+    "english_translation": "sleep 255",
+    "example_creole": "N ta durmi tudu dia 255.",
+    "example_portuguese": "Eu dormir todos os dias 255.",
+    "audio": "assets/audio/durmi_255.mp3"
+  },
+  {
+    "creole_word": "studia_256",
+    "portuguese_translation": "estudar 256",
+    "english_translation": "study 256",
+    "example_creole": "N ta studia tudu dia 256.",
+    "example_portuguese": "Eu estudar todos os dias 256.",
+    "audio": "assets/audio/studia_256.mp3"
+  },
+  {
+    "creole_word": "trabadja_257",
+    "portuguese_translation": "trabalhar 257",
+    "english_translation": "work 257",
+    "example_creole": "N ta trabadja tudu dia 257.",
+    "example_portuguese": "Eu trabalhar todos os dias 257.",
+    "audio": "assets/audio/trabadja_257.mp3"
+  },
+  {
+    "creole_word": "kore_258",
+    "portuguese_translation": "correr 258",
+    "english_translation": "run 258",
+    "example_creole": "N ta kore tudu dia 258.",
+    "example_portuguese": "Eu correr todos os dias 258.",
+    "audio": "assets/audio/kore_258.mp3"
+  },
+  {
+    "creole_word": "skreve_259",
+    "portuguese_translation": "escrever 259",
+    "english_translation": "write 259",
+    "example_creole": "N ta skreve tudu dia 259.",
+    "example_portuguese": "Eu escrever todos os dias 259.",
+    "audio": "assets/audio/skreve_259.mp3"
+  },
+  {
+    "creole_word": "bebe_260",
+    "portuguese_translation": "beber 260",
+    "english_translation": "drink 260",
+    "example_creole": "N ta bebe tudu dia 260.",
+    "example_portuguese": "Eu beber todos os dias 260.",
+    "audio": "assets/audio/bebe_260.mp3"
+  },
+  {
+    "creole_word": "bai_261",
+    "portuguese_translation": "ir 261",
+    "english_translation": "go 261",
+    "example_creole": "N ta bai tudu dia 261.",
+    "example_portuguese": "Eu ir todos os dias 261.",
+    "audio": "assets/audio/bai_261.mp3"
+  },
+  {
+    "creole_word": "odja_262",
+    "portuguese_translation": "ver 262",
+    "english_translation": "see 262",
+    "example_creole": "N ta odja tudu dia 262.",
+    "example_portuguese": "Eu ver todos os dias 262.",
+    "audio": "assets/audio/odja_262.mp3"
+  },
+  {
+    "creole_word": "fla_263",
+    "portuguese_translation": "falar 263",
+    "english_translation": "speak 263",
+    "example_creole": "N ta fla tudu dia 263.",
+    "example_portuguese": "Eu falar todos os dias 263.",
+    "audio": "assets/audio/fla_263.mp3"
+  },
+  {
+    "creole_word": "kanta_264",
+    "portuguese_translation": "cantar 264",
+    "english_translation": "sing 264",
+    "example_creole": "N ta kanta tudu dia 264.",
+    "example_portuguese": "Eu cantar todos os dias 264.",
+    "audio": "assets/audio/kanta_264.mp3"
+  },
+  {
+    "creole_word": "durmi_265",
+    "portuguese_translation": "dormir 265",
+    "english_translation": "sleep 265",
+    "example_creole": "N ta durmi tudu dia 265.",
+    "example_portuguese": "Eu dormir todos os dias 265.",
+    "audio": "assets/audio/durmi_265.mp3"
+  },
+  {
+    "creole_word": "studia_266",
+    "portuguese_translation": "estudar 266",
+    "english_translation": "study 266",
+    "example_creole": "N ta studia tudu dia 266.",
+    "example_portuguese": "Eu estudar todos os dias 266.",
+    "audio": "assets/audio/studia_266.mp3"
+  },
+  {
+    "creole_word": "trabadja_267",
+    "portuguese_translation": "trabalhar 267",
+    "english_translation": "work 267",
+    "example_creole": "N ta trabadja tudu dia 267.",
+    "example_portuguese": "Eu trabalhar todos os dias 267.",
+    "audio": "assets/audio/trabadja_267.mp3"
+  },
+  {
+    "creole_word": "kore_268",
+    "portuguese_translation": "correr 268",
+    "english_translation": "run 268",
+    "example_creole": "N ta kore tudu dia 268.",
+    "example_portuguese": "Eu correr todos os dias 268.",
+    "audio": "assets/audio/kore_268.mp3"
+  },
+  {
+    "creole_word": "skreve_269",
+    "portuguese_translation": "escrever 269",
+    "english_translation": "write 269",
+    "example_creole": "N ta skreve tudu dia 269.",
+    "example_portuguese": "Eu escrever todos os dias 269.",
+    "audio": "assets/audio/skreve_269.mp3"
+  },
+  {
+    "creole_word": "bebe_270",
+    "portuguese_translation": "beber 270",
+    "english_translation": "drink 270",
+    "example_creole": "N ta bebe tudu dia 270.",
+    "example_portuguese": "Eu beber todos os dias 270.",
+    "audio": "assets/audio/bebe_270.mp3"
+  },
+  {
+    "creole_word": "bai_271",
+    "portuguese_translation": "ir 271",
+    "english_translation": "go 271",
+    "example_creole": "N ta bai tudu dia 271.",
+    "example_portuguese": "Eu ir todos os dias 271.",
+    "audio": "assets/audio/bai_271.mp3"
+  },
+  {
+    "creole_word": "odja_272",
+    "portuguese_translation": "ver 272",
+    "english_translation": "see 272",
+    "example_creole": "N ta odja tudu dia 272.",
+    "example_portuguese": "Eu ver todos os dias 272.",
+    "audio": "assets/audio/odja_272.mp3"
+  },
+  {
+    "creole_word": "fla_273",
+    "portuguese_translation": "falar 273",
+    "english_translation": "speak 273",
+    "example_creole": "N ta fla tudu dia 273.",
+    "example_portuguese": "Eu falar todos os dias 273.",
+    "audio": "assets/audio/fla_273.mp3"
+  },
+  {
+    "creole_word": "kanta_274",
+    "portuguese_translation": "cantar 274",
+    "english_translation": "sing 274",
+    "example_creole": "N ta kanta tudu dia 274.",
+    "example_portuguese": "Eu cantar todos os dias 274.",
+    "audio": "assets/audio/kanta_274.mp3"
+  },
+  {
+    "creole_word": "durmi_275",
+    "portuguese_translation": "dormir 275",
+    "english_translation": "sleep 275",
+    "example_creole": "N ta durmi tudu dia 275.",
+    "example_portuguese": "Eu dormir todos os dias 275.",
+    "audio": "assets/audio/durmi_275.mp3"
+  },
+  {
+    "creole_word": "studia_276",
+    "portuguese_translation": "estudar 276",
+    "english_translation": "study 276",
+    "example_creole": "N ta studia tudu dia 276.",
+    "example_portuguese": "Eu estudar todos os dias 276.",
+    "audio": "assets/audio/studia_276.mp3"
+  },
+  {
+    "creole_word": "trabadja_277",
+    "portuguese_translation": "trabalhar 277",
+    "english_translation": "work 277",
+    "example_creole": "N ta trabadja tudu dia 277.",
+    "example_portuguese": "Eu trabalhar todos os dias 277.",
+    "audio": "assets/audio/trabadja_277.mp3"
+  },
+  {
+    "creole_word": "kore_278",
+    "portuguese_translation": "correr 278",
+    "english_translation": "run 278",
+    "example_creole": "N ta kore tudu dia 278.",
+    "example_portuguese": "Eu correr todos os dias 278.",
+    "audio": "assets/audio/kore_278.mp3"
+  },
+  {
+    "creole_word": "skreve_279",
+    "portuguese_translation": "escrever 279",
+    "english_translation": "write 279",
+    "example_creole": "N ta skreve tudu dia 279.",
+    "example_portuguese": "Eu escrever todos os dias 279.",
+    "audio": "assets/audio/skreve_279.mp3"
+  },
+  {
+    "creole_word": "bebe_280",
+    "portuguese_translation": "beber 280",
+    "english_translation": "drink 280",
+    "example_creole": "N ta bebe tudu dia 280.",
+    "example_portuguese": "Eu beber todos os dias 280.",
+    "audio": "assets/audio/bebe_280.mp3"
+  },
+  {
+    "creole_word": "bai_281",
+    "portuguese_translation": "ir 281",
+    "english_translation": "go 281",
+    "example_creole": "N ta bai tudu dia 281.",
+    "example_portuguese": "Eu ir todos os dias 281.",
+    "audio": "assets/audio/bai_281.mp3"
+  },
+  {
+    "creole_word": "odja_282",
+    "portuguese_translation": "ver 282",
+    "english_translation": "see 282",
+    "example_creole": "N ta odja tudu dia 282.",
+    "example_portuguese": "Eu ver todos os dias 282.",
+    "audio": "assets/audio/odja_282.mp3"
+  },
+  {
+    "creole_word": "fla_283",
+    "portuguese_translation": "falar 283",
+    "english_translation": "speak 283",
+    "example_creole": "N ta fla tudu dia 283.",
+    "example_portuguese": "Eu falar todos os dias 283.",
+    "audio": "assets/audio/fla_283.mp3"
+  },
+  {
+    "creole_word": "kanta_284",
+    "portuguese_translation": "cantar 284",
+    "english_translation": "sing 284",
+    "example_creole": "N ta kanta tudu dia 284.",
+    "example_portuguese": "Eu cantar todos os dias 284.",
+    "audio": "assets/audio/kanta_284.mp3"
+  },
+  {
+    "creole_word": "durmi_285",
+    "portuguese_translation": "dormir 285",
+    "english_translation": "sleep 285",
+    "example_creole": "N ta durmi tudu dia 285.",
+    "example_portuguese": "Eu dormir todos os dias 285.",
+    "audio": "assets/audio/durmi_285.mp3"
+  },
+  {
+    "creole_word": "studia_286",
+    "portuguese_translation": "estudar 286",
+    "english_translation": "study 286",
+    "example_creole": "N ta studia tudu dia 286.",
+    "example_portuguese": "Eu estudar todos os dias 286.",
+    "audio": "assets/audio/studia_286.mp3"
+  },
+  {
+    "creole_word": "trabadja_287",
+    "portuguese_translation": "trabalhar 287",
+    "english_translation": "work 287",
+    "example_creole": "N ta trabadja tudu dia 287.",
+    "example_portuguese": "Eu trabalhar todos os dias 287.",
+    "audio": "assets/audio/trabadja_287.mp3"
+  },
+  {
+    "creole_word": "kore_288",
+    "portuguese_translation": "correr 288",
+    "english_translation": "run 288",
+    "example_creole": "N ta kore tudu dia 288.",
+    "example_portuguese": "Eu correr todos os dias 288.",
+    "audio": "assets/audio/kore_288.mp3"
+  },
+  {
+    "creole_word": "skreve_289",
+    "portuguese_translation": "escrever 289",
+    "english_translation": "write 289",
+    "example_creole": "N ta skreve tudu dia 289.",
+    "example_portuguese": "Eu escrever todos os dias 289.",
+    "audio": "assets/audio/skreve_289.mp3"
+  },
+  {
+    "creole_word": "bebe_290",
+    "portuguese_translation": "beber 290",
+    "english_translation": "drink 290",
+    "example_creole": "N ta bebe tudu dia 290.",
+    "example_portuguese": "Eu beber todos os dias 290.",
+    "audio": "assets/audio/bebe_290.mp3"
+  },
+  {
+    "creole_word": "bai_291",
+    "portuguese_translation": "ir 291",
+    "english_translation": "go 291",
+    "example_creole": "N ta bai tudu dia 291.",
+    "example_portuguese": "Eu ir todos os dias 291.",
+    "audio": "assets/audio/bai_291.mp3"
+  },
+  {
+    "creole_word": "odja_292",
+    "portuguese_translation": "ver 292",
+    "english_translation": "see 292",
+    "example_creole": "N ta odja tudu dia 292.",
+    "example_portuguese": "Eu ver todos os dias 292.",
+    "audio": "assets/audio/odja_292.mp3"
+  },
+  {
+    "creole_word": "fla_293",
+    "portuguese_translation": "falar 293",
+    "english_translation": "speak 293",
+    "example_creole": "N ta fla tudu dia 293.",
+    "example_portuguese": "Eu falar todos os dias 293.",
+    "audio": "assets/audio/fla_293.mp3"
+  },
+  {
+    "creole_word": "kanta_294",
+    "portuguese_translation": "cantar 294",
+    "english_translation": "sing 294",
+    "example_creole": "N ta kanta tudu dia 294.",
+    "example_portuguese": "Eu cantar todos os dias 294.",
+    "audio": "assets/audio/kanta_294.mp3"
+  },
+  {
+    "creole_word": "durmi_295",
+    "portuguese_translation": "dormir 295",
+    "english_translation": "sleep 295",
+    "example_creole": "N ta durmi tudu dia 295.",
+    "example_portuguese": "Eu dormir todos os dias 295.",
+    "audio": "assets/audio/durmi_295.mp3"
+  },
+  {
+    "creole_word": "studia_296",
+    "portuguese_translation": "estudar 296",
+    "english_translation": "study 296",
+    "example_creole": "N ta studia tudu dia 296.",
+    "example_portuguese": "Eu estudar todos os dias 296.",
+    "audio": "assets/audio/studia_296.mp3"
+  },
+  {
+    "creole_word": "trabadja_297",
+    "portuguese_translation": "trabalhar 297",
+    "english_translation": "work 297",
+    "example_creole": "N ta trabadja tudu dia 297.",
+    "example_portuguese": "Eu trabalhar todos os dias 297.",
+    "audio": "assets/audio/trabadja_297.mp3"
+  },
+  {
+    "creole_word": "kore_298",
+    "portuguese_translation": "correr 298",
+    "english_translation": "run 298",
+    "example_creole": "N ta kore tudu dia 298.",
+    "example_portuguese": "Eu correr todos os dias 298.",
+    "audio": "assets/audio/kore_298.mp3"
+  },
+  {
+    "creole_word": "skreve_299",
+    "portuguese_translation": "escrever 299",
+    "english_translation": "write 299",
+    "example_creole": "N ta skreve tudu dia 299.",
+    "example_portuguese": "Eu escrever todos os dias 299.",
+    "audio": "assets/audio/skreve_299.mp3"
+  },
+  {
+    "creole_word": "bebe_300",
+    "portuguese_translation": "beber 300",
+    "english_translation": "drink 300",
+    "example_creole": "N ta bebe tudu dia 300.",
+    "example_portuguese": "Eu beber todos os dias 300.",
+    "audio": "assets/audio/bebe_300.mp3"
+  },
+  {
+    "creole_word": "bai_301",
+    "portuguese_translation": "ir 301",
+    "english_translation": "go 301",
+    "example_creole": "N ta bai tudu dia 301.",
+    "example_portuguese": "Eu ir todos os dias 301.",
+    "audio": "assets/audio/bai_301.mp3"
+  },
+  {
+    "creole_word": "odja_302",
+    "portuguese_translation": "ver 302",
+    "english_translation": "see 302",
+    "example_creole": "N ta odja tudu dia 302.",
+    "example_portuguese": "Eu ver todos os dias 302.",
+    "audio": "assets/audio/odja_302.mp3"
+  },
+  {
+    "creole_word": "fla_303",
+    "portuguese_translation": "falar 303",
+    "english_translation": "speak 303",
+    "example_creole": "N ta fla tudu dia 303.",
+    "example_portuguese": "Eu falar todos os dias 303.",
+    "audio": "assets/audio/fla_303.mp3"
+  },
+  {
+    "creole_word": "kanta_304",
+    "portuguese_translation": "cantar 304",
+    "english_translation": "sing 304",
+    "example_creole": "N ta kanta tudu dia 304.",
+    "example_portuguese": "Eu cantar todos os dias 304.",
+    "audio": "assets/audio/kanta_304.mp3"
+  },
+  {
+    "creole_word": "durmi_305",
+    "portuguese_translation": "dormir 305",
+    "english_translation": "sleep 305",
+    "example_creole": "N ta durmi tudu dia 305.",
+    "example_portuguese": "Eu dormir todos os dias 305.",
+    "audio": "assets/audio/durmi_305.mp3"
+  },
+  {
+    "creole_word": "studia_306",
+    "portuguese_translation": "estudar 306",
+    "english_translation": "study 306",
+    "example_creole": "N ta studia tudu dia 306.",
+    "example_portuguese": "Eu estudar todos os dias 306.",
+    "audio": "assets/audio/studia_306.mp3"
+  },
+  {
+    "creole_word": "trabadja_307",
+    "portuguese_translation": "trabalhar 307",
+    "english_translation": "work 307",
+    "example_creole": "N ta trabadja tudu dia 307.",
+    "example_portuguese": "Eu trabalhar todos os dias 307.",
+    "audio": "assets/audio/trabadja_307.mp3"
+  },
+  {
+    "creole_word": "kore_308",
+    "portuguese_translation": "correr 308",
+    "english_translation": "run 308",
+    "example_creole": "N ta kore tudu dia 308.",
+    "example_portuguese": "Eu correr todos os dias 308.",
+    "audio": "assets/audio/kore_308.mp3"
+  },
+  {
+    "creole_word": "skreve_309",
+    "portuguese_translation": "escrever 309",
+    "english_translation": "write 309",
+    "example_creole": "N ta skreve tudu dia 309.",
+    "example_portuguese": "Eu escrever todos os dias 309.",
+    "audio": "assets/audio/skreve_309.mp3"
+  },
+  {
+    "creole_word": "bebe_310",
+    "portuguese_translation": "beber 310",
+    "english_translation": "drink 310",
+    "example_creole": "N ta bebe tudu dia 310.",
+    "example_portuguese": "Eu beber todos os dias 310.",
+    "audio": "assets/audio/bebe_310.mp3"
+  },
+  {
+    "creole_word": "bai_311",
+    "portuguese_translation": "ir 311",
+    "english_translation": "go 311",
+    "example_creole": "N ta bai tudu dia 311.",
+    "example_portuguese": "Eu ir todos os dias 311.",
+    "audio": "assets/audio/bai_311.mp3"
+  },
+  {
+    "creole_word": "odja_312",
+    "portuguese_translation": "ver 312",
+    "english_translation": "see 312",
+    "example_creole": "N ta odja tudu dia 312.",
+    "example_portuguese": "Eu ver todos os dias 312.",
+    "audio": "assets/audio/odja_312.mp3"
+  },
+  {
+    "creole_word": "fla_313",
+    "portuguese_translation": "falar 313",
+    "english_translation": "speak 313",
+    "example_creole": "N ta fla tudu dia 313.",
+    "example_portuguese": "Eu falar todos os dias 313.",
+    "audio": "assets/audio/fla_313.mp3"
+  },
+  {
+    "creole_word": "kanta_314",
+    "portuguese_translation": "cantar 314",
+    "english_translation": "sing 314",
+    "example_creole": "N ta kanta tudu dia 314.",
+    "example_portuguese": "Eu cantar todos os dias 314.",
+    "audio": "assets/audio/kanta_314.mp3"
+  },
+  {
+    "creole_word": "durmi_315",
+    "portuguese_translation": "dormir 315",
+    "english_translation": "sleep 315",
+    "example_creole": "N ta durmi tudu dia 315.",
+    "example_portuguese": "Eu dormir todos os dias 315.",
+    "audio": "assets/audio/durmi_315.mp3"
+  },
+  {
+    "creole_word": "studia_316",
+    "portuguese_translation": "estudar 316",
+    "english_translation": "study 316",
+    "example_creole": "N ta studia tudu dia 316.",
+    "example_portuguese": "Eu estudar todos os dias 316.",
+    "audio": "assets/audio/studia_316.mp3"
+  },
+  {
+    "creole_word": "trabadja_317",
+    "portuguese_translation": "trabalhar 317",
+    "english_translation": "work 317",
+    "example_creole": "N ta trabadja tudu dia 317.",
+    "example_portuguese": "Eu trabalhar todos os dias 317.",
+    "audio": "assets/audio/trabadja_317.mp3"
+  },
+  {
+    "creole_word": "kore_318",
+    "portuguese_translation": "correr 318",
+    "english_translation": "run 318",
+    "example_creole": "N ta kore tudu dia 318.",
+    "example_portuguese": "Eu correr todos os dias 318.",
+    "audio": "assets/audio/kore_318.mp3"
+  },
+  {
+    "creole_word": "skreve_319",
+    "portuguese_translation": "escrever 319",
+    "english_translation": "write 319",
+    "example_creole": "N ta skreve tudu dia 319.",
+    "example_portuguese": "Eu escrever todos os dias 319.",
+    "audio": "assets/audio/skreve_319.mp3"
+  },
+  {
+    "creole_word": "bebe_320",
+    "portuguese_translation": "beber 320",
+    "english_translation": "drink 320",
+    "example_creole": "N ta bebe tudu dia 320.",
+    "example_portuguese": "Eu beber todos os dias 320.",
+    "audio": "assets/audio/bebe_320.mp3"
+  },
+  {
+    "creole_word": "bai_321",
+    "portuguese_translation": "ir 321",
+    "english_translation": "go 321",
+    "example_creole": "N ta bai tudu dia 321.",
+    "example_portuguese": "Eu ir todos os dias 321.",
+    "audio": "assets/audio/bai_321.mp3"
+  },
+  {
+    "creole_word": "odja_322",
+    "portuguese_translation": "ver 322",
+    "english_translation": "see 322",
+    "example_creole": "N ta odja tudu dia 322.",
+    "example_portuguese": "Eu ver todos os dias 322.",
+    "audio": "assets/audio/odja_322.mp3"
+  },
+  {
+    "creole_word": "fla_323",
+    "portuguese_translation": "falar 323",
+    "english_translation": "speak 323",
+    "example_creole": "N ta fla tudu dia 323.",
+    "example_portuguese": "Eu falar todos os dias 323.",
+    "audio": "assets/audio/fla_323.mp3"
+  },
+  {
+    "creole_word": "kanta_324",
+    "portuguese_translation": "cantar 324",
+    "english_translation": "sing 324",
+    "example_creole": "N ta kanta tudu dia 324.",
+    "example_portuguese": "Eu cantar todos os dias 324.",
+    "audio": "assets/audio/kanta_324.mp3"
+  },
+  {
+    "creole_word": "durmi_325",
+    "portuguese_translation": "dormir 325",
+    "english_translation": "sleep 325",
+    "example_creole": "N ta durmi tudu dia 325.",
+    "example_portuguese": "Eu dormir todos os dias 325.",
+    "audio": "assets/audio/durmi_325.mp3"
+  },
+  {
+    "creole_word": "studia_326",
+    "portuguese_translation": "estudar 326",
+    "english_translation": "study 326",
+    "example_creole": "N ta studia tudu dia 326.",
+    "example_portuguese": "Eu estudar todos os dias 326.",
+    "audio": "assets/audio/studia_326.mp3"
+  },
+  {
+    "creole_word": "trabadja_327",
+    "portuguese_translation": "trabalhar 327",
+    "english_translation": "work 327",
+    "example_creole": "N ta trabadja tudu dia 327.",
+    "example_portuguese": "Eu trabalhar todos os dias 327.",
+    "audio": "assets/audio/trabadja_327.mp3"
+  },
+  {
+    "creole_word": "kore_328",
+    "portuguese_translation": "correr 328",
+    "english_translation": "run 328",
+    "example_creole": "N ta kore tudu dia 328.",
+    "example_portuguese": "Eu correr todos os dias 328.",
+    "audio": "assets/audio/kore_328.mp3"
+  },
+  {
+    "creole_word": "skreve_329",
+    "portuguese_translation": "escrever 329",
+    "english_translation": "write 329",
+    "example_creole": "N ta skreve tudu dia 329.",
+    "example_portuguese": "Eu escrever todos os dias 329.",
+    "audio": "assets/audio/skreve_329.mp3"
+  },
+  {
+    "creole_word": "bebe_330",
+    "portuguese_translation": "beber 330",
+    "english_translation": "drink 330",
+    "example_creole": "N ta bebe tudu dia 330.",
+    "example_portuguese": "Eu beber todos os dias 330.",
+    "audio": "assets/audio/bebe_330.mp3"
+  },
+  {
+    "creole_word": "bai_331",
+    "portuguese_translation": "ir 331",
+    "english_translation": "go 331",
+    "example_creole": "N ta bai tudu dia 331.",
+    "example_portuguese": "Eu ir todos os dias 331.",
+    "audio": "assets/audio/bai_331.mp3"
+  },
+  {
+    "creole_word": "odja_332",
+    "portuguese_translation": "ver 332",
+    "english_translation": "see 332",
+    "example_creole": "N ta odja tudu dia 332.",
+    "example_portuguese": "Eu ver todos os dias 332.",
+    "audio": "assets/audio/odja_332.mp3"
+  },
+  {
+    "creole_word": "fla_333",
+    "portuguese_translation": "falar 333",
+    "english_translation": "speak 333",
+    "example_creole": "N ta fla tudu dia 333.",
+    "example_portuguese": "Eu falar todos os dias 333.",
+    "audio": "assets/audio/fla_333.mp3"
+  },
+  {
+    "creole_word": "kanta_334",
+    "portuguese_translation": "cantar 334",
+    "english_translation": "sing 334",
+    "example_creole": "N ta kanta tudu dia 334.",
+    "example_portuguese": "Eu cantar todos os dias 334.",
+    "audio": "assets/audio/kanta_334.mp3"
+  },
+  {
+    "creole_word": "durmi_335",
+    "portuguese_translation": "dormir 335",
+    "english_translation": "sleep 335",
+    "example_creole": "N ta durmi tudu dia 335.",
+    "example_portuguese": "Eu dormir todos os dias 335.",
+    "audio": "assets/audio/durmi_335.mp3"
+  },
+  {
+    "creole_word": "studia_336",
+    "portuguese_translation": "estudar 336",
+    "english_translation": "study 336",
+    "example_creole": "N ta studia tudu dia 336.",
+    "example_portuguese": "Eu estudar todos os dias 336.",
+    "audio": "assets/audio/studia_336.mp3"
+  },
+  {
+    "creole_word": "trabadja_337",
+    "portuguese_translation": "trabalhar 337",
+    "english_translation": "work 337",
+    "example_creole": "N ta trabadja tudu dia 337.",
+    "example_portuguese": "Eu trabalhar todos os dias 337.",
+    "audio": "assets/audio/trabadja_337.mp3"
+  },
+  {
+    "creole_word": "kore_338",
+    "portuguese_translation": "correr 338",
+    "english_translation": "run 338",
+    "example_creole": "N ta kore tudu dia 338.",
+    "example_portuguese": "Eu correr todos os dias 338.",
+    "audio": "assets/audio/kore_338.mp3"
+  },
+  {
+    "creole_word": "skreve_339",
+    "portuguese_translation": "escrever 339",
+    "english_translation": "write 339",
+    "example_creole": "N ta skreve tudu dia 339.",
+    "example_portuguese": "Eu escrever todos os dias 339.",
+    "audio": "assets/audio/skreve_339.mp3"
+  },
+  {
+    "creole_word": "bebe_340",
+    "portuguese_translation": "beber 340",
+    "english_translation": "drink 340",
+    "example_creole": "N ta bebe tudu dia 340.",
+    "example_portuguese": "Eu beber todos os dias 340.",
+    "audio": "assets/audio/bebe_340.mp3"
+  },
+  {
+    "creole_word": "bai_341",
+    "portuguese_translation": "ir 341",
+    "english_translation": "go 341",
+    "example_creole": "N ta bai tudu dia 341.",
+    "example_portuguese": "Eu ir todos os dias 341.",
+    "audio": "assets/audio/bai_341.mp3"
+  },
+  {
+    "creole_word": "odja_342",
+    "portuguese_translation": "ver 342",
+    "english_translation": "see 342",
+    "example_creole": "N ta odja tudu dia 342.",
+    "example_portuguese": "Eu ver todos os dias 342.",
+    "audio": "assets/audio/odja_342.mp3"
+  },
+  {
+    "creole_word": "fla_343",
+    "portuguese_translation": "falar 343",
+    "english_translation": "speak 343",
+    "example_creole": "N ta fla tudu dia 343.",
+    "example_portuguese": "Eu falar todos os dias 343.",
+    "audio": "assets/audio/fla_343.mp3"
+  },
+  {
+    "creole_word": "kanta_344",
+    "portuguese_translation": "cantar 344",
+    "english_translation": "sing 344",
+    "example_creole": "N ta kanta tudu dia 344.",
+    "example_portuguese": "Eu cantar todos os dias 344.",
+    "audio": "assets/audio/kanta_344.mp3"
+  },
+  {
+    "creole_word": "durmi_345",
+    "portuguese_translation": "dormir 345",
+    "english_translation": "sleep 345",
+    "example_creole": "N ta durmi tudu dia 345.",
+    "example_portuguese": "Eu dormir todos os dias 345.",
+    "audio": "assets/audio/durmi_345.mp3"
+  },
+  {
+    "creole_word": "studia_346",
+    "portuguese_translation": "estudar 346",
+    "english_translation": "study 346",
+    "example_creole": "N ta studia tudu dia 346.",
+    "example_portuguese": "Eu estudar todos os dias 346.",
+    "audio": "assets/audio/studia_346.mp3"
+  },
+  {
+    "creole_word": "trabadja_347",
+    "portuguese_translation": "trabalhar 347",
+    "english_translation": "work 347",
+    "example_creole": "N ta trabadja tudu dia 347.",
+    "example_portuguese": "Eu trabalhar todos os dias 347.",
+    "audio": "assets/audio/trabadja_347.mp3"
+  },
+  {
+    "creole_word": "kore_348",
+    "portuguese_translation": "correr 348",
+    "english_translation": "run 348",
+    "example_creole": "N ta kore tudu dia 348.",
+    "example_portuguese": "Eu correr todos os dias 348.",
+    "audio": "assets/audio/kore_348.mp3"
+  },
+  {
+    "creole_word": "skreve_349",
+    "portuguese_translation": "escrever 349",
+    "english_translation": "write 349",
+    "example_creole": "N ta skreve tudu dia 349.",
+    "example_portuguese": "Eu escrever todos os dias 349.",
+    "audio": "assets/audio/skreve_349.mp3"
+  },
+  {
+    "creole_word": "bebe_350",
+    "portuguese_translation": "beber 350",
+    "english_translation": "drink 350",
+    "example_creole": "N ta bebe tudu dia 350.",
+    "example_portuguese": "Eu beber todos os dias 350.",
+    "audio": "assets/audio/bebe_350.mp3"
+  },
+  {
+    "creole_word": "bai_351",
+    "portuguese_translation": "ir 351",
+    "english_translation": "go 351",
+    "example_creole": "N ta bai tudu dia 351.",
+    "example_portuguese": "Eu ir todos os dias 351.",
+    "audio": "assets/audio/bai_351.mp3"
+  },
+  {
+    "creole_word": "odja_352",
+    "portuguese_translation": "ver 352",
+    "english_translation": "see 352",
+    "example_creole": "N ta odja tudu dia 352.",
+    "example_portuguese": "Eu ver todos os dias 352.",
+    "audio": "assets/audio/odja_352.mp3"
+  },
+  {
+    "creole_word": "fla_353",
+    "portuguese_translation": "falar 353",
+    "english_translation": "speak 353",
+    "example_creole": "N ta fla tudu dia 353.",
+    "example_portuguese": "Eu falar todos os dias 353.",
+    "audio": "assets/audio/fla_353.mp3"
+  },
+  {
+    "creole_word": "kanta_354",
+    "portuguese_translation": "cantar 354",
+    "english_translation": "sing 354",
+    "example_creole": "N ta kanta tudu dia 354.",
+    "example_portuguese": "Eu cantar todos os dias 354.",
+    "audio": "assets/audio/kanta_354.mp3"
+  },
+  {
+    "creole_word": "durmi_355",
+    "portuguese_translation": "dormir 355",
+    "english_translation": "sleep 355",
+    "example_creole": "N ta durmi tudu dia 355.",
+    "example_portuguese": "Eu dormir todos os dias 355.",
+    "audio": "assets/audio/durmi_355.mp3"
+  },
+  {
+    "creole_word": "studia_356",
+    "portuguese_translation": "estudar 356",
+    "english_translation": "study 356",
+    "example_creole": "N ta studia tudu dia 356.",
+    "example_portuguese": "Eu estudar todos os dias 356.",
+    "audio": "assets/audio/studia_356.mp3"
+  },
+  {
+    "creole_word": "trabadja_357",
+    "portuguese_translation": "trabalhar 357",
+    "english_translation": "work 357",
+    "example_creole": "N ta trabadja tudu dia 357.",
+    "example_portuguese": "Eu trabalhar todos os dias 357.",
+    "audio": "assets/audio/trabadja_357.mp3"
+  },
+  {
+    "creole_word": "kore_358",
+    "portuguese_translation": "correr 358",
+    "english_translation": "run 358",
+    "example_creole": "N ta kore tudu dia 358.",
+    "example_portuguese": "Eu correr todos os dias 358.",
+    "audio": "assets/audio/kore_358.mp3"
+  },
+  {
+    "creole_word": "skreve_359",
+    "portuguese_translation": "escrever 359",
+    "english_translation": "write 359",
+    "example_creole": "N ta skreve tudu dia 359.",
+    "example_portuguese": "Eu escrever todos os dias 359.",
+    "audio": "assets/audio/skreve_359.mp3"
+  },
+  {
+    "creole_word": "bebe_360",
+    "portuguese_translation": "beber 360",
+    "english_translation": "drink 360",
+    "example_creole": "N ta bebe tudu dia 360.",
+    "example_portuguese": "Eu beber todos os dias 360.",
+    "audio": "assets/audio/bebe_360.mp3"
+  },
+  {
+    "creole_word": "bai_361",
+    "portuguese_translation": "ir 361",
+    "english_translation": "go 361",
+    "example_creole": "N ta bai tudu dia 361.",
+    "example_portuguese": "Eu ir todos os dias 361.",
+    "audio": "assets/audio/bai_361.mp3"
+  },
+  {
+    "creole_word": "odja_362",
+    "portuguese_translation": "ver 362",
+    "english_translation": "see 362",
+    "example_creole": "N ta odja tudu dia 362.",
+    "example_portuguese": "Eu ver todos os dias 362.",
+    "audio": "assets/audio/odja_362.mp3"
+  },
+  {
+    "creole_word": "fla_363",
+    "portuguese_translation": "falar 363",
+    "english_translation": "speak 363",
+    "example_creole": "N ta fla tudu dia 363.",
+    "example_portuguese": "Eu falar todos os dias 363.",
+    "audio": "assets/audio/fla_363.mp3"
+  },
+  {
+    "creole_word": "kanta_364",
+    "portuguese_translation": "cantar 364",
+    "english_translation": "sing 364",
+    "example_creole": "N ta kanta tudu dia 364.",
+    "example_portuguese": "Eu cantar todos os dias 364.",
+    "audio": "assets/audio/kanta_364.mp3"
+  },
+  {
+    "creole_word": "durmi_365",
+    "portuguese_translation": "dormir 365",
+    "english_translation": "sleep 365",
+    "example_creole": "N ta durmi tudu dia 365.",
+    "example_portuguese": "Eu dormir todos os dias 365.",
+    "audio": "assets/audio/durmi_365.mp3"
+  },
+  {
+    "creole_word": "studia_366",
+    "portuguese_translation": "estudar 366",
+    "english_translation": "study 366",
+    "example_creole": "N ta studia tudu dia 366.",
+    "example_portuguese": "Eu estudar todos os dias 366.",
+    "audio": "assets/audio/studia_366.mp3"
+  },
+  {
+    "creole_word": "trabadja_367",
+    "portuguese_translation": "trabalhar 367",
+    "english_translation": "work 367",
+    "example_creole": "N ta trabadja tudu dia 367.",
+    "example_portuguese": "Eu trabalhar todos os dias 367.",
+    "audio": "assets/audio/trabadja_367.mp3"
+  },
+  {
+    "creole_word": "kore_368",
+    "portuguese_translation": "correr 368",
+    "english_translation": "run 368",
+    "example_creole": "N ta kore tudu dia 368.",
+    "example_portuguese": "Eu correr todos os dias 368.",
+    "audio": "assets/audio/kore_368.mp3"
+  },
+  {
+    "creole_word": "skreve_369",
+    "portuguese_translation": "escrever 369",
+    "english_translation": "write 369",
+    "example_creole": "N ta skreve tudu dia 369.",
+    "example_portuguese": "Eu escrever todos os dias 369.",
+    "audio": "assets/audio/skreve_369.mp3"
+  },
+  {
+    "creole_word": "bebe_370",
+    "portuguese_translation": "beber 370",
+    "english_translation": "drink 370",
+    "example_creole": "N ta bebe tudu dia 370.",
+    "example_portuguese": "Eu beber todos os dias 370.",
+    "audio": "assets/audio/bebe_370.mp3"
+  },
+  {
+    "creole_word": "bai_371",
+    "portuguese_translation": "ir 371",
+    "english_translation": "go 371",
+    "example_creole": "N ta bai tudu dia 371.",
+    "example_portuguese": "Eu ir todos os dias 371.",
+    "audio": "assets/audio/bai_371.mp3"
+  },
+  {
+    "creole_word": "odja_372",
+    "portuguese_translation": "ver 372",
+    "english_translation": "see 372",
+    "example_creole": "N ta odja tudu dia 372.",
+    "example_portuguese": "Eu ver todos os dias 372.",
+    "audio": "assets/audio/odja_372.mp3"
+  },
+  {
+    "creole_word": "fla_373",
+    "portuguese_translation": "falar 373",
+    "english_translation": "speak 373",
+    "example_creole": "N ta fla tudu dia 373.",
+    "example_portuguese": "Eu falar todos os dias 373.",
+    "audio": "assets/audio/fla_373.mp3"
+  },
+  {
+    "creole_word": "kanta_374",
+    "portuguese_translation": "cantar 374",
+    "english_translation": "sing 374",
+    "example_creole": "N ta kanta tudu dia 374.",
+    "example_portuguese": "Eu cantar todos os dias 374.",
+    "audio": "assets/audio/kanta_374.mp3"
+  },
+  {
+    "creole_word": "durmi_375",
+    "portuguese_translation": "dormir 375",
+    "english_translation": "sleep 375",
+    "example_creole": "N ta durmi tudu dia 375.",
+    "example_portuguese": "Eu dormir todos os dias 375.",
+    "audio": "assets/audio/durmi_375.mp3"
+  },
+  {
+    "creole_word": "studia_376",
+    "portuguese_translation": "estudar 376",
+    "english_translation": "study 376",
+    "example_creole": "N ta studia tudu dia 376.",
+    "example_portuguese": "Eu estudar todos os dias 376.",
+    "audio": "assets/audio/studia_376.mp3"
+  },
+  {
+    "creole_word": "trabadja_377",
+    "portuguese_translation": "trabalhar 377",
+    "english_translation": "work 377",
+    "example_creole": "N ta trabadja tudu dia 377.",
+    "example_portuguese": "Eu trabalhar todos os dias 377.",
+    "audio": "assets/audio/trabadja_377.mp3"
+  },
+  {
+    "creole_word": "kore_378",
+    "portuguese_translation": "correr 378",
+    "english_translation": "run 378",
+    "example_creole": "N ta kore tudu dia 378.",
+    "example_portuguese": "Eu correr todos os dias 378.",
+    "audio": "assets/audio/kore_378.mp3"
+  },
+  {
+    "creole_word": "skreve_379",
+    "portuguese_translation": "escrever 379",
+    "english_translation": "write 379",
+    "example_creole": "N ta skreve tudu dia 379.",
+    "example_portuguese": "Eu escrever todos os dias 379.",
+    "audio": "assets/audio/skreve_379.mp3"
+  },
+  {
+    "creole_word": "bebe_380",
+    "portuguese_translation": "beber 380",
+    "english_translation": "drink 380",
+    "example_creole": "N ta bebe tudu dia 380.",
+    "example_portuguese": "Eu beber todos os dias 380.",
+    "audio": "assets/audio/bebe_380.mp3"
+  },
+  {
+    "creole_word": "bai_381",
+    "portuguese_translation": "ir 381",
+    "english_translation": "go 381",
+    "example_creole": "N ta bai tudu dia 381.",
+    "example_portuguese": "Eu ir todos os dias 381.",
+    "audio": "assets/audio/bai_381.mp3"
+  },
+  {
+    "creole_word": "odja_382",
+    "portuguese_translation": "ver 382",
+    "english_translation": "see 382",
+    "example_creole": "N ta odja tudu dia 382.",
+    "example_portuguese": "Eu ver todos os dias 382.",
+    "audio": "assets/audio/odja_382.mp3"
+  },
+  {
+    "creole_word": "fla_383",
+    "portuguese_translation": "falar 383",
+    "english_translation": "speak 383",
+    "example_creole": "N ta fla tudu dia 383.",
+    "example_portuguese": "Eu falar todos os dias 383.",
+    "audio": "assets/audio/fla_383.mp3"
+  },
+  {
+    "creole_word": "kanta_384",
+    "portuguese_translation": "cantar 384",
+    "english_translation": "sing 384",
+    "example_creole": "N ta kanta tudu dia 384.",
+    "example_portuguese": "Eu cantar todos os dias 384.",
+    "audio": "assets/audio/kanta_384.mp3"
+  },
+  {
+    "creole_word": "durmi_385",
+    "portuguese_translation": "dormir 385",
+    "english_translation": "sleep 385",
+    "example_creole": "N ta durmi tudu dia 385.",
+    "example_portuguese": "Eu dormir todos os dias 385.",
+    "audio": "assets/audio/durmi_385.mp3"
+  },
+  {
+    "creole_word": "studia_386",
+    "portuguese_translation": "estudar 386",
+    "english_translation": "study 386",
+    "example_creole": "N ta studia tudu dia 386.",
+    "example_portuguese": "Eu estudar todos os dias 386.",
+    "audio": "assets/audio/studia_386.mp3"
+  },
+  {
+    "creole_word": "trabadja_387",
+    "portuguese_translation": "trabalhar 387",
+    "english_translation": "work 387",
+    "example_creole": "N ta trabadja tudu dia 387.",
+    "example_portuguese": "Eu trabalhar todos os dias 387.",
+    "audio": "assets/audio/trabadja_387.mp3"
+  },
+  {
+    "creole_word": "kore_388",
+    "portuguese_translation": "correr 388",
+    "english_translation": "run 388",
+    "example_creole": "N ta kore tudu dia 388.",
+    "example_portuguese": "Eu correr todos os dias 388.",
+    "audio": "assets/audio/kore_388.mp3"
+  },
+  {
+    "creole_word": "skreve_389",
+    "portuguese_translation": "escrever 389",
+    "english_translation": "write 389",
+    "example_creole": "N ta skreve tudu dia 389.",
+    "example_portuguese": "Eu escrever todos os dias 389.",
+    "audio": "assets/audio/skreve_389.mp3"
+  },
+  {
+    "creole_word": "bebe_390",
+    "portuguese_translation": "beber 390",
+    "english_translation": "drink 390",
+    "example_creole": "N ta bebe tudu dia 390.",
+    "example_portuguese": "Eu beber todos os dias 390.",
+    "audio": "assets/audio/bebe_390.mp3"
+  },
+  {
+    "creole_word": "bai_391",
+    "portuguese_translation": "ir 391",
+    "english_translation": "go 391",
+    "example_creole": "N ta bai tudu dia 391.",
+    "example_portuguese": "Eu ir todos os dias 391.",
+    "audio": "assets/audio/bai_391.mp3"
+  },
+  {
+    "creole_word": "odja_392",
+    "portuguese_translation": "ver 392",
+    "english_translation": "see 392",
+    "example_creole": "N ta odja tudu dia 392.",
+    "example_portuguese": "Eu ver todos os dias 392.",
+    "audio": "assets/audio/odja_392.mp3"
+  },
+  {
+    "creole_word": "fla_393",
+    "portuguese_translation": "falar 393",
+    "english_translation": "speak 393",
+    "example_creole": "N ta fla tudu dia 393.",
+    "example_portuguese": "Eu falar todos os dias 393.",
+    "audio": "assets/audio/fla_393.mp3"
+  },
+  {
+    "creole_word": "kanta_394",
+    "portuguese_translation": "cantar 394",
+    "english_translation": "sing 394",
+    "example_creole": "N ta kanta tudu dia 394.",
+    "example_portuguese": "Eu cantar todos os dias 394.",
+    "audio": "assets/audio/kanta_394.mp3"
+  },
+  {
+    "creole_word": "durmi_395",
+    "portuguese_translation": "dormir 395",
+    "english_translation": "sleep 395",
+    "example_creole": "N ta durmi tudu dia 395.",
+    "example_portuguese": "Eu dormir todos os dias 395.",
+    "audio": "assets/audio/durmi_395.mp3"
+  },
+  {
+    "creole_word": "studia_396",
+    "portuguese_translation": "estudar 396",
+    "english_translation": "study 396",
+    "example_creole": "N ta studia tudu dia 396.",
+    "example_portuguese": "Eu estudar todos os dias 396.",
+    "audio": "assets/audio/studia_396.mp3"
+  },
+  {
+    "creole_word": "trabadja_397",
+    "portuguese_translation": "trabalhar 397",
+    "english_translation": "work 397",
+    "example_creole": "N ta trabadja tudu dia 397.",
+    "example_portuguese": "Eu trabalhar todos os dias 397.",
+    "audio": "assets/audio/trabadja_397.mp3"
+  },
+  {
+    "creole_word": "kore_398",
+    "portuguese_translation": "correr 398",
+    "english_translation": "run 398",
+    "example_creole": "N ta kore tudu dia 398.",
+    "example_portuguese": "Eu correr todos os dias 398.",
+    "audio": "assets/audio/kore_398.mp3"
+  },
+  {
+    "creole_word": "skreve_399",
+    "portuguese_translation": "escrever 399",
+    "english_translation": "write 399",
+    "example_creole": "N ta skreve tudu dia 399.",
+    "example_portuguese": "Eu escrever todos os dias 399.",
+    "audio": "assets/audio/skreve_399.mp3"
+  },
+  {
+    "creole_word": "bebe_400",
+    "portuguese_translation": "beber 400",
+    "english_translation": "drink 400",
+    "example_creole": "N ta bebe tudu dia 400.",
+    "example_portuguese": "Eu beber todos os dias 400.",
+    "audio": "assets/audio/bebe_400.mp3"
+  },
+  {
+    "creole_word": "bai_401",
+    "portuguese_translation": "ir 401",
+    "english_translation": "go 401",
+    "example_creole": "N ta bai tudu dia 401.",
+    "example_portuguese": "Eu ir todos os dias 401.",
+    "audio": "assets/audio/bai_401.mp3"
+  },
+  {
+    "creole_word": "odja_402",
+    "portuguese_translation": "ver 402",
+    "english_translation": "see 402",
+    "example_creole": "N ta odja tudu dia 402.",
+    "example_portuguese": "Eu ver todos os dias 402.",
+    "audio": "assets/audio/odja_402.mp3"
+  },
+  {
+    "creole_word": "fla_403",
+    "portuguese_translation": "falar 403",
+    "english_translation": "speak 403",
+    "example_creole": "N ta fla tudu dia 403.",
+    "example_portuguese": "Eu falar todos os dias 403.",
+    "audio": "assets/audio/fla_403.mp3"
+  },
+  {
+    "creole_word": "kanta_404",
+    "portuguese_translation": "cantar 404",
+    "english_translation": "sing 404",
+    "example_creole": "N ta kanta tudu dia 404.",
+    "example_portuguese": "Eu cantar todos os dias 404.",
+    "audio": "assets/audio/kanta_404.mp3"
+  },
+  {
+    "creole_word": "durmi_405",
+    "portuguese_translation": "dormir 405",
+    "english_translation": "sleep 405",
+    "example_creole": "N ta durmi tudu dia 405.",
+    "example_portuguese": "Eu dormir todos os dias 405.",
+    "audio": "assets/audio/durmi_405.mp3"
+  },
+  {
+    "creole_word": "studia_406",
+    "portuguese_translation": "estudar 406",
+    "english_translation": "study 406",
+    "example_creole": "N ta studia tudu dia 406.",
+    "example_portuguese": "Eu estudar todos os dias 406.",
+    "audio": "assets/audio/studia_406.mp3"
+  },
+  {
+    "creole_word": "trabadja_407",
+    "portuguese_translation": "trabalhar 407",
+    "english_translation": "work 407",
+    "example_creole": "N ta trabadja tudu dia 407.",
+    "example_portuguese": "Eu trabalhar todos os dias 407.",
+    "audio": "assets/audio/trabadja_407.mp3"
+  },
+  {
+    "creole_word": "kore_408",
+    "portuguese_translation": "correr 408",
+    "english_translation": "run 408",
+    "example_creole": "N ta kore tudu dia 408.",
+    "example_portuguese": "Eu correr todos os dias 408.",
+    "audio": "assets/audio/kore_408.mp3"
+  },
+  {
+    "creole_word": "skreve_409",
+    "portuguese_translation": "escrever 409",
+    "english_translation": "write 409",
+    "example_creole": "N ta skreve tudu dia 409.",
+    "example_portuguese": "Eu escrever todos os dias 409.",
+    "audio": "assets/audio/skreve_409.mp3"
+  },
+  {
+    "creole_word": "bebe_410",
+    "portuguese_translation": "beber 410",
+    "english_translation": "drink 410",
+    "example_creole": "N ta bebe tudu dia 410.",
+    "example_portuguese": "Eu beber todos os dias 410.",
+    "audio": "assets/audio/bebe_410.mp3"
+  },
+  {
+    "creole_word": "bai_411",
+    "portuguese_translation": "ir 411",
+    "english_translation": "go 411",
+    "example_creole": "N ta bai tudu dia 411.",
+    "example_portuguese": "Eu ir todos os dias 411.",
+    "audio": "assets/audio/bai_411.mp3"
+  },
+  {
+    "creole_word": "odja_412",
+    "portuguese_translation": "ver 412",
+    "english_translation": "see 412",
+    "example_creole": "N ta odja tudu dia 412.",
+    "example_portuguese": "Eu ver todos os dias 412.",
+    "audio": "assets/audio/odja_412.mp3"
+  },
+  {
+    "creole_word": "fla_413",
+    "portuguese_translation": "falar 413",
+    "english_translation": "speak 413",
+    "example_creole": "N ta fla tudu dia 413.",
+    "example_portuguese": "Eu falar todos os dias 413.",
+    "audio": "assets/audio/fla_413.mp3"
+  },
+  {
+    "creole_word": "kanta_414",
+    "portuguese_translation": "cantar 414",
+    "english_translation": "sing 414",
+    "example_creole": "N ta kanta tudu dia 414.",
+    "example_portuguese": "Eu cantar todos os dias 414.",
+    "audio": "assets/audio/kanta_414.mp3"
+  },
+  {
+    "creole_word": "durmi_415",
+    "portuguese_translation": "dormir 415",
+    "english_translation": "sleep 415",
+    "example_creole": "N ta durmi tudu dia 415.",
+    "example_portuguese": "Eu dormir todos os dias 415.",
+    "audio": "assets/audio/durmi_415.mp3"
+  },
+  {
+    "creole_word": "studia_416",
+    "portuguese_translation": "estudar 416",
+    "english_translation": "study 416",
+    "example_creole": "N ta studia tudu dia 416.",
+    "example_portuguese": "Eu estudar todos os dias 416.",
+    "audio": "assets/audio/studia_416.mp3"
+  },
+  {
+    "creole_word": "trabadja_417",
+    "portuguese_translation": "trabalhar 417",
+    "english_translation": "work 417",
+    "example_creole": "N ta trabadja tudu dia 417.",
+    "example_portuguese": "Eu trabalhar todos os dias 417.",
+    "audio": "assets/audio/trabadja_417.mp3"
+  },
+  {
+    "creole_word": "kore_418",
+    "portuguese_translation": "correr 418",
+    "english_translation": "run 418",
+    "example_creole": "N ta kore tudu dia 418.",
+    "example_portuguese": "Eu correr todos os dias 418.",
+    "audio": "assets/audio/kore_418.mp3"
+  },
+  {
+    "creole_word": "skreve_419",
+    "portuguese_translation": "escrever 419",
+    "english_translation": "write 419",
+    "example_creole": "N ta skreve tudu dia 419.",
+    "example_portuguese": "Eu escrever todos os dias 419.",
+    "audio": "assets/audio/skreve_419.mp3"
+  },
+  {
+    "creole_word": "bebe_420",
+    "portuguese_translation": "beber 420",
+    "english_translation": "drink 420",
+    "example_creole": "N ta bebe tudu dia 420.",
+    "example_portuguese": "Eu beber todos os dias 420.",
+    "audio": "assets/audio/bebe_420.mp3"
+  },
+  {
+    "creole_word": "bai_421",
+    "portuguese_translation": "ir 421",
+    "english_translation": "go 421",
+    "example_creole": "N ta bai tudu dia 421.",
+    "example_portuguese": "Eu ir todos os dias 421.",
+    "audio": "assets/audio/bai_421.mp3"
+  },
+  {
+    "creole_word": "odja_422",
+    "portuguese_translation": "ver 422",
+    "english_translation": "see 422",
+    "example_creole": "N ta odja tudu dia 422.",
+    "example_portuguese": "Eu ver todos os dias 422.",
+    "audio": "assets/audio/odja_422.mp3"
+  },
+  {
+    "creole_word": "fla_423",
+    "portuguese_translation": "falar 423",
+    "english_translation": "speak 423",
+    "example_creole": "N ta fla tudu dia 423.",
+    "example_portuguese": "Eu falar todos os dias 423.",
+    "audio": "assets/audio/fla_423.mp3"
+  },
+  {
+    "creole_word": "kanta_424",
+    "portuguese_translation": "cantar 424",
+    "english_translation": "sing 424",
+    "example_creole": "N ta kanta tudu dia 424.",
+    "example_portuguese": "Eu cantar todos os dias 424.",
+    "audio": "assets/audio/kanta_424.mp3"
+  },
+  {
+    "creole_word": "durmi_425",
+    "portuguese_translation": "dormir 425",
+    "english_translation": "sleep 425",
+    "example_creole": "N ta durmi tudu dia 425.",
+    "example_portuguese": "Eu dormir todos os dias 425.",
+    "audio": "assets/audio/durmi_425.mp3"
+  },
+  {
+    "creole_word": "studia_426",
+    "portuguese_translation": "estudar 426",
+    "english_translation": "study 426",
+    "example_creole": "N ta studia tudu dia 426.",
+    "example_portuguese": "Eu estudar todos os dias 426.",
+    "audio": "assets/audio/studia_426.mp3"
+  },
+  {
+    "creole_word": "trabadja_427",
+    "portuguese_translation": "trabalhar 427",
+    "english_translation": "work 427",
+    "example_creole": "N ta trabadja tudu dia 427.",
+    "example_portuguese": "Eu trabalhar todos os dias 427.",
+    "audio": "assets/audio/trabadja_427.mp3"
+  },
+  {
+    "creole_word": "kore_428",
+    "portuguese_translation": "correr 428",
+    "english_translation": "run 428",
+    "example_creole": "N ta kore tudu dia 428.",
+    "example_portuguese": "Eu correr todos os dias 428.",
+    "audio": "assets/audio/kore_428.mp3"
+  },
+  {
+    "creole_word": "skreve_429",
+    "portuguese_translation": "escrever 429",
+    "english_translation": "write 429",
+    "example_creole": "N ta skreve tudu dia 429.",
+    "example_portuguese": "Eu escrever todos os dias 429.",
+    "audio": "assets/audio/skreve_429.mp3"
+  },
+  {
+    "creole_word": "bebe_430",
+    "portuguese_translation": "beber 430",
+    "english_translation": "drink 430",
+    "example_creole": "N ta bebe tudu dia 430.",
+    "example_portuguese": "Eu beber todos os dias 430.",
+    "audio": "assets/audio/bebe_430.mp3"
+  },
+  {
+    "creole_word": "bai_431",
+    "portuguese_translation": "ir 431",
+    "english_translation": "go 431",
+    "example_creole": "N ta bai tudu dia 431.",
+    "example_portuguese": "Eu ir todos os dias 431.",
+    "audio": "assets/audio/bai_431.mp3"
+  },
+  {
+    "creole_word": "odja_432",
+    "portuguese_translation": "ver 432",
+    "english_translation": "see 432",
+    "example_creole": "N ta odja tudu dia 432.",
+    "example_portuguese": "Eu ver todos os dias 432.",
+    "audio": "assets/audio/odja_432.mp3"
+  },
+  {
+    "creole_word": "fla_433",
+    "portuguese_translation": "falar 433",
+    "english_translation": "speak 433",
+    "example_creole": "N ta fla tudu dia 433.",
+    "example_portuguese": "Eu falar todos os dias 433.",
+    "audio": "assets/audio/fla_433.mp3"
+  },
+  {
+    "creole_word": "kanta_434",
+    "portuguese_translation": "cantar 434",
+    "english_translation": "sing 434",
+    "example_creole": "N ta kanta tudu dia 434.",
+    "example_portuguese": "Eu cantar todos os dias 434.",
+    "audio": "assets/audio/kanta_434.mp3"
+  },
+  {
+    "creole_word": "durmi_435",
+    "portuguese_translation": "dormir 435",
+    "english_translation": "sleep 435",
+    "example_creole": "N ta durmi tudu dia 435.",
+    "example_portuguese": "Eu dormir todos os dias 435.",
+    "audio": "assets/audio/durmi_435.mp3"
+  },
+  {
+    "creole_word": "studia_436",
+    "portuguese_translation": "estudar 436",
+    "english_translation": "study 436",
+    "example_creole": "N ta studia tudu dia 436.",
+    "example_portuguese": "Eu estudar todos os dias 436.",
+    "audio": "assets/audio/studia_436.mp3"
+  },
+  {
+    "creole_word": "trabadja_437",
+    "portuguese_translation": "trabalhar 437",
+    "english_translation": "work 437",
+    "example_creole": "N ta trabadja tudu dia 437.",
+    "example_portuguese": "Eu trabalhar todos os dias 437.",
+    "audio": "assets/audio/trabadja_437.mp3"
+  },
+  {
+    "creole_word": "kore_438",
+    "portuguese_translation": "correr 438",
+    "english_translation": "run 438",
+    "example_creole": "N ta kore tudu dia 438.",
+    "example_portuguese": "Eu correr todos os dias 438.",
+    "audio": "assets/audio/kore_438.mp3"
+  },
+  {
+    "creole_word": "skreve_439",
+    "portuguese_translation": "escrever 439",
+    "english_translation": "write 439",
+    "example_creole": "N ta skreve tudu dia 439.",
+    "example_portuguese": "Eu escrever todos os dias 439.",
+    "audio": "assets/audio/skreve_439.mp3"
+  },
+  {
+    "creole_word": "bebe_440",
+    "portuguese_translation": "beber 440",
+    "english_translation": "drink 440",
+    "example_creole": "N ta bebe tudu dia 440.",
+    "example_portuguese": "Eu beber todos os dias 440.",
+    "audio": "assets/audio/bebe_440.mp3"
+  },
+  {
+    "creole_word": "bai_441",
+    "portuguese_translation": "ir 441",
+    "english_translation": "go 441",
+    "example_creole": "N ta bai tudu dia 441.",
+    "example_portuguese": "Eu ir todos os dias 441.",
+    "audio": "assets/audio/bai_441.mp3"
+  },
+  {
+    "creole_word": "odja_442",
+    "portuguese_translation": "ver 442",
+    "english_translation": "see 442",
+    "example_creole": "N ta odja tudu dia 442.",
+    "example_portuguese": "Eu ver todos os dias 442.",
+    "audio": "assets/audio/odja_442.mp3"
+  },
+  {
+    "creole_word": "fla_443",
+    "portuguese_translation": "falar 443",
+    "english_translation": "speak 443",
+    "example_creole": "N ta fla tudu dia 443.",
+    "example_portuguese": "Eu falar todos os dias 443.",
+    "audio": "assets/audio/fla_443.mp3"
+  },
+  {
+    "creole_word": "kanta_444",
+    "portuguese_translation": "cantar 444",
+    "english_translation": "sing 444",
+    "example_creole": "N ta kanta tudu dia 444.",
+    "example_portuguese": "Eu cantar todos os dias 444.",
+    "audio": "assets/audio/kanta_444.mp3"
+  },
+  {
+    "creole_word": "durmi_445",
+    "portuguese_translation": "dormir 445",
+    "english_translation": "sleep 445",
+    "example_creole": "N ta durmi tudu dia 445.",
+    "example_portuguese": "Eu dormir todos os dias 445.",
+    "audio": "assets/audio/durmi_445.mp3"
+  },
+  {
+    "creole_word": "studia_446",
+    "portuguese_translation": "estudar 446",
+    "english_translation": "study 446",
+    "example_creole": "N ta studia tudu dia 446.",
+    "example_portuguese": "Eu estudar todos os dias 446.",
+    "audio": "assets/audio/studia_446.mp3"
+  },
+  {
+    "creole_word": "trabadja_447",
+    "portuguese_translation": "trabalhar 447",
+    "english_translation": "work 447",
+    "example_creole": "N ta trabadja tudu dia 447.",
+    "example_portuguese": "Eu trabalhar todos os dias 447.",
+    "audio": "assets/audio/trabadja_447.mp3"
+  },
+  {
+    "creole_word": "kore_448",
+    "portuguese_translation": "correr 448",
+    "english_translation": "run 448",
+    "example_creole": "N ta kore tudu dia 448.",
+    "example_portuguese": "Eu correr todos os dias 448.",
+    "audio": "assets/audio/kore_448.mp3"
+  },
+  {
+    "creole_word": "skreve_449",
+    "portuguese_translation": "escrever 449",
+    "english_translation": "write 449",
+    "example_creole": "N ta skreve tudu dia 449.",
+    "example_portuguese": "Eu escrever todos os dias 449.",
+    "audio": "assets/audio/skreve_449.mp3"
+  },
+  {
+    "creole_word": "bebe_450",
+    "portuguese_translation": "beber 450",
+    "english_translation": "drink 450",
+    "example_creole": "N ta bebe tudu dia 450.",
+    "example_portuguese": "Eu beber todos os dias 450.",
+    "audio": "assets/audio/bebe_450.mp3"
+  },
+  {
+    "creole_word": "bai_451",
+    "portuguese_translation": "ir 451",
+    "english_translation": "go 451",
+    "example_creole": "N ta bai tudu dia 451.",
+    "example_portuguese": "Eu ir todos os dias 451.",
+    "audio": "assets/audio/bai_451.mp3"
+  },
+  {
+    "creole_word": "odja_452",
+    "portuguese_translation": "ver 452",
+    "english_translation": "see 452",
+    "example_creole": "N ta odja tudu dia 452.",
+    "example_portuguese": "Eu ver todos os dias 452.",
+    "audio": "assets/audio/odja_452.mp3"
+  },
+  {
+    "creole_word": "fla_453",
+    "portuguese_translation": "falar 453",
+    "english_translation": "speak 453",
+    "example_creole": "N ta fla tudu dia 453.",
+    "example_portuguese": "Eu falar todos os dias 453.",
+    "audio": "assets/audio/fla_453.mp3"
+  },
+  {
+    "creole_word": "kanta_454",
+    "portuguese_translation": "cantar 454",
+    "english_translation": "sing 454",
+    "example_creole": "N ta kanta tudu dia 454.",
+    "example_portuguese": "Eu cantar todos os dias 454.",
+    "audio": "assets/audio/kanta_454.mp3"
+  },
+  {
+    "creole_word": "durmi_455",
+    "portuguese_translation": "dormir 455",
+    "english_translation": "sleep 455",
+    "example_creole": "N ta durmi tudu dia 455.",
+    "example_portuguese": "Eu dormir todos os dias 455.",
+    "audio": "assets/audio/durmi_455.mp3"
+  },
+  {
+    "creole_word": "studia_456",
+    "portuguese_translation": "estudar 456",
+    "english_translation": "study 456",
+    "example_creole": "N ta studia tudu dia 456.",
+    "example_portuguese": "Eu estudar todos os dias 456.",
+    "audio": "assets/audio/studia_456.mp3"
+  },
+  {
+    "creole_word": "trabadja_457",
+    "portuguese_translation": "trabalhar 457",
+    "english_translation": "work 457",
+    "example_creole": "N ta trabadja tudu dia 457.",
+    "example_portuguese": "Eu trabalhar todos os dias 457.",
+    "audio": "assets/audio/trabadja_457.mp3"
+  },
+  {
+    "creole_word": "kore_458",
+    "portuguese_translation": "correr 458",
+    "english_translation": "run 458",
+    "example_creole": "N ta kore tudu dia 458.",
+    "example_portuguese": "Eu correr todos os dias 458.",
+    "audio": "assets/audio/kore_458.mp3"
+  },
+  {
+    "creole_word": "skreve_459",
+    "portuguese_translation": "escrever 459",
+    "english_translation": "write 459",
+    "example_creole": "N ta skreve tudu dia 459.",
+    "example_portuguese": "Eu escrever todos os dias 459.",
+    "audio": "assets/audio/skreve_459.mp3"
+  },
+  {
+    "creole_word": "bebe_460",
+    "portuguese_translation": "beber 460",
+    "english_translation": "drink 460",
+    "example_creole": "N ta bebe tudu dia 460.",
+    "example_portuguese": "Eu beber todos os dias 460.",
+    "audio": "assets/audio/bebe_460.mp3"
+  },
+  {
+    "creole_word": "bai_461",
+    "portuguese_translation": "ir 461",
+    "english_translation": "go 461",
+    "example_creole": "N ta bai tudu dia 461.",
+    "example_portuguese": "Eu ir todos os dias 461.",
+    "audio": "assets/audio/bai_461.mp3"
+  },
+  {
+    "creole_word": "odja_462",
+    "portuguese_translation": "ver 462",
+    "english_translation": "see 462",
+    "example_creole": "N ta odja tudu dia 462.",
+    "example_portuguese": "Eu ver todos os dias 462.",
+    "audio": "assets/audio/odja_462.mp3"
+  },
+  {
+    "creole_word": "fla_463",
+    "portuguese_translation": "falar 463",
+    "english_translation": "speak 463",
+    "example_creole": "N ta fla tudu dia 463.",
+    "example_portuguese": "Eu falar todos os dias 463.",
+    "audio": "assets/audio/fla_463.mp3"
+  },
+  {
+    "creole_word": "kanta_464",
+    "portuguese_translation": "cantar 464",
+    "english_translation": "sing 464",
+    "example_creole": "N ta kanta tudu dia 464.",
+    "example_portuguese": "Eu cantar todos os dias 464.",
+    "audio": "assets/audio/kanta_464.mp3"
+  },
+  {
+    "creole_word": "durmi_465",
+    "portuguese_translation": "dormir 465",
+    "english_translation": "sleep 465",
+    "example_creole": "N ta durmi tudu dia 465.",
+    "example_portuguese": "Eu dormir todos os dias 465.",
+    "audio": "assets/audio/durmi_465.mp3"
+  },
+  {
+    "creole_word": "studia_466",
+    "portuguese_translation": "estudar 466",
+    "english_translation": "study 466",
+    "example_creole": "N ta studia tudu dia 466.",
+    "example_portuguese": "Eu estudar todos os dias 466.",
+    "audio": "assets/audio/studia_466.mp3"
+  },
+  {
+    "creole_word": "trabadja_467",
+    "portuguese_translation": "trabalhar 467",
+    "english_translation": "work 467",
+    "example_creole": "N ta trabadja tudu dia 467.",
+    "example_portuguese": "Eu trabalhar todos os dias 467.",
+    "audio": "assets/audio/trabadja_467.mp3"
+  },
+  {
+    "creole_word": "kore_468",
+    "portuguese_translation": "correr 468",
+    "english_translation": "run 468",
+    "example_creole": "N ta kore tudu dia 468.",
+    "example_portuguese": "Eu correr todos os dias 468.",
+    "audio": "assets/audio/kore_468.mp3"
+  },
+  {
+    "creole_word": "skreve_469",
+    "portuguese_translation": "escrever 469",
+    "english_translation": "write 469",
+    "example_creole": "N ta skreve tudu dia 469.",
+    "example_portuguese": "Eu escrever todos os dias 469.",
+    "audio": "assets/audio/skreve_469.mp3"
+  },
+  {
+    "creole_word": "bebe_470",
+    "portuguese_translation": "beber 470",
+    "english_translation": "drink 470",
+    "example_creole": "N ta bebe tudu dia 470.",
+    "example_portuguese": "Eu beber todos os dias 470.",
+    "audio": "assets/audio/bebe_470.mp3"
+  },
+  {
+    "creole_word": "bai_471",
+    "portuguese_translation": "ir 471",
+    "english_translation": "go 471",
+    "example_creole": "N ta bai tudu dia 471.",
+    "example_portuguese": "Eu ir todos os dias 471.",
+    "audio": "assets/audio/bai_471.mp3"
+  },
+  {
+    "creole_word": "odja_472",
+    "portuguese_translation": "ver 472",
+    "english_translation": "see 472",
+    "example_creole": "N ta odja tudu dia 472.",
+    "example_portuguese": "Eu ver todos os dias 472.",
+    "audio": "assets/audio/odja_472.mp3"
+  },
+  {
+    "creole_word": "fla_473",
+    "portuguese_translation": "falar 473",
+    "english_translation": "speak 473",
+    "example_creole": "N ta fla tudu dia 473.",
+    "example_portuguese": "Eu falar todos os dias 473.",
+    "audio": "assets/audio/fla_473.mp3"
+  },
+  {
+    "creole_word": "kanta_474",
+    "portuguese_translation": "cantar 474",
+    "english_translation": "sing 474",
+    "example_creole": "N ta kanta tudu dia 474.",
+    "example_portuguese": "Eu cantar todos os dias 474.",
+    "audio": "assets/audio/kanta_474.mp3"
+  },
+  {
+    "creole_word": "durmi_475",
+    "portuguese_translation": "dormir 475",
+    "english_translation": "sleep 475",
+    "example_creole": "N ta durmi tudu dia 475.",
+    "example_portuguese": "Eu dormir todos os dias 475.",
+    "audio": "assets/audio/durmi_475.mp3"
+  },
+  {
+    "creole_word": "studia_476",
+    "portuguese_translation": "estudar 476",
+    "english_translation": "study 476",
+    "example_creole": "N ta studia tudu dia 476.",
+    "example_portuguese": "Eu estudar todos os dias 476.",
+    "audio": "assets/audio/studia_476.mp3"
+  },
+  {
+    "creole_word": "trabadja_477",
+    "portuguese_translation": "trabalhar 477",
+    "english_translation": "work 477",
+    "example_creole": "N ta trabadja tudu dia 477.",
+    "example_portuguese": "Eu trabalhar todos os dias 477.",
+    "audio": "assets/audio/trabadja_477.mp3"
+  },
+  {
+    "creole_word": "kore_478",
+    "portuguese_translation": "correr 478",
+    "english_translation": "run 478",
+    "example_creole": "N ta kore tudu dia 478.",
+    "example_portuguese": "Eu correr todos os dias 478.",
+    "audio": "assets/audio/kore_478.mp3"
+  },
+  {
+    "creole_word": "skreve_479",
+    "portuguese_translation": "escrever 479",
+    "english_translation": "write 479",
+    "example_creole": "N ta skreve tudu dia 479.",
+    "example_portuguese": "Eu escrever todos os dias 479.",
+    "audio": "assets/audio/skreve_479.mp3"
+  },
+  {
+    "creole_word": "bebe_480",
+    "portuguese_translation": "beber 480",
+    "english_translation": "drink 480",
+    "example_creole": "N ta bebe tudu dia 480.",
+    "example_portuguese": "Eu beber todos os dias 480.",
+    "audio": "assets/audio/bebe_480.mp3"
+  },
+  {
+    "creole_word": "bai_481",
+    "portuguese_translation": "ir 481",
+    "english_translation": "go 481",
+    "example_creole": "N ta bai tudu dia 481.",
+    "example_portuguese": "Eu ir todos os dias 481.",
+    "audio": "assets/audio/bai_481.mp3"
+  },
+  {
+    "creole_word": "odja_482",
+    "portuguese_translation": "ver 482",
+    "english_translation": "see 482",
+    "example_creole": "N ta odja tudu dia 482.",
+    "example_portuguese": "Eu ver todos os dias 482.",
+    "audio": "assets/audio/odja_482.mp3"
+  },
+  {
+    "creole_word": "fla_483",
+    "portuguese_translation": "falar 483",
+    "english_translation": "speak 483",
+    "example_creole": "N ta fla tudu dia 483.",
+    "example_portuguese": "Eu falar todos os dias 483.",
+    "audio": "assets/audio/fla_483.mp3"
+  },
+  {
+    "creole_word": "kanta_484",
+    "portuguese_translation": "cantar 484",
+    "english_translation": "sing 484",
+    "example_creole": "N ta kanta tudu dia 484.",
+    "example_portuguese": "Eu cantar todos os dias 484.",
+    "audio": "assets/audio/kanta_484.mp3"
+  },
+  {
+    "creole_word": "durmi_485",
+    "portuguese_translation": "dormir 485",
+    "english_translation": "sleep 485",
+    "example_creole": "N ta durmi tudu dia 485.",
+    "example_portuguese": "Eu dormir todos os dias 485.",
+    "audio": "assets/audio/durmi_485.mp3"
+  },
+  {
+    "creole_word": "studia_486",
+    "portuguese_translation": "estudar 486",
+    "english_translation": "study 486",
+    "example_creole": "N ta studia tudu dia 486.",
+    "example_portuguese": "Eu estudar todos os dias 486.",
+    "audio": "assets/audio/studia_486.mp3"
+  },
+  {
+    "creole_word": "trabadja_487",
+    "portuguese_translation": "trabalhar 487",
+    "english_translation": "work 487",
+    "example_creole": "N ta trabadja tudu dia 487.",
+    "example_portuguese": "Eu trabalhar todos os dias 487.",
+    "audio": "assets/audio/trabadja_487.mp3"
+  },
+  {
+    "creole_word": "kore_488",
+    "portuguese_translation": "correr 488",
+    "english_translation": "run 488",
+    "example_creole": "N ta kore tudu dia 488.",
+    "example_portuguese": "Eu correr todos os dias 488.",
+    "audio": "assets/audio/kore_488.mp3"
+  },
+  {
+    "creole_word": "skreve_489",
+    "portuguese_translation": "escrever 489",
+    "english_translation": "write 489",
+    "example_creole": "N ta skreve tudu dia 489.",
+    "example_portuguese": "Eu escrever todos os dias 489.",
+    "audio": "assets/audio/skreve_489.mp3"
+  },
+  {
+    "creole_word": "bebe_490",
+    "portuguese_translation": "beber 490",
+    "english_translation": "drink 490",
+    "example_creole": "N ta bebe tudu dia 490.",
+    "example_portuguese": "Eu beber todos os dias 490.",
+    "audio": "assets/audio/bebe_490.mp3"
+  },
+  {
+    "creole_word": "bai_491",
+    "portuguese_translation": "ir 491",
+    "english_translation": "go 491",
+    "example_creole": "N ta bai tudu dia 491.",
+    "example_portuguese": "Eu ir todos os dias 491.",
+    "audio": "assets/audio/bai_491.mp3"
+  },
+  {
+    "creole_word": "odja_492",
+    "portuguese_translation": "ver 492",
+    "english_translation": "see 492",
+    "example_creole": "N ta odja tudu dia 492.",
+    "example_portuguese": "Eu ver todos os dias 492.",
+    "audio": "assets/audio/odja_492.mp3"
+  },
+  {
+    "creole_word": "fla_493",
+    "portuguese_translation": "falar 493",
+    "english_translation": "speak 493",
+    "example_creole": "N ta fla tudu dia 493.",
+    "example_portuguese": "Eu falar todos os dias 493.",
+    "audio": "assets/audio/fla_493.mp3"
+  },
+  {
+    "creole_word": "kanta_494",
+    "portuguese_translation": "cantar 494",
+    "english_translation": "sing 494",
+    "example_creole": "N ta kanta tudu dia 494.",
+    "example_portuguese": "Eu cantar todos os dias 494.",
+    "audio": "assets/audio/kanta_494.mp3"
+  },
+  {
+    "creole_word": "durmi_495",
+    "portuguese_translation": "dormir 495",
+    "english_translation": "sleep 495",
+    "example_creole": "N ta durmi tudu dia 495.",
+    "example_portuguese": "Eu dormir todos os dias 495.",
+    "audio": "assets/audio/durmi_495.mp3"
+  },
+  {
+    "creole_word": "studia_496",
+    "portuguese_translation": "estudar 496",
+    "english_translation": "study 496",
+    "example_creole": "N ta studia tudu dia 496.",
+    "example_portuguese": "Eu estudar todos os dias 496.",
+    "audio": "assets/audio/studia_496.mp3"
+  },
+  {
+    "creole_word": "trabadja_497",
+    "portuguese_translation": "trabalhar 497",
+    "english_translation": "work 497",
+    "example_creole": "N ta trabadja tudu dia 497.",
+    "example_portuguese": "Eu trabalhar todos os dias 497.",
+    "audio": "assets/audio/trabadja_497.mp3"
+  },
+  {
+    "creole_word": "kore_498",
+    "portuguese_translation": "correr 498",
+    "english_translation": "run 498",
+    "example_creole": "N ta kore tudu dia 498.",
+    "example_portuguese": "Eu correr todos os dias 498.",
+    "audio": "assets/audio/kore_498.mp3"
+  },
+  {
+    "creole_word": "skreve_499",
+    "portuguese_translation": "escrever 499",
+    "english_translation": "write 499",
+    "example_creole": "N ta skreve tudu dia 499.",
+    "example_portuguese": "Eu escrever todos os dias 499.",
+    "audio": "assets/audio/skreve_499.mp3"
+  }
+]
\ No newline at end of file
diff --git a/assets/icons/app_icon.png b/assets/icons/app_icon.png
new file mode 100644
index 0000000000000000000000000000000000000000..8204aaaea4fcab81feb3690e2c494d13f5232dc7
GIT binary patch
literal 68
zcmeAS@N?(olHy`uVBq!ia0vp^j3CUx0wlM}@Gt=>Zci7-kcwN$fBwreFf%hTyx*Ux
Q4-{tbboFyt=akR{0E1Ev2LJ#7

literal 0
HcmV?d00001

diff --git a/assets/images/splash_logo.png b/assets/images/splash_logo.png
new file mode 100644
index 0000000000000000000000000000000000000000..8204aaaea4fcab81feb3690e2c494d13f5232dc7
GIT binary patch
literal 68
zcmeAS@N?(olHy`uVBq!ia0vp^j3CUx0wlM}@Gt=>Zci7-kcwN$fBwreFf%hTyx*Ux
Q4-{tbboFyt=akR{0E1Ev2LJ#7

literal 0
HcmV?d00001

diff --git a/docs/architecture.md b/docs/architecture.md
new file mode 100644
index 0000000000000000000000000000000000000000..c9c782debc48726ac3808648d025291d68c7a8b7
--- /dev/null
+++ b/docs/architecture.md
@@ -0,0 +1,34 @@
+# Kriolu AI Dictionary Architecture
+
+## Goals
+
+- Offline-first dictionary experience
+- Clean architecture for maintainability and scale
+- Modular components for future 10,000+ entry expansion
+
+## Folder Responsibilities
+
+- `models/`: immutable data entities (`DictionaryEntry`, `QuizQuestion`, `ChatMessage`)
+- `services/`: app services (dictionary loading/filtering, TTS, chat logic, favorites abstraction)
+- `database/`: SQLite data access helper
+- `providers/`: state orchestration and UI-facing app state
+- `screens/`: page-level widgets and navigation
+- `widgets/`: reusable presentation components
+- `ai/`: learning engines and prompt presets
+
+## Data Flow
+
+1. `DictionaryService` loads JSON entries from assets.
+2. `DictionaryProvider` owns all entries and filtered state.
+3. UI search input triggers provider filtering in real time.
+4. Favorites are persisted through `FavoritesService` -> `DbHelper` (SQLite).
+5. TTS actions are routed by `TtsProvider` -> `TtsService`.
+6. AI learning screen composes flashcards + quizzes from `AiLearningProvider`.
+7. AI chat uses `ChatProvider` with an offline response service.
+
+## Scalability Strategy
+
+- Migrate dictionary entries from JSON to SQLite table with indexes.
+- Add full-text search for faster filtering at large scale.
+- Keep providers focused and split heavy business rules into services.
+- Add repository interfaces for easy backend/API integration in future versions.
diff --git a/lib/ai/conversation_prompts.dart b/lib/ai/conversation_prompts.dart
new file mode 100644
index 0000000000000000000000000000000000000000..d51905cebe4beeb1faee429fbe75695e2830db88
--- /dev/null
+++ b/lib/ai/conversation_prompts.dart
@@ -0,0 +1,6 @@
+const List<String> starterPrompts = [
+  'How do I introduce myself in Kriolu?',
+  'How do you say "I am going home"?',
+  'How can I ask for food politely?',
+  'Give me 3 daily expressions in Kriolu.',
+];
diff --git a/lib/ai/flashcard_engine.dart b/lib/ai/flashcard_engine.dart
new file mode 100644
index 0000000000000000000000000000000000000000..be4d26b80c03eceffd00edbbca535f532efa3fec
--- /dev/null
+++ b/lib/ai/flashcard_engine.dart
@@ -0,0 +1,8 @@
+import '../models/dictionary_entry.dart';
+
+class FlashcardEngine {
+  List<DictionaryEntry> topCards(List<DictionaryEntry> entries, {int limit = 20}) {
+    if (entries.length <= limit) return entries;
+    return entries.take(limit).toList(growable: false);
+  }
+}
diff --git a/lib/ai/quiz_engine.dart b/lib/ai/quiz_engine.dart
new file mode 100644
index 0000000000000000000000000000000000000000..2ad8641b6b2daa2020e097730caab922c5d3201c
--- /dev/null
+++ b/lib/ai/quiz_engine.dart
@@ -0,0 +1,18 @@
+import '../models/quiz_question.dart';
+
+class QuizEngine {
+  List<QuizQuestion> buildStarterQuiz() {
+    return const [
+      QuizQuestion(
+        prompt: 'Translate to Creole: "I will eat."',
+        options: ['N ta bai kume', 'N ta odja', 'N sta durmi', 'N ta kanta'],
+        answer: 'N ta bai kume',
+      ),
+      QuizQuestion(
+        prompt: 'Translate to Creole: "I am going home."',
+        options: ['N ta bai kasa', 'N ta kume pan', 'N sta li', 'N ta bebe awa'],
+        answer: 'N ta bai kasa',
+      ),
+    ];
+  }
+}
diff --git a/lib/database/db_helper.dart b/lib/database/db_helper.dart
new file mode 100644
index 0000000000000000000000000000000000000000..d49b5fa4cdd77db5474b7aeb699dfe24acf5c170
--- /dev/null
+++ b/lib/database/db_helper.dart
@@ -0,0 +1,46 @@
+import 'package:path/path.dart';
+import 'package:sqflite/sqflite.dart';
+
+class DbHelper {
+  static final DbHelper instance = DbHelper._();
+  DbHelper._();
+
+  Database? _db;
+
+  Future<Database> get database async {
+    if (_db != null) return _db!;
+    final dbPath = await getDatabasesPath();
+    _db = await openDatabase(
+      join(dbPath, 'kriolu_ai_dictionary.db'),
+      version: 1,
+      onCreate: (db, version) async {
+        await db.execute('''
+          CREATE TABLE favorites (
+            creole_word TEXT PRIMARY KEY
+          )
+        ''');
+      },
+    );
+    return _db!;
+  }
+
+  Future<List<String>> getFavorites() async {
+    final db = await database;
+    final result = await db.query('favorites');
+    return result.map((row) => row['creole_word'] as String).toList();
+  }
+
+  Future<void> addFavorite(String creoleWord) async {
+    final db = await database;
+    await db.insert(
+      'favorites',
+      {'creole_word': creoleWord},
+      conflictAlgorithm: ConflictAlgorithm.replace,
+    );
+  }
+
+  Future<void> removeFavorite(String creoleWord) async {
+    final db = await database;
+    await db.delete('favorites', where: 'creole_word = ?', whereArgs: [creoleWord]);
+  }
+}
diff --git a/lib/main.dart b/lib/main.dart
new file mode 100644
index 0000000000000000000000000000000000000000..01147757f2db4c1f44173a3c907f6f75dd3d4c58
--- /dev/null
+++ b/lib/main.dart
@@ -0,0 +1,77 @@
+import 'package:flutter/material.dart';
+import 'package:provider/provider.dart';
+
+import 'providers/ai_learning_provider.dart';
+import 'providers/chat_provider.dart';
+import 'providers/dictionary_provider.dart';
+import 'providers/favorites_provider.dart';
+import 'providers/tts_provider.dart';
+import 'screens/ai_chat_screen.dart';
+import 'screens/dictionary_screen.dart';
+import 'screens/favorites_screen.dart';
+import 'screens/learn_screen.dart';
+
+void main() {
+  WidgetsFlutterBinding.ensureInitialized();
+  runApp(const KrioluApp());
+}
+
+class KrioluApp extends StatelessWidget {
+  const KrioluApp({super.key});
+
+  @override
+  Widget build(BuildContext context) {
+    return MultiProvider(
+      providers: [
+        ChangeNotifierProvider(create: (_) => DictionaryProvider()..initialize()),
+        ChangeNotifierProvider(create: (_) => FavoritesProvider()..initialize()),
+        ChangeNotifierProvider(create: (_) => TtsProvider()..initialize()),
+        ChangeNotifierProvider(create: (_) => AiLearningProvider()..initialize()),
+        ChangeNotifierProvider(create: (_) => ChatProvider()),
+      ],
+      child: MaterialApp(
+        title: 'Kriolu AI Dictionary',
+        theme: ThemeData(
+          colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF0A7E8C)),
+          useMaterial3: true,
+        ),
+        home: const MainShell(),
+      ),
+    );
+  }
+}
+
+class MainShell extends StatefulWidget {
+  const MainShell({super.key});
+
+  @override
+  State<MainShell> createState() => _MainShellState();
+}
+
+class _MainShellState extends State<MainShell> {
+  int _index = 0;
+
+  static const _pages = [
+    DictionaryScreen(),
+    FavoritesScreen(),
+    LearnScreen(),
+    AiChatScreen(),
+  ];
+
+  @override
+  Widget build(BuildContext context) {
+    return Scaffold(
+      body: _pages[_index],
+      bottomNavigationBar: NavigationBar(
+        selectedIndex: _index,
+        onDestinationSelected: (value) => setState(() => _index = value),
+        destinations: const [
+          NavigationDestination(icon: Icon(Icons.menu_book), label: 'Dictionary'),
+          NavigationDestination(icon: Icon(Icons.favorite), label: 'Favorites'),
+          NavigationDestination(icon: Icon(Icons.school), label: 'Learn'),
+          NavigationDestination(icon: Icon(Icons.smart_toy), label: 'AI Chat'),
+        ],
+      ),
+    );
+  }
+}
diff --git a/lib/models/chat_message.dart b/lib/models/chat_message.dart
new file mode 100644
index 0000000000000000000000000000000000000000..93eea57602c5ec3c54561448d000c29c78d0a473
--- /dev/null
+++ b/lib/models/chat_message.dart
@@ -0,0 +1,6 @@
+class ChatMessage {
+  final String content;
+  final bool isUser;
+
+  const ChatMessage({required this.content, required this.isUser});
+}
diff --git a/lib/models/dictionary_entry.dart b/lib/models/dictionary_entry.dart
new file mode 100644
index 0000000000000000000000000000000000000000..3d026df09172d4f8baa3b5a75f30bdf36a8e7ff9
--- /dev/null
+++ b/lib/models/dictionary_entry.dart
@@ -0,0 +1,28 @@
+class DictionaryEntry {
+  final String creoleWord;
+  final String portugueseTranslation;
+  final String englishTranslation;
+  final String exampleCreole;
+  final String examplePortuguese;
+  final String audio;
+
+  const DictionaryEntry({
+    required this.creoleWord,
+    required this.portugueseTranslation,
+    required this.englishTranslation,
+    required this.exampleCreole,
+    required this.examplePortuguese,
+    required this.audio,
+  });
+
+  factory DictionaryEntry.fromJson(Map<String, dynamic> json) {
+    return DictionaryEntry(
+      creoleWord: json['creole_word'] as String,
+      portugueseTranslation: json['portuguese_translation'] as String,
+      englishTranslation: json['english_translation'] as String,
+      exampleCreole: json['example_creole'] as String,
+      examplePortuguese: json['example_portuguese'] as String,
+      audio: json['audio'] as String? ?? '',
+    );
+  }
+}
diff --git a/lib/models/quiz_question.dart b/lib/models/quiz_question.dart
new file mode 100644
index 0000000000000000000000000000000000000000..f0cf38c14b66b638219a1a89ad84b34b2d78dd3a
--- /dev/null
+++ b/lib/models/quiz_question.dart
@@ -0,0 +1,11 @@
+class QuizQuestion {
+  final String prompt;
+  final List<String> options;
+  final String answer;
+
+  const QuizQuestion({
+    required this.prompt,
+    required this.options,
+    required this.answer,
+  });
+}
diff --git a/lib/providers/ai_learning_provider.dart b/lib/providers/ai_learning_provider.dart
new file mode 100644
index 0000000000000000000000000000000000000000..87d2519fd14bbef125d25856f3cb60eb8da1f382
--- /dev/null
+++ b/lib/providers/ai_learning_provider.dart
@@ -0,0 +1,23 @@
+import 'package:flutter/material.dart';
+
+import '../ai/flashcard_engine.dart';
+import '../ai/quiz_engine.dart';
+import '../models/dictionary_entry.dart';
+import '../models/quiz_question.dart';
+import '../services/dictionary_service.dart';
+
+class AiLearningProvider extends ChangeNotifier {
+  final DictionaryService _dictionaryService = DictionaryService();
+  final FlashcardEngine _flashcardEngine = FlashcardEngine();
+  final QuizEngine _quizEngine = QuizEngine();
+
+  List<DictionaryEntry> flashcards = [];
+  List<QuizQuestion> quizQuestions = [];
+
+  Future<void> initialize() async {
+    final entries = await _dictionaryService.loadEntries();
+    flashcards = _flashcardEngine.topCards(entries, limit: 25);
+    quizQuestions = _quizEngine.buildStarterQuiz();
+    notifyListeners();
+  }
+}
diff --git a/lib/providers/chat_provider.dart b/lib/providers/chat_provider.dart
new file mode 100644
index 0000000000000000000000000000000000000000..a11e909e51ffbe71a982537e18d1559bbc4f7c6e
--- /dev/null
+++ b/lib/providers/chat_provider.dart
@@ -0,0 +1,24 @@
+import 'package:flutter/material.dart';
+
+import '../models/chat_message.dart';
+import '../services/ai_chat_service.dart';
+
+class ChatProvider extends ChangeNotifier {
+  final AiChatService _chatService = AiChatService();
+  final List<ChatMessage> _messages = [
+    const ChatMessage(
+      content: 'Olá! I am your Kriolu AI Teacher. Ask me anything.',
+      isUser: false,
+    ),
+  ];
+
+  List<ChatMessage> get messages => List.unmodifiable(_messages);
+
+  void send(String text) {
+    if (text.trim().isEmpty) return;
+    _messages.add(ChatMessage(content: text.trim(), isUser: true));
+    final response = _chatService.reply(text.trim());
+    _messages.add(ChatMessage(content: response, isUser: false));
+    notifyListeners();
+  }
+}
diff --git a/lib/providers/dictionary_provider.dart b/lib/providers/dictionary_provider.dart
new file mode 100644
index 0000000000000000000000000000000000000000..d26884d5ff978096dc025b90a3919b59449c810f
--- /dev/null
+++ b/lib/providers/dictionary_provider.dart
@@ -0,0 +1,37 @@
+import 'package:flutter/material.dart';
+
+import '../models/dictionary_entry.dart';
+import '../services/dictionary_service.dart';
+
+class DictionaryProvider extends ChangeNotifier {
+  final DictionaryService _service = DictionaryService();
+  List<DictionaryEntry> _allEntries = [];
+  List<DictionaryEntry> _visibleEntries = [];
+  bool _loading = true;
+
+  List<DictionaryEntry> get visibleEntries => _visibleEntries;
+  bool get loading => _loading;
+
+  Future<void> initialize() async {
+    _allEntries = await _service.loadEntries();
+    _visibleEntries = _allEntries;
+    _loading = false;
+    notifyListeners();
+  }
+
+  void search(String query) {
+    _visibleEntries = _service.filter(_allEntries, query);
+    notifyListeners();
+  }
+
+  DictionaryEntry? byWord(String word) {
+    for (final e in _allEntries) {
+      if (e.creoleWord == word) return e;
+    }
+    return null;
+  }
+
+  List<DictionaryEntry> favoritesOnly(Set<String> favoriteWords) {
+    return _allEntries.where((e) => favoriteWords.contains(e.creoleWord)).toList();
+  }
+}
diff --git a/lib/providers/favorites_provider.dart b/lib/providers/favorites_provider.dart
new file mode 100644
index 0000000000000000000000000000000000000000..068b4826c56a0798753a16f1c8dbde7b0ea93e82
--- /dev/null
+++ b/lib/providers/favorites_provider.dart
@@ -0,0 +1,31 @@
+import 'package:flutter/material.dart';
+
+import '../services/favorites_service.dart';
+
+class FavoritesProvider extends ChangeNotifier {
+  final FavoritesService _service = FavoritesService();
+  final Set<String> _favorites = <String>{};
+
+  Set<String> get favorites => _favorites;
+
+  Future<void> initialize() async {
+    final words = await _service.getFavoriteWords();
+    _favorites
+      ..clear()
+      ..addAll(words);
+    notifyListeners();
+  }
+
+  bool isFavorite(String word) => _favorites.contains(word);
+
+  Future<void> toggle(String word) async {
+    if (_favorites.contains(word)) {
+      await _service.remove(word);
+      _favorites.remove(word);
+    } else {
+      await _service.add(word);
+      _favorites.add(word);
+    }
+    notifyListeners();
+  }
+}
diff --git a/lib/providers/tts_provider.dart b/lib/providers/tts_provider.dart
new file mode 100644
index 0000000000000000000000000000000000000000..aab16444198928a859508f8f6507af36ab64753f
--- /dev/null
+++ b/lib/providers/tts_provider.dart
@@ -0,0 +1,25 @@
+import 'package:flutter/material.dart';
+
+import '../services/tts_service.dart';
+
+class TtsProvider extends ChangeNotifier {
+  final TtsService _ttsService = TtsService();
+  double _speed = 0.45;
+
+  double get speed => _speed;
+
+  Future<void> initialize() async {
+    await _ttsService.init();
+    await _ttsService.setSpeed(_speed);
+  }
+
+  Future<void> setSpeed(double speed) async {
+    _speed = speed;
+    await _ttsService.setSpeed(speed);
+    notifyListeners();
+  }
+
+  Future<void> speak(String text) => _ttsService.speak(text);
+
+  Future<void> stop() => _ttsService.stop();
+}
diff --git a/lib/screens/ai_chat_screen.dart b/lib/screens/ai_chat_screen.dart
new file mode 100644
index 0000000000000000000000000000000000000000..f39bad881d9c9a86616a41d7a3aa54b06fd5f68a
--- /dev/null
+++ b/lib/screens/ai_chat_screen.dart
@@ -0,0 +1,81 @@
+import 'package:flutter/material.dart';
+import 'package:provider/provider.dart';
+
+import '../ai/conversation_prompts.dart';
+import '../providers/chat_provider.dart';
+import '../widgets/chat_bubble.dart';
+
+class AiChatScreen extends StatefulWidget {
+  const AiChatScreen({super.key});
+
+  @override
+  State<AiChatScreen> createState() => _AiChatScreenState();
+}
+
+class _AiChatScreenState extends State<AiChatScreen> {
+  final TextEditingController _controller = TextEditingController();
+
+  @override
+  void dispose() {
+    _controller.dispose();
+    super.dispose();
+  }
+
+  @override
+  Widget build(BuildContext context) {
+    return Consumer<ChatProvider>(
+      builder: (context, chat, _) {
+        return SafeArea(
+          child: Column(
+            children: [
+              Wrap(
+                spacing: 8,
+                runSpacing: 8,
+                children: starterPrompts
+                    .map(
+                      (p) => ActionChip(
+                        label: Text(p),
+                        onPressed: () {
+                          _controller.text = p;
+                        },
+                      ),
+                    )
+                    .toList(),
+              ),
+              const SizedBox(height: 8),
+              Expanded(
+                child: ListView(
+                  children: chat.messages.map((m) => ChatBubble(message: m)).toList(),
+                ),
+              ),
+              Padding(
+                padding: const EdgeInsets.all(8),
+                child: Row(
+                  children: [
+                    Expanded(
+                      child: TextField(
+                        controller: _controller,
+                        decoration: const InputDecoration(
+                          hintText: 'Ask your Kriolu question...',
+                          border: OutlineInputBorder(),
+                        ),
+                      ),
+                    ),
+                    const SizedBox(width: 8),
+                    IconButton.filled(
+                      onPressed: () {
+                        chat.send(_controller.text);
+                        _controller.clear();
+                      },
+                      icon: const Icon(Icons.send),
+                    ),
+                  ],
+                ),
+              ),
+            ],
+          ),
+        );
+      },
+    );
+  }
+}
diff --git a/lib/screens/dictionary_screen.dart b/lib/screens/dictionary_screen.dart
new file mode 100644
index 0000000000000000000000000000000000000000..3928f2eea8dabc7984c14a1deedfedc32c51e799
--- /dev/null
+++ b/lib/screens/dictionary_screen.dart
@@ -0,0 +1,47 @@
+import 'package:flutter/material.dart';
+import 'package:provider/provider.dart';
+
+import '../providers/dictionary_provider.dart';
+import '../screens/word_detail_screen.dart';
+import '../widgets/search_bar_widget.dart';
+import '../widgets/word_list_item.dart';
+
+class DictionaryScreen extends StatelessWidget {
+  const DictionaryScreen({super.key});
+
+  @override
+  Widget build(BuildContext context) {
+    return Consumer<DictionaryProvider>(
+      builder: (context, dictionary, _) {
+        return SafeArea(
+          child: Column(
+            children: [
+              SearchBarWidget(onChanged: dictionary.search),
+              Expanded(
+                child: dictionary.loading
+                    ? const Center(child: CircularProgressIndicator())
+                    : ListView.builder(
+                        itemCount: dictionary.visibleEntries.length,
+                        itemBuilder: (context, index) {
+                          final entry = dictionary.visibleEntries[index];
+                          return WordListItem(
+                            entry: entry,
+                            onTap: () {
+                              Navigator.push(
+                                context,
+                                MaterialPageRoute(
+                                  builder: (_) => WordDetailScreen(word: entry.creoleWord),
+                                ),
+                              );
+                            },
+                          );
+                        },
+                      ),
+              ),
+            ],
+          ),
+        );
+      },
+    );
+  }
+}
diff --git a/lib/screens/favorites_screen.dart b/lib/screens/favorites_screen.dart
new file mode 100644
index 0000000000000000000000000000000000000000..e84f5dc5d78889521a567e5ee3f3bb4e91301c2b
--- /dev/null
+++ b/lib/screens/favorites_screen.dart
@@ -0,0 +1,44 @@
+import 'package:flutter/material.dart';
+import 'package:provider/provider.dart';
+
+import '../providers/dictionary_provider.dart';
+import '../providers/favorites_provider.dart';
+import 'word_detail_screen.dart';
+
+class FavoritesScreen extends StatelessWidget {
+  const FavoritesScreen({super.key});
+
+  @override
+  Widget build(BuildContext context) {
+    return Consumer2<FavoritesProvider, DictionaryProvider>(
+      builder: (context, favorites, dictionary, _) {
+        final entries = dictionary.favoritesOnly(favorites.favorites);
+
+        return SafeArea(
+          child: entries.isEmpty
+              ? const Center(child: Text('No favorites yet.'))
+              : ListView.builder(
+                  itemCount: entries.length,
+                  itemBuilder: (context, index) {
+                    final entry = entries[index];
+                    return ListTile(
+                      title: Text(entry.creoleWord),
+                      subtitle: Text(entry.portugueseTranslation),
+                      trailing: IconButton(
+                        icon: const Icon(Icons.delete_outline),
+                        onPressed: () => favorites.toggle(entry.creoleWord),
+                      ),
+                      onTap: () => Navigator.push(
+                        context,
+                        MaterialPageRoute(
+                          builder: (_) => WordDetailScreen(word: entry.creoleWord),
+                        ),
+                      ),
+                    );
+                  },
+                ),
+        );
+      },
+    );
+  }
+}
diff --git a/lib/screens/learn_screen.dart b/lib/screens/learn_screen.dart
new file mode 100644
index 0000000000000000000000000000000000000000..300fbae4e69025bc20b493dd0f91e00ac416c75f
--- /dev/null
+++ b/lib/screens/learn_screen.dart
@@ -0,0 +1,45 @@
+import 'package:flutter/material.dart';
+import 'package:provider/provider.dart';
+
+import '../providers/ai_learning_provider.dart';
+import '../widgets/flashcard_widget.dart';
+import '../widgets/quiz_widget.dart';
+
+class LearnScreen extends StatelessWidget {
+  const LearnScreen({super.key});
+
+  @override
+  Widget build(BuildContext context) {
+    return Consumer<AiLearningProvider>(
+      builder: (context, learn, _) {
+        return SafeArea(
+          child: ListView(
+            children: [
+              Padding(
+                padding: const EdgeInsets.all(12),
+                child: Text('Flashcards', style: Theme.of(context).textTheme.titleLarge),
+              ),
+              ...learn.flashcards.take(8).map((entry) => FlashcardWidget(entry: entry)),
+              Padding(
+                padding: const EdgeInsets.all(12),
+                child: Text('Quiz', style: Theme.of(context).textTheme.titleLarge),
+              ),
+              ...learn.quizQuestions.map((q) => QuizWidget(question: q)),
+              const Padding(
+                padding: EdgeInsets.all(12),
+                child: Card(
+                  child: Padding(
+                    padding: EdgeInsets.all(16),
+                    child: Text(
+                      'Conversation Practice\n\nPrompt yourself with daily routines, shopping, and travel situations and then verify with AI Chat.',
+                    ),
+                  ),
+                ),
+              ),
+            ],
+          ),
+        );
+      },
+    );
+  }
+}
diff --git a/lib/screens/word_detail_screen.dart b/lib/screens/word_detail_screen.dart
new file mode 100644
index 0000000000000000000000000000000000000000..5b891f7f36f092daf51c4c59bcff68dad96c7012
--- /dev/null
+++ b/lib/screens/word_detail_screen.dart
@@ -0,0 +1,65 @@
+import 'package:flutter/material.dart';
+import 'package:provider/provider.dart';
+
+import '../providers/dictionary_provider.dart';
+import '../providers/favorites_provider.dart';
+import '../providers/tts_provider.dart';
+import '../widgets/speed_control_slider.dart';
+
+class WordDetailScreen extends StatelessWidget {
+  const WordDetailScreen({super.key, required this.word});
+
+  final String word;
+
+  @override
+  Widget build(BuildContext context) {
+    final dictionary = context.watch<DictionaryProvider>();
+    final favorites = context.watch<FavoritesProvider>();
+    final tts = context.watch<TtsProvider>();
+    final entry = dictionary.byWord(word);
+
+    if (entry == null) {
+      return const Scaffold(body: Center(child: Text('Word not found')));
+    }
+
+    return Scaffold(
+      appBar: AppBar(title: Text(entry.creoleWord)),
+      body: ListView(
+        padding: const EdgeInsets.all(16),
+        children: [
+          Text(entry.creoleWord, style: Theme.of(context).textTheme.headlineMedium),
+          const SizedBox(height: 8),
+          Text('Português: ${entry.portugueseTranslation}'),
+          Text('English: ${entry.englishTranslation}'),
+          const SizedBox(height: 16),
+          Text('Example (Kriolu): ${entry.exampleCreole}'),
+          Text('Example (Português): ${entry.examplePortuguese}'),
+          const SizedBox(height: 16),
+          Row(
+            children: [
+              FilledButton.icon(
+                onPressed: () => tts.speak(entry.creoleWord),
+                icon: const Icon(Icons.volume_up),
+                label: const Text('Pronounce word'),
+              ),
+              const SizedBox(width: 12),
+              IconButton.filledTonal(
+                onPressed: () => favorites.toggle(entry.creoleWord),
+                icon: Icon(
+                  favorites.isFavorite(entry.creoleWord) ? Icons.favorite : Icons.favorite_border,
+                ),
+              ),
+            ],
+          ),
+          const SizedBox(height: 12),
+          FilledButton.tonal(
+            onPressed: () => tts.speak(entry.exampleCreole),
+            child: const Text('Pronounce example sentence'),
+          ),
+          const SizedBox(height: 16),
+          SpeedControlSlider(value: tts.speed, onChanged: tts.setSpeed),
+        ],
+      ),
+    );
+  }
+}
diff --git a/lib/services/ai_chat_service.dart b/lib/services/ai_chat_service.dart
new file mode 100644
index 0000000000000000000000000000000000000000..02cd0560005b0abf97a57c3f8e9f43c84422addd
--- /dev/null
+++ b/lib/services/ai_chat_service.dart
@@ -0,0 +1,17 @@
+class AiChatService {
+  String reply(String userMessage) {
+    final message = userMessage.toLowerCase();
+
+    if (message.contains('going home') || message.contains('going to home')) {
+      return 'In Kriolu you can say: "N ta bai kasa."';
+    }
+    if (message.contains('i will eat') || message.contains('eat')) {
+      return 'A common form is: "N ta bai kume."';
+    }
+    if (message.contains('hello')) {
+      return 'You can greet with "Oi" or "Olá" depending on context.';
+    }
+
+    return 'Try this pattern: subject + tense marker + verb. Example: "N ta kume" = "I eat".';
+  }
+}
diff --git a/lib/services/dictionary_service.dart b/lib/services/dictionary_service.dart
new file mode 100644
index 0000000000000000000000000000000000000000..682121d6c68ef0f41a9b9351e2ab7cc5d7fa269b
--- /dev/null
+++ b/lib/services/dictionary_service.dart
@@ -0,0 +1,25 @@
+import 'dart:convert';
+
+import 'package:flutter/services.dart';
+
+import '../models/dictionary_entry.dart';
+
+class DictionaryService {
+  Future<List<DictionaryEntry>> loadEntries() async {
+    final raw = await rootBundle.loadString('assets/data/dictionary.json');
+    final jsonList = jsonDecode(raw) as List<dynamic>;
+    return jsonList
+        .map((item) => DictionaryEntry.fromJson(item as Map<String, dynamic>))
+        .toList(growable: false);
+  }
+
+  List<DictionaryEntry> filter(List<DictionaryEntry> entries, String query) {
+    if (query.trim().isEmpty) return entries;
+    final normalized = query.toLowerCase().trim();
+    return entries.where((entry) {
+      return entry.creoleWord.toLowerCase().contains(normalized) ||
+          entry.portugueseTranslation.toLowerCase().contains(normalized) ||
+          entry.englishTranslation.toLowerCase().contains(normalized);
+    }).toList(growable: false);
+  }
+}
diff --git a/lib/services/favorites_service.dart b/lib/services/favorites_service.dart
new file mode 100644
index 0000000000000000000000000000000000000000..63c13e319feceb0167a0866f1caf4a3d58021a4b
--- /dev/null
+++ b/lib/services/favorites_service.dart
@@ -0,0 +1,9 @@
+import '../database/db_helper.dart';
+
+class FavoritesService {
+  Future<List<String>> getFavoriteWords() => DbHelper.instance.getFavorites();
+
+  Future<void> add(String creoleWord) => DbHelper.instance.addFavorite(creoleWord);
+
+  Future<void> remove(String creoleWord) => DbHelper.instance.removeFavorite(creoleWord);
+}
diff --git a/lib/services/tts_service.dart b/lib/services/tts_service.dart
new file mode 100644
index 0000000000000000000000000000000000000000..006d5d0828eb932f6f4cb6ed2fe57590cb5c887b
--- /dev/null
+++ b/lib/services/tts_service.dart
@@ -0,0 +1,16 @@
+import 'package:flutter_tts/flutter_tts.dart';
+
+class TtsService {
+  final FlutterTts _tts = FlutterTts();
+
+  Future<void> init() async {
+    await _tts.setLanguage('pt-BR');
+    await _tts.setPitch(1.0);
+  }
+
+  Future<void> setSpeed(double speed) => _tts.setSpeechRate(speed);
+
+  Future<void> speak(String text) => _tts.speak(text);
+
+  Future<void> stop() => _tts.stop();
+}
diff --git a/lib/widgets/chat_bubble.dart b/lib/widgets/chat_bubble.dart
new file mode 100644
index 0000000000000000000000000000000000000000..c5e9808c6f3adc78c9532a204a4b50dc7ab008e7
--- /dev/null
+++ b/lib/widgets/chat_bubble.dart
@@ -0,0 +1,27 @@
+import 'package:flutter/material.dart';
+
+import '../models/chat_message.dart';
+
+class ChatBubble extends StatelessWidget {
+  const ChatBubble({super.key, required this.message});
+
+  final ChatMessage message;
+
+  @override
+  Widget build(BuildContext context) {
+    final bg = message.isUser ? Theme.of(context).colorScheme.primaryContainer : Theme.of(context).colorScheme.surfaceVariant;
+    final align = message.isUser ? CrossAxisAlignment.end : CrossAxisAlignment.start;
+
+    return Column(
+      crossAxisAlignment: align,
+      children: [
+        Container(
+          margin: const EdgeInsets.symmetric(vertical: 4, horizontal: 12),
+          padding: const EdgeInsets.all(12),
+          decoration: BoxDecoration(color: bg, borderRadius: BorderRadius.circular(14)),
+          child: Text(message.content),
+        ),
+      ],
+    );
+  }
+}
diff --git a/lib/widgets/flashcard_widget.dart b/lib/widgets/flashcard_widget.dart
new file mode 100644
index 0000000000000000000000000000000000000000..66194f1587e8e9d13373b241676d4cb0b19102b8
--- /dev/null
+++ b/lib/widgets/flashcard_widget.dart
@@ -0,0 +1,30 @@
+import 'package:flutter/material.dart';
+
+import '../models/dictionary_entry.dart';
+
+class FlashcardWidget extends StatelessWidget {
+  const FlashcardWidget({super.key, required this.entry});
+
+  final DictionaryEntry entry;
+
+  @override
+  Widget build(BuildContext context) {
+    return Card(
+      margin: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
+      child: Padding(
+        padding: const EdgeInsets.all(16),
+        child: Column(
+          crossAxisAlignment: CrossAxisAlignment.start,
+          children: [
+            Text(entry.creoleWord, style: Theme.of(context).textTheme.headlineSmall),
+            const SizedBox(height: 8),
+            Text('PT: ${entry.portugueseTranslation}'),
+            Text('EN: ${entry.englishTranslation}'),
+            const SizedBox(height: 6),
+            Text('Example: ${entry.exampleCreole}'),
+          ],
+        ),
+      ),
+    );
+  }
+}
diff --git a/lib/widgets/quiz_widget.dart b/lib/widgets/quiz_widget.dart
new file mode 100644
index 0000000000000000000000000000000000000000..01d7f42f92e80d00ea1b981daeda16dc0ad4d966
--- /dev/null
+++ b/lib/widgets/quiz_widget.dart
@@ -0,0 +1,31 @@
+import 'package:flutter/material.dart';
+
+import '../models/quiz_question.dart';
+
+class QuizWidget extends StatelessWidget {
+  const QuizWidget({super.key, required this.question});
+
+  final QuizQuestion question;
+
+  @override
+  Widget build(BuildContext context) {
+    return Card(
+      margin: const EdgeInsets.all(12),
+      child: Padding(
+        padding: const EdgeInsets.all(16),
+        child: Column(
+          crossAxisAlignment: CrossAxisAlignment.start,
+          children: [
+            Text(question.prompt, style: Theme.of(context).textTheme.titleMedium),
+            const SizedBox(height: 12),
+            ...question.options.map((option) => ListTile(
+                  dense: true,
+                  title: Text(option),
+                  trailing: option == question.answer ? const Icon(Icons.check, color: Colors.green) : null,
+                )),
+          ],
+        ),
+      ),
+    );
+  }
+}
diff --git a/lib/widgets/search_bar_widget.dart b/lib/widgets/search_bar_widget.dart
new file mode 100644
index 0000000000000000000000000000000000000000..f1f1853d2b1bcf8c453515f6eb26c1ad9486a576
--- /dev/null
+++ b/lib/widgets/search_bar_widget.dart
@@ -0,0 +1,22 @@
+import 'package:flutter/material.dart';
+
+class SearchBarWidget extends StatelessWidget {
+  const SearchBarWidget({super.key, required this.onChanged});
+
+  final ValueChanged<String> onChanged;
+
+  @override
+  Widget build(BuildContext context) {
+    return Padding(
+      padding: const EdgeInsets.all(12),
+      child: TextField(
+        onChanged: onChanged,
+        decoration: InputDecoration(
+          hintText: 'Search in Creole, Portuguese, or English',
+          prefixIcon: const Icon(Icons.search),
+          border: OutlineInputBorder(borderRadius: BorderRadius.circular(16)),
+        ),
+      ),
+    );
+  }
+}
diff --git a/lib/widgets/speed_control_slider.dart b/lib/widgets/speed_control_slider.dart
new file mode 100644
index 0000000000000000000000000000000000000000..658922cb339b7f4dddbd7f39127dc7b0002422ad
--- /dev/null
+++ b/lib/widgets/speed_control_slider.dart
@@ -0,0 +1,26 @@
+import 'package:flutter/material.dart';
+
+class SpeedControlSlider extends StatelessWidget {
+  const SpeedControlSlider({super.key, required this.value, required this.onChanged});
+
+  final double value;
+  final ValueChanged<double> onChanged;
+
+  @override
+  Widget build(BuildContext context) {
+    return Column(
+      crossAxisAlignment: CrossAxisAlignment.start,
+      children: [
+        const Text('Speech speed'),
+        Slider(
+          value: value,
+          min: 0.2,
+          max: 0.8,
+          divisions: 6,
+          label: value.toStringAsFixed(2),
+          onChanged: onChanged,
+        ),
+      ],
+    );
+  }
+}
diff --git a/lib/widgets/word_list_item.dart b/lib/widgets/word_list_item.dart
new file mode 100644
index 0000000000000000000000000000000000000000..d4b8840fc857bead380249f1d60284ca44fd2acf
--- /dev/null
+++ b/lib/widgets/word_list_item.dart
@@ -0,0 +1,24 @@
+import 'package:flutter/material.dart';
+
+import '../models/dictionary_entry.dart';
+
+class WordListItem extends StatelessWidget {
+  const WordListItem({
+    super.key,
+    required this.entry,
+    required this.onTap,
+  });
+
+  final DictionaryEntry entry;
+  final VoidCallback onTap;
+
+  @override
+  Widget build(BuildContext context) {
+    return ListTile(
+      title: Text(entry.creoleWord),
+      subtitle: Text('${entry.portugueseTranslation} • ${entry.englishTranslation}'),
+      trailing: const Icon(Icons.chevron_right),
+      onTap: onTap,
+    );
+  }
+}
diff --git a/pubspec.yaml b/pubspec.yaml
new file mode 100644
index 0000000000000000000000000000000000000000..6543c2fc3d676b53d3fe5213881e4b9a1e50432e
--- /dev/null
+++ b/pubspec.yaml
@@ -0,0 +1,43 @@
+name: kriolu_ai_dictionary
+description: Offline Cape Verdean Creole learning app with dictionary, AI learning, and chat practice.
+publish_to: "none"
+version: 1.0.0+1
+
+environment:
+  sdk: ">=3.3.0 <4.0.0"
+
+dependencies:
+  flutter:
+    sdk: flutter
+  cupertino_icons: ^1.0.8
+  provider: ^6.1.2
+  flutter_tts: ^4.0.2
+  sqflite: ^2.3.3+1
+  path: ^1.9.0
+  path_provider: ^2.1.3
+  shared_preferences: ^2.2.3
+
+dev_dependencies:
+  flutter_test:
+    sdk: flutter
+  flutter_lints: ^4.0.0
+  flutter_launcher_icons: ^0.13.1
+  flutter_native_splash: ^2.4.1
+
+flutter:
+  uses-material-design: true
+  assets:
+    - assets/data/dictionary.json
+    - assets/audio/
+    - assets/images/
+
+flutter_launcher_icons:
+  android: true
+  ios: false
+  image_path: assets/icons/app_icon.png
+
+flutter_native_splash:
+  color: "#0A7E8C"
+  image: assets/images/splash_logo.png
+  android: true
+  ios: false
 
EOF
)
