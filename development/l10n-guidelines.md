# 🌍 Flutter/Dart Internationalization (l10n) Guidelines

## 1. Purpose
- Ensure the app supports multiple languages, is maintainable, and easily extendable.
- Separate text from code for easier translation and management.

## 2. Recommended Tools
- **Flutter built-in l10n** (`flutter_localizations`, `intl`)
- **ARB files**: Use `.arb` files to manage multilingual text.
- **Flutter Intl plugin** (VSCode/Android Studio) for automatic code generation.

## 3. Directory Structure
```
lib/
  l10n/
    intl_en.arb
    intl_vi.arb
    intl_...
```

## 4. Setting Up l10n in Flutter
1. Add to `pubspec.yaml`:
   ```yaml
   dependencies:
     flutter_localizations:
       sdk: flutter
     intl: ^0.18.0
   flutter:
     generate: true
   ```
2. Create the `lib/l10n/` directory and `.arb` files for each language:
   - `intl_en.arb`, `intl_vi.arb`, ...
3. Add key-value text pairs to the `.arb` files:
   ```json
   {
     "hello": "Hello",
     "greeting": "Hi {name}!"
   }
   ```
4. Configure `MaterialApp`:
   ```dart
   import 'package:flutter_localizations/flutter_localizations.dart';
   // ...
   MaterialApp(
     localizationsDelegates: [
       AppLocalizations.delegate,
       GlobalMaterialLocalizations.delegate,
       GlobalWidgetsLocalizations.delegate,
       GlobalCupertinoLocalizations.delegate,
     ],
     supportedLocales: [
       Locale('en'),
       Locale('vi'),
     ],
     // ...
   )
   ```
5. Use l10n in your code:
   ```dart
   AppLocalizations.of(context)!.hello
   AppLocalizations.of(context)!.greeting(name: 'Teo')
   ```

## 5. Best Practices
- **Never hardcode text** in UI code.
- **Use clear, contextual key names** (e.g., `login_button`, `home_title`).
- **Use placeholders** for dynamic values (e.g., `greeting": "Hi {name}!"`).
- **Review text keys regularly** to avoid duplication.
- **Separate technical text** (errors, labels, messages) from marketing text if needed.
- **Test multilingual UI** on real devices (use device language switch tools).

## 6. Suggested Workflow
1. When adding/editing text, update the `.arb` files.
2. Run `flutter gen-l10n` or use a plugin to generate code.
3. Review the UI in each supported language.
4. Before release, send `.arb` files to the translation team if needed.

## 7. Useful Tools
- [Flutter Intl plugin](https://marketplace.visualstudio.com/items?itemName=localizely.flutter-intl)
- [Localizely](https://localizely.com/) (online translation, repo sync)
- [Arb Editor](https://github.com/localizely/arb-editor)

## 8. Example .arb Files
`intl_en.arb`:
```json
{
  "app_title": "My App",
  "login_button": "Login",
  "greeting": "Hi {name}!"
}
```

`intl_vi.arb`:
```json
{
  "app_title": "Ứng dụng của tôi",
  "login_button": "Đăng nhập",
  "greeting": "Chào {name}!"
}
```

## 9. Auto-translate ARB Files Using Google Translate or Gemini API

You can automate the translation of your `.arb` files using Google Translate API or Gemini API. This is useful for quickly generating translations for multiple languages, but always review machine translations for accuracy.

### 1. Get an API Key
- **Google Translate**: Sign up at [Google Cloud Console](https://console.cloud.google.com/), enable the Cloud Translation API, and create an API key.
- **Gemini API**: Sign up and get your API key from Gemini (if it supports translation).

### 2. Install Dependencies (Node.js Example)
We recommend using Node.js for scripting, but you can use Dart or Python as well.

```bash
npm install axios
```

### 3. Example Script to Auto-translate ARB (Node.js)
This script reads your source `.arb` (e.g., `intl_en.arb`), translates each value, and writes a new `.arb` (e.g., `intl_vi.arb`).

```js
const fs = require('fs');
const axios = require('axios');

const API_KEY = 'YOUR_GOOGLE_API_KEY'; // Or Gemini API key
const TARGET_LANG = 'vi'; // Target language code
const INPUT_FILE = 'lib/l10n/intl_en.arb';
const OUTPUT_FILE = `lib/l10n/intl_${TARGET_LANG}.arb`;

async function translateText(text, target) {
  const url = `https://translation.googleapis.com/language/translate/v2?key=${API_KEY}`;
  const res = await axios.post(url, {
    q: text,
    target: target,
    format: 'text',
  });
  return res.data.data.translations[0].translatedText;
}

async function translateArb() {
  const data = JSON.parse(fs.readFileSync(INPUT_FILE, 'utf8'));
  const result = {};
  for (const [key, value] of Object.entries(data)) {
    if (typeof value === 'string') {
      result[key] = await translateText(value, TARGET_LANG);
      console.log(`${key}: ${result[key]}`);
    } else {
      result[key] = value;
    }
  }
  fs.writeFileSync(OUTPUT_FILE, JSON.stringify(result, null, 2), 'utf8');
  console.log(`Done! Translated file saved to ${OUTPUT_FILE}`);
}

translateArb();
```

- For **Gemini API**, change the `translateText` function to use Gemini's endpoint and request format.

### 4. Notes
- Always review the translated `.arb` file for context and accuracy.
- Some keys may contain placeholders (e.g., `{name}`); make sure the translation preserves them.
- You can extend the script to support batch translation for multiple languages.
- For large files, consider adding delay or batching to avoid rate limits.

---

**Notes:**
- For large apps, consider splitting `.arb` files by module.
- Always test UI with both short and long text.
- If supporting RTL (right-to-left) languages, test with Arabic/Hebrew as well. 