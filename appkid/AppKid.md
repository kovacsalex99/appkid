# ActiveUnlock: Complete Implementation Plan

## Project Structure

```
ActiveUnlock/
├── ActiveUnlock.xcodeproj
├── ActiveUnlock/                          # Main App Target
│   ├── App/
│   │   ├── ActiveUnlockApp.swift          # App entry point
│   │   └── ContentView.swift              # Root view controller
│   │
│   ├── Core/
│   │   ├── AppState.swift                 # Global app state manager
│   │   └── Constants.swift                # App-wide constants
│   │
│   ├── Models/
│   │   ├── ScreenTimeModel.swift          # FamilyActivitySelection storage
│   │   ├── SettingsModel.swift            # User preferences (time, reps)
│   │   └── ExerciseState.swift            # Exercise session state
│   │
│   ├── Services/
│   │   ├── ScreenTimeManager.swift        # FamilyControls authorization
│   │   ├── ShieldManager.swift            # Apply/remove shields
│   │   ├── MonitoringScheduler.swift      # DeviceActivity scheduling
│   │   ├── CameraService.swift            # AVFoundation camera setup
│   │   ├── PoseDetectionService.swift     # Vision framework processing
│   │   ├── AudioService.swift             # Sound effects
│   │   └── StorageService.swift           # UserDefaults (App Group)
│   │
│   ├── Views/
│   │   ├── Onboarding/
│   │   │   ├── OnboardingContainerView.swift
│   │   │   ├── WelcomeView.swift
│   │   │   ├── PermissionRequestView.swift
│   │   │   └── PinSetupView.swift
│   │   │
│   │   ├── Home/
│   │   │   ├── HomeView.swift             # Main child-facing screen
│   │   │   └── BatteryIndicator.swift     # Animated battery component
│   │   │
│   │   ├── Exercise/
│   │   │   ├── ExerciseView.swift         # Camera + counter screen
│   │   │   ├── CameraPreviewView.swift    # UIViewRepresentable for camera
│   │   │   ├── RepCounterOverlay.swift    # Big number display
│   │   │   └── CompletionView.swift       # Confetti celebration
│   │   │
│   │   ├── Parent/
│   │   │   ├── PinEntryView.swift         # 4-digit keypad
│   │   │   ├── ParentDashboardView.swift  # Settings screen
│   │   │   └── AppPickerView.swift        # FamilyActivityPicker wrapper
│   │   │
│   │   └── Components/
│   │       ├── BigButton.swift            # Reusable large button
│   │       └── ConfettiView.swift         # Celebration animation
│   │
│   ├── Resources/
│   │   ├── Assets.xcassets
│   │   ├── Sounds/
│   │   │   ├── click.mp3
│   │   │   ├── ding.mp3
│   │   │   └── success.mp3
│   │   └── Info.plist
│   │
│   └── ActiveUnlock.entitlements
│
├── ActiveMonitor/                         # DeviceActivityMonitor Extension
│   ├── ActiveMonitor.swift                # Extension entry point
│   ├── Info.plist
│   └── ActiveMonitor.entitlements
│
├── ActiveShield/                          # ShieldConfiguration Extension
│   ├── ShieldConfigurationExtension.swift # Custom shield UI
│   ├── Info.plist
│   └── ActiveShield.entitlements
│
└── Shared/                                # Code shared between targets
    ├── SharedConstants.swift              # App Group ID, keys
    └── SharedStorage.swift                # UserDefaults wrapper
```

---

## Implementation Steps

---

### STEP 1: Create Xcode Project
**Time: 30 minutes**

1. Open Xcode → File → New → Project
2. Select "App" under iOS
3. Configure:
   - Product Name: `ActiveUnlock`
   - Team: Your Apple Developer account
   - Organization Identifier: `com.yourname`
   - Interface: SwiftUI
   - Language: Swift
   - Uncheck "Include Tests" (add later)
4. Save to your preferred location

---

### STEP 2: Add Extension Targets
**Time: 30 minutes**

**Add DeviceActivityMonitor Extension:**
1. File → New → Target
2. Search for "Device Activity Monitor Extension"
3. Name it: `ActiveMonitor`
4. Click Finish
5. When prompted "Activate scheme?", click Cancel

**Add ShieldConfiguration Extension:**
1. File → New → Target
2. Search for "Shield Configuration Extension"
3. Name it: `ActiveShield`
4. Click Finish

**Verify:** You should now see 3 targets in the project navigator sidebar.

---

### STEP 3: Configure App Groups
**Time: 20 minutes**

**For EACH of the 3 targets (ActiveUnlock, ActiveMonitor, ActiveShield):**

1. Select the target in Project Navigator
2. Go to "Signing & Capabilities" tab
3. Click "+ Capability"
4. Add "App Groups"
5. Click the "+" under App Groups
6. Enter: `group.com.yourname.activeunlock`
7. Ensure the checkbox is checked

**Verify:** All 3 targets show the same App Group ID with a checkmark.

---

### STEP 4: Configure Family Controls Entitlement
**Time: 15 minutes**

**For EACH of the 3 targets:**

1. Select target → Signing & Capabilities
2. Click "+ Capability"
3. Add "Family Controls"

**Note:** This capability must be enabled in your Apple Developer account first. If you see an error, go to developer.apple.com → Certificates, Identifiers & Profiles → Identifiers → Select your App ID → Enable Family Controls.

---

### STEP 5: Create Folder Structure
**Time: 15 minutes**

In Xcode, right-click the `ActiveUnlock` folder:
1. Create groups (folders) matching the structure above:
   - App
   - Core
   - Models
   - Services
   - Views (with subfolders: Onboarding, Home, Exercise, Parent, Components)
   - Resources

---

### STEP 6: Create Shared Code
**Time: 30 minutes**

**File: `Shared/SharedConstants.swift`**
```swift
import Foundation

enum AppGroup {
    static let identifier = "group.com.yourname.activeunlock"
}

enum StorageKeys {
    static let selectedApps = "selectedApps"
    static let sessionDuration = "sessionDuration"
    static let requiredReps = "requiredReps"
    static let parentPin = "parentPin"
    static let isOnboarded = "isOnboarded"
    static let isShieldActive = "isShieldActive"
}

enum Defaults {
    static let sessionDuration: Int = 30  // minutes
    static let requiredReps: Int = 10
}
```

