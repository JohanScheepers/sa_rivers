# Implementation Plan - SA Rivers IoT

## Goal Description
Initialize the Flutter application and implement Firebase Authentication with email verification. The app will allow users to sign up and log in using their email and password. Access to the main dashboard will be restricted to verified users.

## User Review Required
> [!IMPORTANT]
> **Firebase Setup**: The user needs to provide the `google-services.json` (Android) and `GoogleService-Info.plist` (iOS) files after I add the dependencies, or I can use `flutterfire configure` if they have the CLI set up. For now, I will assume manual setup or mock it if needed, but the code will be ready for real Firebase.

## Proposed Changes

### Flutter App (`app/`)

#### [NEW] Dependencies
- `firebase_core`
- `firebase_auth`
- `flutter_signals` (State Management)
- `go_router` (Navigation)

#### [NEW] `lib/services/auth_service.dart`
- `AuthService` class:
    - `Stream<User?> get authStateChanges`
    - `Future<void> signIn(String email, String password)`
    - `Future<void> signUp(String email, String password)`
    - `Future<void> signOut()`
    - `Future<void> sendEmailVerification()`
    - `bool get isEmailVerified`

#### [NEW] `lib/features/auth/`
- `login_screen.dart`: UI for logging in.
- `register_screen.dart`: UI for registration.
- `verify_email_screen.dart`: UI prompting user to verify email.

#### [NEW] `lib/main.dart`
- Initialize Firebase.
- Setup `GoRouter` with redirects based on auth state.

### Documentation
- [x] Update `README.md` with sensor image and Auth details.
- [x] Update `task.md` with Auth tasks.

## Verification Plan

### Automated Tests
- Widget tests for Login/Register screens.

### Manual Verification
1.  Start the app.
2.  Verify redirection to Login screen.
3.  Register a new user.
4.  Verify "Verify Email" screen appears.
5.  (Mock/Real) Verify email link click.
6.  Verify redirection to Dashboard.
