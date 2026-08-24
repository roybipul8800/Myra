# MYRA AI Assistant — Kotlin / Jetpack Compose starter

এই প্রজেক্টটা Android Studio-তে সরাসরি **Open** করা যাবে (File → Open → এই ফোল্ডার সিলেক্ট করো)। এটা একটা কাজ-করা **স্টার্টার প্রজেক্ট** — persona, permissions, floating orb, continuous voice loop, আর call/SMS function-calling সব কিছুর কাঠামো রেডি আছে। LLM backend আর ফাইনাল polish তোমাকে বসাতে হবে।

## ফাইল স্ট্রাকচার

```
MyraAI/
├── app/src/main/java/com/myra/assistant/
│   ├── MyraApp.kt              → notification channel setup
│   ├── MainActivity.kt         → Compose onboarding screen (permission grants, Start/Stop)
│   ├── PermissionManager.kt    → runtime + overlay permission helpers
│   ├── MyraPersona.kt          → system prompt + tiny offline fallback parser
│   ├── MyraBackendClient.kt    → calls YOUR backend LLM, parses reply+action JSON
│   ├── VoiceManager.kt         → STT (SpeechRecognizer) + TTS wrapper
│   ├── FunctionExecutor.kt     → makePhoneCall / sendSMS / readSMS / openApp
│   ├── OverlayService.kt       → foreground service: floating orb + listen loop
│   ├── SmsReceiver.kt          → announces incoming SMS out loud
│   └── ui/theme/Theme.kt
└── app/src/main/AndroidManifest.xml   → all requested permissions + components
```

## জরুরি জিনিস যেগুলো তোমাকে নিজে বসাতে হবে

1. **LLM backend**: `MyraBackendClient.BACKEND_URL` এ তোমার নিজের সার্ভারের endpoint বসাও। **কখনোই Anthropic/OpenAI API key সরাসরি অ্যাপের কোডে hardcode কোরো না** — key leak হয়ে যাবে decompile করলেই। তোমার backend সেই key নিরাপদে রাখবে, `MyraPersona.SYSTEM_PROMPT` + ব্যবহারকারীর কথা backend-এ পাঠাবে, LLM থেকে JSON রেসপন্স নিয়ে আবার অ্যাপে ফেরত পাঠাবে। Network না থাকলে `MyraPersona.localFallback()` সাধারণ কিছু command (call/sms/read sms) অফলাইনেও বোঝে।
2. **ElevenLabs-এর মতো premium TTS voice** চাইলে `VoiceManager.speak()`-এর জায়গায় সেই API কল বসাও (audio বাইট স্ট্রিম করে play করতে হবে) — এখন এটা built-in Android TTS ব্যবহার করছে যেটা কোনো key ছাড়াই কাজ করে।
3. **App icon**: এখন একটা সাধারণ placeholder adaptive icon আছে (`res/drawable/ic_launcher_foreground.xml`)। Android Studio-র Image Asset Studio দিয়ে replace করে নিতে পারো।

## নিরাপত্তার জন্য যা ইচ্ছাকৃতভাবে রাখা হয়েছে

- **Emergency number auto-dial বন্ধ**: `FunctionExecutor.callNumber()` ইমার্জেন্সি নম্বর হলে dialer খুলে দেয় (auto-dial করে না), যাতে ভুল করে 911/999/112-এ কল না চলে যায়।
- **Contact resolution**: call/SMS করার আগে নাম দিয়ে Contacts থেকে নম্বর খোঁজা হয়, তাই voice command ভুল শোনা গেলেও random নম্বরে message যাওয়ার সম্ভাবনা কমে।
- **Default = confirm-before-call**: `callNumber()`-এ `autoDialWithoutConfirmation = false` default, মানে এখন MYRA dialer খুলে দেয়, ইউজারকে শেষ ট্যাপ নিজে করতে হয়। Fully hands-free করতে চাইলে সেই ফ্ল্যাগ true করো — কিন্তু এতে ভুল কল হওয়ার ঝুঁকি বাড়ে, তাই production-এ এটা একটা explicit in-app toggle হিসেবে রাখা ভালো, hidden default হিসেবে না।

## Runtime behavior (Android 13+/14 বিবেচনায়)

- `SYSTEM_ALERT_WINDOW`, `READ_SMS`, `CALL_PHONE` — এগুলো "sensitive" permission হিসেবে ধরা হয় বলে Play Store-এ প্রকাশ করলে Google-এর **Restricted Permissions declaration form** পূরণ করতে হবে এবং app-কে justify করতে হবে কেন এগুলো দরকার। ব্যক্তিগত/sideload ব্যবহারে এই সমস্যা নেই।
- `READ_SMS` কিছু Android ভার্সনে ভালোভাবে কাজ করতে হলে অ্যাপকে user-এর **default SMS app** বানাতে হতে পারে — নাহলে শুধু নিজের পাঠানো/receive করা কিছু মেসেজ দেখা যাবে, পুরো ইনবক্স না।
- Background-এ SpeechRecognizer চালিয়ে রাখা battery-heavy; বাস্তব ব্যবহারে wake-word detection (যেমন Porcupine) দিয়ে continuous full STT-র বদলে "hotword শুনে তারপর STT চালু করা" করাই ভালো।

## Build করার ধাপ

1. Android Studio (Hedgehog+) এ প্রজেক্ট Open করো, Gradle sync হতে দাও।
2. একটা physical device বা emulator এ Run করো (`app` module)।
3. প্রথমবার চালালে সব permission grant করতে হবে (MainActivity-র "Grant" বাটনগুলো থেকে), তারপর "Start MYRA" চাপলে floating orb দেখা যাবে।