**File: `Shared/SharedStorage.swift`**
```swift
import Foundation
import FamilyControls

class SharedStorage {
    static let shared = SharedStorage()
    
    private let defaults: UserDefaults
    
    private init() {
        defaults = UserDefaults(suiteName: AppGroup.identifier)!
    }
    
    // MARK: - App Selection
    var selectedApps: FamilyActivitySelection {
        get {
            guard let data = defaults.data(forKey: StorageKeys.selectedApps),
                  let selection = try? JSONDecoder().decode(FamilyActivitySelection.self, from: data)
            else { return FamilyActivitySelection() }
            return selection
        }
        set {
            let data = try? JSONEncoder().encode(newValue)
            defaults.set(data, forKey: StorageKeys.selectedApps)
        }
    }
    
    // MARK: - Settings
    var sessionDuration: Int {
        get { defaults.integer(forKey: StorageKeys.sessionDuration).nonZero ?? Defaults.sessionDuration }
        set { defaults.set(newValue, forKey: StorageKeys.sessionDuration) }
    }
    
    var requiredReps: Int {
        get { defaults.integer(forKey: StorageKeys.requiredReps).nonZero ?? Defaults.requiredReps }
        set { defaults.set(newValue, forKey: StorageKeys.requiredReps) }
    }
    
    var parentPin: String? {
        get { defaults.string(forKey: StorageKeys.parentPin) }
        set { defaults.set(newValue, forKey: StorageKeys.parentPin) }
    }
    
    var isOnboarded: Bool {
        get { defaults.bool(forKey: StorageKeys.isOnboarded) }
        set { defaults.set(newValue, forKey: StorageKeys.isOnboarded) }
    }
    
    var isShieldActive: Bool {
        get { defaults.bool(forKey: StorageKeys.isShieldActive) }
        set { defaults.set(newValue, forKey: StorageKeys.isShieldActive) }
    }
}

private extension Int {
    var nonZero: Int? { self == 0 ? nil : self }
}
```

**Important:** Add both files to ALL 3 targets (check the file inspector on the right).

---

### STEP 7: Implement ScreenTimeManager
**Time: 45 minutes**

**File: `Services/ScreenTimeManager.swift`**
```swift
import SwiftUI
import FamilyControls

@MainActor
class ScreenTimeManager: ObservableObject {
    static let shared = ScreenTimeManager()
    
    @Published var authorizationStatus: AuthorizationStatus = .notDetermined
    
    enum AuthorizationStatus {
        case notDetermined
        case approved
        case denied
    }
    
    private let authCenter = AuthorizationCenter.shared
    
    private init() {
        // Check current status on init
        checkAuthorizationStatus()
    }
    
    func checkAuthorizationStatus() {
        switch authCenter.authorizationStatus {
        case .notDetermined:
            authorizationStatus = .notDetermined
        case .approved:
            authorizationStatus = .approved
        case .denied:
            authorizationStatus = .denied
        @unknown default:
            authorizationStatus = .notDetermined
        }
    }
    
    func requestAuthorization() async {
        do {
            try await authCenter.requestAuthorization(for: .individual)
            authorizationStatus = .approved
        } catch {
            print("Authorization failed: \(error)")
            authorizationStatus = .denied
        }
    }
}
```

---

### STEP 8: Implement ShieldManager
**Time: 30 minutes**

**File: `Services/ShieldManager.swift`**
```swift
import Foundation
import ManagedSettings
import FamilyControls

class ShieldManager {
    static let shared = ShieldManager()
    
    private let store = ManagedSettingsStore()
    private let storage = SharedStorage.shared
    
    private init() {}
    
    /// Apply shield to selected apps
    func applyShield() {
        let selection = storage.selectedApps
        
        store.shield.applications = selection.applicationTokens
        store.shield.applicationCategories = .specific(selection.categoryTokens)
        store.shield.webDomainCategories = .specific(selection.categoryTokens)
        
        storage.isShieldActive = true
    }
    
    /// Remove shield from all apps
    func removeShield() {
        store.shield.applications = nil
        store.shield.applicationCategories = nil
        store.shield.webDomainCategories = nil
        
        storage.isShieldActive = false
    }
    
    /// Check if shield is currently active
    var isShieldActive: Bool {
        storage.isShieldActive
    }
}
```

---

### STEP 9: Implement MonitoringScheduler
**Time: 45 minutes**

**File: `Services/MonitoringScheduler.swift`**
```swift
import Foundation
import DeviceActivity
import FamilyControls

extension DeviceActivityName {
    static let daily = DeviceActivityName("com.activeunlock.daily")
}

extension DeviceActivityEvent.Name {
    static let screenTimeThreshold = DeviceActivityEvent.Name("com.activeunlock.threshold")
}

class MonitoringScheduler {
    static let shared = MonitoringScheduler()
    
    private let center = DeviceActivityCenter()
    private let storage = SharedStorage.shared
    
    private init() {}
    
    /// Start monitoring screen time
    func startMonitoring() {
        stopMonitoring() // Clear any existing schedule
        
        let selection = storage.selectedApps
        let durationMinutes = storage.sessionDuration
        
        // Schedule runs all day
        let schedule = DeviceActivitySchedule(
            intervalStart: DateComponents(hour: 0, minute: 0, second: 0),
            intervalEnd: DateComponents(hour: 23, minute: 59, second: 59),
            repeats: true
        )
        
        // Event triggers when threshold is reached
        let event = DeviceActivityEvent(
            applications: selection.applicationTokens,
            categories: selection.categoryTokens,
            webDomains: selection.webDomainTokens,
            threshold: DateComponents(minute: durationMinutes)
        )
        
        do {
            try center.startMonitoring(
                .daily,
                during: schedule,
                events: [.screenTimeThreshold: event]
            )
            print("Monitoring started: \(durationMinutes) minutes")
        } catch {
            print("Failed to start monitoring: \(error)")
        }
    }
    
    /// Stop all monitoring
    func stopMonitoring() {
        center.stopMonitoring([.daily])
    }
}
```

---

### STEP 10: Implement DeviceActivityMonitor Extension
**Time: 30 minutes**

