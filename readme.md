Step 0 — Install tooling
Install Flutter SDK (flutter.dev → get started for your OS)
Run flutter doctor in terminal — fix any red X's it shows (usually Android SDK/licenses)
Install Android Studio (gives you the Android SDK + emulator + AVD manager)
Install VS Code + the "Flutter" and "Dart" extensions (nicer daily editor than Android Studio)
Create an Android emulator via Android Studio's Device Manager (pick a Pixel profile, latest Android version)
Verify setup: flutter doctor should show all green (iOS checkmarks only matter if you have a Mac)
Step 1 — Create the project
flutter create imposter_word_game
cd imposter_word_game
flutter run

You should see the default counter app on your emulator. If this works, your environment is good.

Step 2 — Clean up and set folder structure

Delete the default counter code in main.dart, create these folders under lib/:

lib/models/
lib/data/
lib/screens/
lib/state/
lib/widgets/

Add assets/ at project root for word_packs.json, and register it in pubspec.yaml under flutter: assets:.

Step 3 — Add core packages

In pubspec.yaml, add:

yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.1
  shared_preferences: ^2.2.2

Run flutter pub get.

Step 4 — Build the data models (no UI yet)

Create models/player.dart — id, name, isImposter, hasRevealed.
Create models/game_state.dart — enum GamePhase { setup, reveal, clues, voting, results }, plus current word, category, players list, imposter index.

Step 5 — Write word_packs.json

Start with one category, ~15-20 words, to test the flow. Expand later once the app works end to end.

json
{
  "categories": {
    "Food": ["Pizza", "Sushi", "Burger", "Tacos", "..."]
  }
}
Step 6 — Build GameController (pure logic, test without any screens)

This is the heart of the app — write it and test it before touching UI:

startGame(playerNames, category) → randomly assigns 1 imposter, picks a random word
nextPlayer() → advances whose turn it is to reveal
allRevealed() → bool check
startVoting() → phase transition
submitVote(voterId, suspectId) → tallies
getResults() → returns imposter id + whether group guessed correctly

Test this in isolation with a quick main() print test or a Dart unit test before wiring any UI — bugs here are much easier to catch without screens in the way.

Step 7 — Wire up state management

Wrap GameController in a ChangeNotifier, provide it at the app root with ChangeNotifierProvider, so any screen can read/update game state.

Step 8 — Build screens in this order (each one runnable/testable immediately)
Home screen — "New Game" button → goes to Setup
Setup screen — text fields for player names (add/remove), category dropdown, "Start" button
Reveal screen — the core UX. Show current player's name → "Tap and hold to reveal" → show word or "IMPOSTER" while held → release hides it again → "Pass to next player" button
Clue order screen — just displays player order as a list, "Start Voting" button when done
Voting screen — simplest v1: one list of players, host taps who the group suspects
Results screen — reveals imposter, shows win/lose, "Play Again" / "New Game" buttons

Test each screen on the emulator as you build it — don't wait until all 6 are done.

Step 9 — Build the flip/reveal widget properly

Once the basic reveal screen works with a simple show/hide, upgrade it to a proper animated card flip using AnimatedBuilder + Transform (rotateY), and use GestureDetector's onLongPress/onLongPressUp for the tap-and-hold behavior. Test this specifically on a real phone — it's the most "felt" part of the app.

Step 10 — Add settings persistence

Use shared_preferences to remember: last player names, last category picked, sound on/off. Small win for repeat play sessions.

Step 11 — Expand content

Add more categories to word_packs.json (Movies, Animals, Random Mix, etc.), maybe a difficulty tag per word.

Step 12 — Polish pass
Haptic feedback on reveal (HapticFeedback.mediumImpact())
Sound effects (audioplayers package) — optional toggle
App icon + splash screen (flutter_launcher_icons, flutter_native_splash packages)
Dark mode support if desired
Step 13 — Test on real device

Plug in your Android phone, run flutter run, play a full round with actual pass-the-phone behavior. Fix anything that feels off timing/animation-wise — this always differs from emulator feel.

Step 14 — Build release version
flutter build apk --release

This gives you an installable APK to share directly, before you even bother with the Play Store submission process.

Step 15 — (Later) Store submission

Play Store requires a developer account ($25 one-time), privacy policy page, screenshots, and app listing details. iOS requires a Mac + Apple Developer account ($99/year). This step can wait until the app itself is solid.