**File: `ActiveMonitor/ActiveMonitor.swift`**
```swift
import DeviceActivity
import ManagedSettings
import Foundation

class ActiveMonitor: DeviceActivityMonitor {
    
    private let store = ManagedSettingsStore()
    private let storage = SharedStorage.shared
    
    // Called when the time threshold is reached
    override func eventDidReachThreshold(_ event: DeviceActivityEvent.Name, activity: DeviceActivityName) {
        super.eventDidReachThreshold(event, activity: activity)
        
        // Apply shield to block apps
        let selection = storage.selectedApps
        
        store.shield.applications = selection.applicationTokens
        store.shield.applicationCategories = .specific(selection.categoryTokens)
        
        storage.isShieldActive = true
        
        print("Threshold reached - Shield applied")
    }
    
    // Called at the start of each monitoring interval
    override func intervalDidStart(for activity: DeviceActivityName) {
        super.intervalDidStart(for: activity)
        print("Monitoring interval started")
    }
    
    // Called at the end of each monitoring interval
    override func intervalDidEnd(for activity: DeviceActivityName) {
        super.intervalDidEnd(for: activity)
        print("Monitoring interval ended")
    }
}
```

**Verify:** Ensure `SharedStorage.swift` and `SharedConstants.swift` are added to this target.

---

### STEP 11: Implement Shield Configuration Extension
**Time: 30 minutes**

**File: `ActiveShield/ShieldConfigurationExtension.swift`**
```swift
import ManagedSettingsUI
import ManagedSettings
import UIKit

class ShieldConfigurationExtension: ShieldConfigurationDataSource {
    
    override func configuration(shielding application: Application) -> ShieldConfiguration {
        return ShieldConfiguration(
            backgroundBlurStyle: .systemThickMaterial,
            backgroundColor: UIColor.white,
            icon: UIImage(systemName: "battery.0"),
            title: ShieldConfiguration.Label(
                text: "Out of Energy!",
                color: .systemRed
            ),
            subtitle: ShieldConfiguration.Label(
                text: "Time to recharge with exercise",
                color: .secondaryLabel
            ),
            primaryButtonLabel: ShieldConfiguration.Label(
                text: "Recharge Now",
                color: .white
            ),
            primaryButtonBackgroundColor: .systemGreen,
            secondaryButtonLabel: nil
        )
    }
    
    override func configuration(shielding application: Application, in category: ActivityCategory) -> ShieldConfiguration {
        // Use same configuration for category-based shields
        return configuration(shielding: application)
    }
    
    override func configuration(shielding webDomain: WebDomain) -> ShieldConfiguration {
        return ShieldConfiguration(
            backgroundBlurStyle: .systemThickMaterial,
            backgroundColor: UIColor.white,
            icon: UIImage(systemName: "battery.0"),
            title: ShieldConfiguration.Label(
                text: "Website Blocked",
                color: .systemRed
            ),
            subtitle: ShieldConfiguration.Label(
                text: "Complete exercise to continue",
                color: .secondaryLabel
            ),
            primaryButtonLabel: ShieldConfiguration.Label(
                text: "Recharge Now",
                color: .white
            ),
            primaryButtonBackgroundColor: .systemGreen
        )
    }
}
```

---

### STEP 12: Implement Shield Action Extension
**Time: 20 minutes**

Create a new file in ActiveShield target:

**File: `ActiveShield/ShieldActionExtension.swift`**
```swift
import ManagedSettingsUI
import ManagedSettings

class ShieldActionExtension: ShieldActionDelegate {
    
    override func handle(action: ShieldAction, for application: Application, completionHandler: @escaping (ShieldActionResponse) -> Void) {
        switch action {
        case .primaryButtonPressed:
            // Open main app
            completionHandler(.defer)
        case .secondaryButtonPressed:
            completionHandler(.none)
        @unknown default:
            completionHandler(.none)
        }
    }
    
    override func handle(action: ShieldAction, for webDomain: WebDomain, completionHandler: @escaping (ShieldActionResponse) -> Void) {
        switch action {
        case .primaryButtonPressed:
            completionHandler(.defer)
        default:
            completionHandler(.none)
        }
    }
}
```

---

### STEP 13: Implement Camera Service
**Time: 45 minutes**

**File: `Services/CameraService.swift`**
```swift
import AVFoundation
import SwiftUI

class CameraService: NSObject, ObservableObject {
    @Published var isRunning = false
    @Published var error: CameraError?
    
    let captureSession = AVCaptureSession()
    private let sessionQueue = DispatchQueue(label: "camera.session")
    private let videoOutput = AVCaptureVideoDataOutput()
    
    weak var delegate: AVCaptureVideoDataOutputSampleBufferDelegate?
    
    enum CameraError: Error, LocalizedError {
        case cameraUnavailable
        case inputError
        case configurationError
        
        var errorDescription: String? {
            switch self {
            case .cameraUnavailable: return "Camera not available"
            case .inputError: return "Cannot access camera input"
            case .configurationError: return "Camera configuration failed"
            }
        }
    }
    
    func checkPermission() async -> Bool {
        switch AVCaptureDevice.authorizationStatus(for: .video) {
        case .authorized:
            return true
        case .notDetermined:
            return await AVCaptureDevice.requestAccess(for: .video)
        default:
            return false
        }
    }
    
    func setupCamera() {
        sessionQueue.async { [weak self] in
            self?.configureSession()
        }
    }
    
    private func configureSession() {
        captureSession.beginConfiguration()
        captureSession.sessionPreset = .high
        
        // Get front camera
        guard let camera = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .front) else {
            DispatchQueue.main.async { self.error = .cameraUnavailable }
            return
        }
        
        // Add input
        do {
            let input = try AVCaptureDeviceInput(device: camera)
            if captureSession.canAddInput(input) {
                captureSession.addInput(input)
            } else {
                DispatchQueue.main.async { self.error = .inputError }
                return
            }
        } catch {
            DispatchQueue.main.async { self.error = .inputError }
            return
        }
        
        // Add output
        videoOutput.alwaysDiscardsLateVideoFrames = true
        videoOutput.videoSettings = [kCVPixelBufferPixelFormatTypeKey as String: kCVPixelFormatType_32BGRA]
        
        if captureSession.canAddOutput(videoOutput) {
            captureSession.addOutput(videoOutput)
            
            // Set orientation
            if let connection = videoOutput.connection(with: .video) {
                if connection.isVideoRotationAngleSupported(90) {
                    connection.videoRotationAngle = 90
                }
                if connection.isVideoMirroringSupported {
                    connection.isVideoMirrored = true
                }
            }
        }
        
        captureSession.commitConfiguration()
    }
    
    func setDelegate(_ delegate: AVCaptureVideoDataOutputSampleBufferDelegate) {
        self.delegate = delegate
        videoOutput.setSampleBufferDelegate(delegate, queue: DispatchQueue(label: "video.processing"))
    }
    
    func start() {
        sessionQueue.async { [weak self] in
            self?.captureSession.startRunning()
            DispatchQueue.main.async {
                self?.isRunning = true
            }
        }
    }
    
    func stop() {
        sessionQueue.async { [weak self] in
            self?.captureSession.stopRunning()
            DispatchQueue.main.async {
                self?.isRunning = false
            }
        }
    }
}
```

---

### STEP 14: Implement Pose Detection Service
**Time: 1 hour**

**File: `Services/PoseDetectionService.swift`**
```swift
import Vision
import AVFoundation
import Combine

@MainActor
class PoseDetectionService: NSObject, ObservableObject {
    
    // MARK: - Published Properties
    @Published var repCount: Int = 0
    @Published var currentState: ExerciseState = .standing
    @Published var confidence: Float = 0.0
    @Published var feedbackMessage: String = "Stand where I can see you"
    @Published var isBodyVisible: Bool = false
    
    // MARK: - State Machine
    enum ExerciseState {
        case standing
        case squatting
        case unknown
    }
    
    // MARK: - Configuration
    private let requiredReps: Int
    private let minimumConfidence: Float = 0.3
    
    // MARK: - Callbacks
    var onRepCompleted: (() -> Void)?
    var onExerciseCompleted: (() -> Void)?
    
    init(requiredReps: Int = 10) {
        self.requiredReps = requiredReps
        super.init()
    }
    
    // MARK: - Reset
    func reset() {
        repCount = 0
        currentState = .standing
        feedbackMessage = "Stand where I can see you"
    }
    
    // MARK: - Process Frame
    nonisolated func processFrame(_ sampleBuffer: CMSampleBuffer) {
        let request = VNDetectHumanBodyPoseRequest { [weak self] request, error in
            guard let self = self else { return }
            
            if let error = error {
                Task { @MainActor in
                    self.handleNoBodyDetected()
                }
                return
            }
            
            guard let observation = request.results?.first as? VNHumanBodyPoseObservation else {
                Task { @MainActor in
                    self.handleNoBodyDetected()
                }
                return
            }
            
            Task { @MainActor in
                self.analyzeBodyPose(observation)
            }
        }
        
        let handler = VNImageRequestHandler(cmSampleBuffer: sampleBuffer, orientation: .up)
        
        do {
            try handler.perform([request])
        } catch {
            print("Vision request failed: \(error)")
        }
    }
    
    // MARK: - Analysis
    private func analyzeBodyPose(_ observation: VNHumanBodyPoseObservation) {
        // Get key points
        guard let rightHip = try? observation.recognizedPoint(.rightHip),
              let rightKnee = try? observation.recognizedPoint(.rightKnee),
              let rightAnkle = try? observation.recognizedPoint(.rightAnkle),
              let leftHip = try? observation.recognizedPoint(.leftHip),
              let leftKnee = try? observation.recognizedPoint(.leftKnee),
              let leftAnkle = try? observation.recognizedPoint(.leftAnkle)
        else {
            handleNoBodyDetected()
            return
        }
        
        // Check confidence
        let avgConfidence = (rightHip.confidence + rightKnee.confidence + leftHip.confidence + leftKnee.confidence) / 4
        confidence = avgConfidence
        
        if avgConfidence < minimumConfidence {
            handleLowConfidence()
            return
        }
        
        isBodyVisible = true
        
        // Use average of both sides for more accuracy
        let hipY = (rightHip.location.y + leftHip.location.y) / 2
        let kneeY = (rightKnee.location.y + leftKnee.location.y) / 2
        
        // Determine squat state
        // In Vision coordinates, Y increases upward
        // When squatting, hip drops closer to (or below) knee level
        let hipKneeDifference = hipY - kneeY
        
        let previousState = currentState
        
        if hipKneeDifference < 0.05 {
            // Hip is at or below knee level = squatting
            currentState = .squatting
            feedbackMessage = "Good! Now stand up"
        } else if hipKneeDifference > 0.15 {
            // Hip is well above knee = standing
            currentState = .standing
            feedbackMessage = "Go down into a squat"
        }
        
        // Count rep when transitioning from squat to stand
        if previousState == .squatting && currentState == .standing {
            countRep()
        }
    }
    
    private func countRep() {
        repCount += 1
        onRepCompleted?()
        
        if repCount >= requiredReps {
            feedbackMessage = "Great job! 🎉"
            onExerciseCompleted?()
        } else {
            feedbackMessage = "\(requiredReps - repCount) more to go!"
        }
    }
    
    private func handleNoBodyDetected() {
        isBodyVisible = false
        currentState = .unknown
        feedbackMessage = "I can't see you! Step back"
    }
    
    private func handleLowConfidence() {
        isBodyVisible = false
        feedbackMessage = "Move to a brighter area"
    }
}

// MARK: - AVCaptureVideoDataOutputSampleBufferDelegate
extension PoseDetectionService: AVCaptureVideoDataOutputSampleBufferDelegate {
    nonisolated func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        processFrame(sampleBuffer)
    }
}
```

---

### STEP 15: Implement Audio Service
**Time: 20 minutes**

**File: `Services/AudioService.swift`**
```swift
import AVFoundation
import UIKit

class AudioService {
    static let shared = AudioService()
    
    private var players: [String: AVAudioPlayer] = [:]
    
    private init() {
        setupAudioSession()
        preloadSounds()
    }
    
    private func setupAudioSession() {
        try? AVAudioSession.sharedInstance().setCategory(.playback, mode: .default, options: .mixWithOthers)
        try? AVAudioSession.sharedInstance().setActive(true)
    }
    
    private func preloadSounds() {
        // Preload system sounds as fallback
    }
    
    func playClick() {
        // Use system sound for click
        AudioServicesPlaySystemSound(1104)
    }
    
    func playDing() {
        // Success sound
        AudioServicesPlaySystemSound(1025)
        
        // Haptic feedback
        let generator = UINotificationFeedbackGenerator()
        generator.notificationOccurred(.success)
    }
    
    func playSuccess() {
        // Celebration sound
        AudioServicesPlaySystemSound(1335)
        
        // Strong haptic
        let generator = UINotificationFeedbackGenerator()
        generator.notificationOccurred(.success)
    }
}
```

---

### STEP 16: Implement App State Manager
**Time: 30 minutes**

**File: `Core/AppState.swift`**
```swift
import SwiftUI
import Combine

@MainActor
class AppState: ObservableObject {
    static let shared = AppState()
    
    // MARK: - Published State
    @Published var isOnboarded: Bool
    @Published var isShieldActive: Bool
    @Published var currentScreen: Screen = .home
    
    enum Screen {
        case onboarding
        case home
        case exercise
        case parentDashboard
    }
    
    // MARK: - Services
    let screenTimeManager = ScreenTimeManager.shared
    let shieldManager = ShieldManager.shared
    let monitoringScheduler = MonitoringScheduler.shared
    let storage = SharedStorage.shared
    
    private init() {
        isOnboarded = storage.isOnboarded
        isShieldActive = storage.isShieldActive
    }
    
    // MARK: - Actions
    func completeOnboarding() {
        storage.isOnboarded = true
        isOnboarded = true
        currentScreen = .home
        
        // Start monitoring
        monitoringScheduler.startMonitoring()
    }
    
    func startExercise() {
        currentScreen = .exercise
    }
    
    func completeExercise() {
        // Remove shield
        shieldManager.removeShield()
        isShieldActive = false
        
        // Restart monitoring timer
        monitoringScheduler.startMonitoring()
        
        // Return home
        currentScreen = .home
    }
    
    func openParentDashboard() {
        currentScreen = .parentDashboard
    }
    
    func closeParentDashboard() {
        currentScreen = .home
        
        // Restart monitoring with new settings
        monitoringScheduler.startMonitoring()
    }
    
    func refreshShieldState() {
        isShieldActive = storage.isShieldActive
    }
}
```

---

### STEP 17: Implement UI Views
**Time: 2-3 hours**

**File: `Views/Home/HomeView.swift`**
```swift
import SwiftUI

struct HomeView: View {
    @ObservedObject var appState = AppState.shared
    @State private var showPinEntry = false
    
    var body: some View {
        ZStack {
            // Background
            LinearGradient(
                colors: appState.isShieldActive ? [.red.opacity(0.1), .orange.opacity(0.1)] : [.green.opacity(0.1), .blue.opacity(0.1)],
                startPoint: .top,
                endPoint: .bottom
            )
            .ignoresSafeArea()
            
            VStack(spacing: 40) {
                Spacer()
                
                // Battery Indicator
                BatteryIndicator(isCharged: !appState.isShieldActive)
                    .frame(width: 200, height: 300)
                    .onLongPressGesture(minimumDuration: 3) {
                        showPinEntry = true
                    }
                
                // Status Text
                Text(appState.isShieldActive ? "Out of Energy!" : "Energy Full!")
                    .font(.system(size: 36, weight: .bold))
                    .foregroundColor(appState.isShieldActive ? .red : .green)
                
                Text(appState.isShieldActive ? "Time to recharge" : "Go have fun!")
                    .font(.title2)
                    .foregroundColor(.secondary)
                
                Spacer()
                
                // Action Button
                if appState.isShieldActive {
                    Button(action: { appState.startExercise() }) {
                        HStack {
                            Image(systemName: "bolt.fill")
                            Text("Start Recharging")
                        }
                        .font(.title2.bold())
                        .foregroundColor(.white)
                        .frame(maxWidth: .infinity)
                        .padding(.vertical, 20)
                        .background(Color.green)
                        .cornerRadius(16)
                    }
                    .padding(.horizontal, 40)
                }
                
                Spacer().frame(height: 50)
            }
        }
        .sheet(isPresented: $showPinEntry) {
            PinEntryView { success in
                if success {
                    appState.openParentDashboard()
                }
                showPinEntry = false
            }
        }
        .onAppear {
            appState.refreshShieldState()
        }
    }
}
```

**File: `Views/Home/BatteryIndicator.swift`**
```swift
import SwiftUI

struct BatteryIndicator: View {
    let isCharged: Bool
    
    @State private var animationAmount: CGFloat = 1.0
    
    var body: some View {
        ZStack {
            // Battery outline
            RoundedRectangle(cornerRadius: 20)
                .stroke(isCharged ? Color.green : Color.red, lineWidth: 8)
            
            // Battery cap
            Rectangle()
                .fill(isCharged ? Color.green : Color.red)
                .frame(width: 60, height: 20)
                .offset(y: -160)
            
            // Battery fill
            VStack {
                Spacer()
                Rectangle()
                    .fill(
                        LinearGradient(
                            colors: isCharged ? [.green, .green.opacity(0.7)] : [.red.opacity(0.3), .red.opacity(0.1)],
                            startPoint: .bottom,
                            endPoint: .top
                        )
                    )
                    .frame(height: isCharged ? 250 : 30)
            }
            .padding(12)
            .clipShape(RoundedRectangle(cornerRadius: 12))
            
            // Icon
            Image(systemName: isCharged ? "bolt.fill" : "bolt.slash.fill")
                .font(.system(size: 60))
                .foregroundColor(isCharged ? .yellow : .gray)
                .scaleEffect(animationAmount)
        }
        .onAppear {
            if isCharged {
                withAnimation(.easeInOut(duration: 1).repeatForever()) {
                    animationAmount = 1.1
                }
            }
        }
    }
}
```

**File: `Views/Exercise/ExerciseView.swift`**
```swift
import SwiftUI

struct ExerciseView: View {
    @ObservedObject var appState = AppState.shared
    @StateObject private var cameraService = CameraService()
    @StateObject private var poseService: PoseDetectionService
    
    @State private var showCompletion = false
    
    init() {
        let reps = SharedStorage.shared.requiredReps
        _poseService = StateObject(wrappedValue: PoseDetectionService(requiredReps: reps))
    }
    
    var body: some View {
        ZStack {
            // Camera Preview
            CameraPreviewView(session: cameraService.captureSession)
                .ignoresSafeArea()
            
            // Overlay
            VStack {
                // Feedback Message
                Text(poseService.feedbackMessage)
                    .font(.title2.bold())
                    .foregroundColor(.white)
                    .padding()
                    .background(Color.black.opacity(0.6))
                    .cornerRadius(12)
                    .padding(.top, 60)
                
                Spacer()
                
                // Rep Counter
                Text("\(poseService.repCount)")
                    .font(.system(size: 150, weight: .bold, design: .rounded))
                    .foregroundColor(.white)
                    .shadow(color: .black, radius: 10)
                
                Text("of \(SharedStorage.shared.requiredReps)")
                    .font(.title)
                    .foregroundColor(.white.opacity(0.8))
                
                Spacer()
                
                // Progress Bar
                ProgressView(value: Double(poseService.repCount), total: Double(SharedStorage.shared.requiredReps))
                    .progressViewStyle(.linear)
                    .tint(.green)
                    .frame(height: 20)
                    .padding(.horizontal, 40)
                    .padding(.bottom, 60)
            }
            
            // Body visibility warning
            if !poseService.isBodyVisible {
                VStack {
                    Spacer()
                    HStack {
                        Image(systemName: "exclamationmark.triangle.fill")
                        Text("I can't see you!")
                    }
                    .font(.headline)
                    .foregroundColor(.white)
                    .padding()
                    .background(Color.red)
                    .cornerRadius(12)
                    .padding(.bottom, 120)
                }
            }
        }
        .fullScreenCover(isPresented: $showCompletion) {
            CompletionView {
                appState.completeExercise()
            }
        }
        .onAppear {
            setupCamera()
        }
        .onDisappear {
            cameraService.stop()
        }
    }
    
    private func setupCamera() {
        Task {
            guard await cameraService.checkPermission() else { return }
            
            cameraService.setupCamera()
            cameraService.setDelegate(poseService)
            
            poseService.onRepCompleted = {
                AudioService.shared.playDing()
            }
            
            poseService.onExerciseCompleted = {
                AudioService.shared.playSuccess()
                showCompletion = true
            }
            
            cameraService.start()
        }
    }
}
```

**File: `Views/Exercise/CameraPreviewView.swift`**
```swift
import SwiftUI
import AVFoundation

struct CameraPreviewView: UIViewRepresentable {
    let session: AVCaptureSession
    
    func makeUIView(context: Context) -> UIView {
        let view = UIView(frame: .zero)
        
        let previewLayer = AVCaptureVideoPreviewLayer(session: session)
        previewLayer.videoGravity = .resizeAspectFill
        view.layer.addSublayer(previewLayer)
        
        context.coordinator.previewLayer = previewLayer
        
        return view
    }
    
    func updateUIView(_ uiView: UIView, context: Context) {
        context.coordinator.previewLayer?.frame = uiView.bounds
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator()
    }
    
    class Coordinator {
        var previewLayer: AVCaptureVideoPreviewLayer?
    }
}
```

**File: `Views/Exercise/CompletionView.swift`**
```swift
import SwiftUI

struct CompletionView: View {
    let onDismiss: () -> Void
    
    @State private var scale: CGFloat = 0.5
    @State private var opacity: Double = 0
    
    var body: some View {
        ZStack {
            Color.green.ignoresSafeArea()
            
            VStack(spacing: 30) {
                Image(systemName: "checkmark.circle.fill")
                    .font(.system(size: 120))
                    .foregroundColor(.white)
                    .scaleEffect(scale)
                
                Text("Great Job!")
                    .font(.system(size: 48, weight: .bold))
                    .foregroundColor(.white)
                
                Text("Energy Recharged!")
                    .font(.title)
                    .foregroundColor(.white.opacity(0.9))
                
                Button(action: onDismiss) {
                    Text("Continue Playing")
                        .font(.title2.bold())
                        .foregroundColor(.green)
                        .padding(.horizontal, 40)
                        .padding(.vertical, 16)
                        .background(Color.white)
                        .cornerRadius(30)
                }
                .padding(.top, 40)
            }
            .opacity(opacity)
        }
        .onAppear {
            withAnimation(.spring(response: 0.5, dampingFraction: 0.6)) {
                scale = 1.0
            }
            withAnimation(.easeIn(duration: 0.3)) {
                opacity = 1.0
            }
        }
    }
}
```

---

### STEP 18: Implement Parent Views
**Time: 1.5 hours**

**File: `Views/Parent/PinEntryView.swift`**
```swift
import SwiftUI

struct PinEntryView: View {
    let onComplete: (Bool) -> Void
    
    @State private var enteredPin = ""
    @State private var showError = false
    
    private let storage = SharedStorage.shared
    
    var body: some View {
        NavigationView {
            VStack(spacing: 30) {
                Text("Enter Parent PIN")
                    .font(.title.bold())
                
                // PIN Dots
                HStack(spacing: 20) {
                    ForEach(0..<4, id: \.self) { index in
                        Circle()
                            .fill(index < enteredPin.count ? Color.primary : Color.gray.opacity(0.3))
                            .frame(width: 20, height: 20)
                    }
                }
                .padding(.vertical, 20)
                
                if showError {
                    Text("Incorrect PIN")
                        .foregroundColor(.red)
                }
                
                // Keypad
                LazyVGrid(columns: Array(repeating: GridItem(.flexible()), count: 3), spacing: 20) {
                    ForEach(1...9, id: \.self) { number in
                        PinButton(text: "\(number)") {
                            addDigit("\(number)")
                        }
                    }
                    
                    PinButton(text: "", action: {}) // Empty
                    
                    PinButton(text: "0") {
                        addDigit("0")
                    }
                    
                    PinButton(text: "⌫") {
                        if !enteredPin.isEmpty {
                            enteredPin.removeLast()
                            showError = false
                        }
                    }
                }
                .padding(.horizontal, 40)
                
                Spacer()
            }
            .padding(.top, 40)
            .navigationBarItems(trailing: Button("Cancel") {
                onComplete(false)
            })
        }
    }
    
    private func addDigit(_ digit: String) {
        guard enteredPin.count < 4 else { return }
        enteredPin += digit
        showError = false
        
        if enteredPin.count == 4 {
            verifyPin()
        }
    }
    
    private func verifyPin() {
        if storage.parentPin == nil || storage.parentPin == enteredPin {
            onComplete(true)
        } else {
            showError = true
            enteredPin = ""
        }
    }
}

struct PinButton: View {
    let text: String
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Text(text)
                .font(.title)
                .frame(width: 70, height: 70)
                .background(Color.gray.opacity(0.2))
                .cornerRadius(35)
        }
        .disabled(text.isEmpty)
    }
}
```

**File: `Views/Parent/ParentDashboardView.swift`**
```swift
import SwiftUI
import FamilyControls

struct ParentDashboardView: View {
    @ObservedObject var appState = AppState.shared
    
    @State private var sessionDuration: Double
    @State private var requiredReps: Double
    @State private var selection: FamilyActivitySelection
    @State private var showAppPicker = false
    @State private var newPin = ""
    
    private let storage = SharedStorage.shared
    
    init() {
        _sessionDuration = State(initialValue: Double(SharedStorage.shared.sessionDuration))
        _requiredReps = State(initialValue: Double(SharedStorage.shared.requiredReps))
        _selection = State(initialValue: SharedStorage.shared.selectedApps)
    }
    
    var body: some View {
        NavigationView {
            Form {
                // App Selection
                Section("Apps to Block") {
                    Button(action: { showAppPicker = true }) {
                        HStack {
                            Text("Select Apps")
                            Spacer()
                            Text("\(selection.applicationTokens.count) apps")
                                .foregroundColor(.secondary)
                        }
                    }
                }
                
                // Time Settings
                Section("Session Duration") {
                    VStack(alignment: .leading) {
                        Text("\(Int(sessionDuration)) minutes")
                            .font(.headline)
                        Slider(value: $sessionDuration, in: 15...120, step: 5)
                    }
                }
                
                // Exercise Settings
                Section("Exercise Difficulty") {
                    VStack(alignment: .leading) {
                        Text("\(Int(requiredReps)) squats")
                            .font(.headline)
                        Slider(value: $requiredReps, in: 5...25, step: 1)
                    }
                }
                
                // PIN Settings
                Section("Security") {
                    SecureField("Set New PIN (4 digits)", text: $newPin)
                        .keyboardType(.numberPad)
                        .onChange(of: newPin) { _, newValue in
                            if newValue.count > 4 {
                                newPin = String(newValue.prefix(4))
                            }
                        }
                    
                    if newPin.count == 4 {
                        Button("Save PIN") {
                            storage.parentPin = newPin
                            newPin = ""
                        }
                        .foregroundColor(.green)
                    }
                }
                
                // Emergency
                Section("Emergency") {
                    Button("Unlock Apps Now") {
                        ShieldManager.shared.removeShield()
                        appState.refreshShieldState()
                    }
                    .foregroundColor(.red)
                }
            }
            .navigationTitle("Parent Settings")
            .navigationBarItems(trailing: Button("Done") {
                saveSettings()
                appState.closeParentDashboard()
            })
            .familyActivityPicker(isPresented: $showAppPicker, selection: $selection)
        }
    }
    
    private func saveSettings() {
        storage.sessionDuration = Int(sessionDuration)
        storage.requiredReps = Int(requiredReps)
        storage.selectedApps = selection
    }
}
```

---

### STEP 19: Implement Main App Entry
**Time: 30 minutes**

**File: `App/ActiveUnlockApp.swift`**
```swift
import SwiftUI

@main
struct ActiveUnlockApp: App {
    @StateObject private var appState = AppState.shared
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
        }
    }
}
```

**File: `App/ContentView.swift`**
```swift
import SwiftUI

struct ContentView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        Group {
            if !appState.isOnboarded {
                OnboardingContainerView()
            } else {
                switch appState.currentScreen {
                case .home:
                    HomeView()
                case .exercise:
                    ExerciseView()
                case .parentDashboard:
                    ParentDashboardView()
                case .onboarding:
                    OnboardingContainerView()
                }
            }
        }
    }
}
```

---

### STEP 20: Implement Onboarding
**Time: 1 hour**

**File: `Views/Onboarding/OnboardingContainerView.swift`**
```swift
import SwiftUI

struct OnboardingContainerView: View {
    @State private var currentStep = 0
    
    var body: some View {
        TabView(selection: $currentStep) {
            WelcomeView(onContinue: { currentStep = 1 })
                .tag(0)
            
            PermissionRequestView(onContinue: { currentStep = 2 })
                .tag(1)
            
            PinSetupView()
                .tag(2)
        }
        .tabViewStyle(.page(indexDisplayMode: .never))
        .animation(.easeInOut, value: currentStep)
    }
}
```

**File: `Views/Onboarding/WelcomeView.swift`**
```swift
import SwiftUI

struct WelcomeView: View {
    let onContinue: () -> Void
    
    var body: some View {
        VStack(spacing: 40) {
            Spacer()
            
            Image(systemName: "bolt.shield.fill")
                .font(.system(size: 100))
                .foregroundColor(.green)
            
            Text("ActiveUnlock")
                .font(.largeTitle.bold())
            
            Text("Screen time that keeps kids active")
                .font(.title3)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal)
            
            Spacer()
            
            Button(action: onContinue) {
                Text("Get Started")
                    .font(.title2.bold())
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(Color.green)
                    .cornerRadius(16)
            }
            .padding(.horizontal, 40)
            .padding(.bottom, 50)
        }
    }
}
```

**File: `Views/Onboarding/PermissionRequestView.swift`**
```swift
import SwiftUI

struct PermissionRequestView: View {
    let onContinue: () -> Void
    
    @ObservedObject var screenTimeManager = ScreenTimeManager.shared
    @State private var isRequesting = false
    
    var body: some View {
        VStack(spacing: 40) {
            Spacer()
            
            Image(systemName: "lock.shield")
                .font(.system(size: 80))
                .foregroundColor(.blue)
            
            Text("Permission Required")
                .font(.title.bold())
            
            Text("We need Screen Time access to monitor and manage app usage.")
                .font(.body)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal, 40)
            
            Spacer()
            
            if screenTimeManager.authorizationStatus == .approved {
                Button(action: onContinue) {
                    HStack {
                        Image(systemName: "checkmark.circle.fill")
                        Text("Permission Granted - Continue")
                    }
                    .font(.title3.bold())
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(Color.green)
                    .cornerRadius(16)
                }
                .padding(.horizontal, 40)
            } else {
                Button(action: requestPermission) {
                    if isRequesting {
                        ProgressView()
                            .tint(.white)
                    } else {
                        Text("Grant Permission")
                    }
                }
                .font(.title2.bold())
                .foregroundColor(.white)
                .frame(maxWidth: .infinity)
                .padding()
                .background(Color.blue)
                .cornerRadius(16)
                .disabled(isRequesting)
                .padding(.horizontal, 40)
            }
            
            if screenTimeManager.authorizationStatus == .denied {
                Text("Permission denied. Please enable in Settings > Screen Time.")
                    .font(.caption)
                    .foregroundColor(.red)
                    .multilineTextAlignment(.center)
                    .padding(.horizontal)
            }
            
            Spacer().frame(height: 50)
        }
    }
    
    private func requestPermission() {
        isRequesting = true
        Task {
            await screenTimeManager.requestAuthorization()
            isRequesting = false
        }
    }
}
```

**File: `Views/Onboarding/PinSetupView.swift`**
```swift
import SwiftUI
import FamilyControls

struct PinSetupView: View {
    @ObservedObject var appState = AppState.shared
    
    @State private var pin = ""
    @State private var confirmPin = ""
    @State private var step: SetupStep = .enterPin
    @State private var showAppPicker = false
    @State private var selection = FamilyActivitySelection()
    @State private var showError = false
    
    enum SetupStep {
        case enterPin
        case confirmPin
        case selectApps
    }
    
    private let storage = SharedStorage.shared
    
    var body: some View {
        VStack(spacing: 30) {
            Spacer()
            
            switch step {
            case .enterPin:
                pinEntrySection(title: "Create a 4-digit PIN", pin: $pin) {
                    if pin.count == 4 {
                        step = .confirmPin
                    }
                }
                
            case .confirmPin:
                pinEntrySection(title: "Confirm your PIN", pin: $confirmPin) {
                    if confirmPin.count == 4 {
                        if confirmPin == pin {
                            storage.parentPin = pin
                            step = .selectApps
                        } else {
                            showError = true
                            confirmPin = ""
                        }
                    }
                }
                
                if showError {
                    Text("PINs don't match. Try again.")
                        .foregroundColor(.red)
                }
                
            case .selectApps:
                VStack(spacing: 20) {
                    Image(systemName: "apps.iphone")
                        .font(.system(size: 60))
                        .foregroundColor(.purple)
                    
                    Text("Select Apps to Block")
                        .font(.title.bold())
                    
                    Text("Choose which apps will be blocked when time runs out.")
                        .foregroundColor(.secondary)
                        .multilineTextAlignment(.center)
                        .padding(.horizontal)
                    
                    Button("Choose Apps") {
                        showAppPicker = true
                    }
                    .font(.title3.bold())
                    .foregroundColor(.white)
                    .padding(.horizontal, 40)
                    .padding(.vertical, 16)
                    .background(Color.purple)
                    .cornerRadius(12)
                    
                    if !selection.applicationTokens.isEmpty {
                        Text("\(selection.applicationTokens.count) apps selected")
                            .foregroundColor(.green)
                    }
                }
            }
            
            Spacer()
            
            if step == .selectApps && !selection.applicationTokens.isEmpty {
                Button(action: completeSetup) {
                    Text("Finish Setup")
                        .font(.title2.bold())
                        .foregroundColor(.white)
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(Color.green)
                        .cornerRadius(16)
                }
                .padding(.horizontal, 40)
            }
            
            Spacer().frame(height: 50)
        }
        .familyActivityPicker(isPresented: $showAppPicker, selection: $selection)
    }
    
    @ViewBuilder
    private func pinEntrySection(title: String, pin: Binding<String>, onChange: @escaping () -> Void) -> some View {
        VStack(spacing: 20) {
            Image(systemName: "lock.fill")
                .font(.system(size: 60))
                .foregroundColor(.blue)
            
            Text(title)
                .font(.title.bold())
            
            HStack(spacing: 20) {
                ForEach(0..<4, id: \.self) { index in
                    Circle()
                        .fill(index < pin.wrappedValue.count ? Color.blue : Color.gray.opacity(0.3))
                        .frame(width: 20, height: 20)
                }
            }
            
            // Hidden TextField for keyboard
            TextField("", text: pin)
                .keyboardType(.numberPad)
                .opacity(0)
                .frame(width: 0, height: 0)
                .onChange(of: pin.wrappedValue) { _, _ in
                    if pin.wrappedValue.count > 4 {
                        pin.wrappedValue = String(pin.wrappedValue.prefix(4))
                    }
                    onChange()
                }
            
            // Keypad
            LazyVGrid(columns: Array(repeating: GridItem(.flexible()), count: 3), spacing: 15) {
                ForEach(1...9, id: \.self) { num in
                    Button("\(num)") {
                        if pin.wrappedValue.count < 4 {
                            pin.wrappedValue += "\(num)"
                            onChange()
                        }
                    }
                    .font(.title)
                    .frame(width: 60, height: 60)
                    .background(Color.gray.opacity(0.2))
                    .cornerRadius(30)
                }
                
                Spacer().frame(width: 60, height: 60)
                
                Button("0") {
                    if pin.wrappedValue.count < 4 {
                        pin.wrappedValue += "0"
                        onChange()
                    }
                }
                .font(.title)
                .frame(width: 60, height: 60)
                .background(Color.gray.opacity(0.2))
                .cornerRadius(30)
                
                Button(action: {
                    if !pin.wrappedValue.isEmpty {
                        pin.wrappedValue.removeLast()
                    }
                }) {
                    Image(systemName: "delete.left")
                        .font(.title2)
                }
                .frame(width: 60, height: 60)
            }
            .padding(.horizontal, 60)
        }
    }
    
    private func completeSetup() {
        storage.selectedApps = selection
        appState.completeOnboarding()
    }
}
```

---

### STEP 21: Add Info.plist Entries
**Time: 10 minutes**

Add these to your main app's Info.plist:

```xml
<key>NSCameraUsageDescription</key>
<string>We use the camera to verify exercise completion</string>

<key>UIBackgroundModes</key>
<array>
    <string>processing</string>
</array>
```

---

### STEP 22: Test Checklist
**Time: 2-3 days**

| Test | Expected Result |
|------|-----------------|
| Fresh install onboarding | All 3 steps complete |
| FamilyControls permission | Granted successfully |
| App selection | Apps appear in picker |
| Shield activation (set 2 min for testing) | Selected apps show shield |
| Shield "Recharge Now" button | Opens main app |
| Squat detection | Counts go up on good form |
| Rep completion | Confetti shows |
| Shield removal | Apps accessible again |
| Timer restart | Shield returns after interval |
| Parent PIN entry | Correct PIN opens settings |
| Settings changes | Persist after app restart |
| Long press logo | Opens PIN entry |

---

## Build Commands Summary

```bash
# Open in Xcode
open ActiveUnlock.xcodeproj

# Build for device (Screen Time doesn't work in Simulator)
xcodebuild -scheme ActiveUnlock -destination 'platform=iOS,name=Your iPad'

# Archive for TestFlight
xcodebuild archive -scheme ActiveUnlock -archivePath ./build/ActiveUnlock.xcarchive
```

---

## Final Deliverables

After completing all steps, you will have:

1. ✅ Working app with 3 targets
2. ✅ FamilyControls integration
3. ✅ Custom shield UI
4. ✅ Background time monitoring
5. ✅ AI-powered squat detection
6. ✅ Parent dashboard with PIN protection
7. ✅ Full onboarding flow