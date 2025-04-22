# iOS Live Activity Guide 📱
## 🚀 Overview
Live Activities help users stay up to date with an app's most current data right on the Lock Screen and in Dynamic Island. 
This repository provides a comprehensive guide to implementing Live Activities in iOS 16.1+ applications.

## 📱 Requirements
- iOS 16.1+
- Xcode 14.1+
- Swift 5.7+
- Active Apple Developer account
  
## 🛠️ Setup
1. Enable Live Activities in your app
Add the capability in your app's target settings:

```Swift
// In Info.plist
<key>NSSupportsLiveActivities</key>
<true/>
2. Create an Activity Configuration
Swift

import ActivityKit
import SwiftUI

struct DeliveryAttributes: ActivityAttributes {
    public struct ContentState: Codable, Hashable {
        var status: String
        var estimatedDeliveryTime: Date
        var driverLocation: CLLocationCoordinate2D?
    }
    
    var orderNumber: String
    var restaurantName: String
}
```
## 💻 Basic Implementation
Starting a Live Activity

```Swift
Collapse
func startLiveActivity() {
    let attributes = DeliveryAttributes(
        orderNumber: "ABC123",
        restaurantName: "Pizza Palace"
    )
    
    let initialState = DeliveryAttributes.ContentState(
        status: "Preparing",
        estimatedDeliveryTime: Date().addingTimeInterval(30 * 60),
        driverLocation: nil
    )
    
    do {
        let activity = try Activity.request(
            attributes: attributes,
            contentState: initialState,
            pushType: .token
        )
        print("Live Activity started with ID: \(activity.id)")
        
        // Save the activity ID for later updates
        UserDefaults.standard.set(activity.id, forKey: "currentLiveActivityID")
        
    } catch {
        print("Error starting live activity: \(error)")
    }
}
```
## Updating a Live Activity

```Swift
Collapse
func updateLiveActivity() {
    guard let activityID = UserDefaults.standard.string(forKey: "currentLiveActivityID"),
          let activity = Activity<DeliveryAttributes>.activities.first(where: { $0.id == activityID }) else {
        return
    }
    
    let updatedState = DeliveryAttributes.ContentState(
        status: "On the way",
        estimatedDeliveryTime: Date().addingTimeInterval(15 * 60),
        driverLocation: CLLocationCoordinate2D(latitude: 37.7749, longitude: -122.4194)
    )
    
    Task {
        await activity.update(using: updatedState)
    }
}

```
## Ending a Live Activity
```Swift

Collapse
func endLiveActivity() {
    guard let activityID = UserDefaults.standard.string(forKey: "currentLiveActivityID"),
          let activity = Activity<DeliveryAttributes>.activities.first(where: { $0.id == activityID }) else {
        return
    }
    
    let finalState = DeliveryAttributes.ContentState(
        status: "Delivered",
        estimatedDeliveryTime: Date(),
        driverLocation: nil
    )
    
    Task {
        await activity.end(using: finalState, dismissalPolicy: .default)
    }
    
    // Clean up
    UserDefaults.standard.removeObject(forKey: "currentLiveActivityID")
}
```
## 🎨 Activity Configuration
Creating Live Activity Views

```Swift
Collapse
struct DeliveryActivityView: View {
    let context: ActivityViewContext<DeliveryAttributes>
    
    var body: some View {
        VStack {
            HStack {
                Text(context.attributes.restaurantName)
                    .font(.headline)
                Spacer()
                Text("Order #\(context.attributes.orderNumber)")
                    .font(.subheadline)
            }
            
            HStack {
                Image(systemName: "shippingbox.fill")
                Text(context.state.status)
                    .foregroundColor(.primary)
                    .font(.title3)
                Spacer()
            }
            
            if let eta = context.state.estimatedDeliveryTime {
                Text("Estimated delivery: \(eta, formatter: dateFormatter)")
                    .font(.caption)
            }
        }
        .padding()
    }
    
    private var dateFormatter: DateFormatter {
        let formatter = DateFormatter()
        formatter.dateStyle = .none
        formatter.timeStyle = .short
        return formatter
    }
}
```
### Register the Live Activity in your Widget Extension
```Swift

Collapse
@main
struct DeliveryWidgetBundle: WidgetBundle {
    var body: some Widget {
        DeliveryWidget()
        DeliveryLiveActivity()
    }
}

struct DeliveryLiveActivity: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: DeliveryAttributes.self) { context in
            // Lock screen/banner UI
            DeliveryActivityView(context: context)
        } dynamicIsland: { context in
            // Dynamic Island UI
            DynamicIsland {
                // Expanded UI
                DynamicIslandExpandedRegion(.leading) {
                    Text(context.attributes.restaurantName)
                        .font(.headline)
                }
                
                DynamicIslandExpandedRegion(.trailing) {
                    Text(context.state.status)
                        .font(.headline)
                }
                
                DynamicIslandExpandedRegion(.bottom) {
                    // Bottom expanded UI
                    HStack {
                        Image(systemName: "clock")
                        Text(context.state.estimatedDeliveryTime, style: .relative)
                    }
                }
            } compactLeading: {
                // Compact leading UI
                Image(systemName: "shippingbox.fill")
            } compactTrailing: {
                // Compact trailing UI
                Text(context.state.estimatedDeliveryTime, style: .timer)
                    .font(.caption2)
            } minimal: {
                // Minimal UI
                Image(systemName: "shippingbox.fill")
            }
        }
    }
}
```
# 🏝️ Dynamic Island
## Compact Leading View
Small view displayed on the leading side of the Dynamic Island when minimized.

## Compact Trailing View
Small view displayed on the trailing side of the Dynamic Island when minimized.

## Minimal View
Smallest representation shown when multiple Live Activities are active.

## Expanded View
Full view displayed when the user long-presses the Dynamic Island.

## 🌟 Best Practices
Keep content concise: Display only the most important information.
Update sparingly: Only update when there's meaningful new information.
End appropriately: End activities when they're no longer relevant.
Handle errors gracefully: Implement proper error handling for activity requests.
Test on real devices: Dynamic Island behavior is best tested on physical devices.
Consider battery impact: Frequent updates can impact battery life.
## 📋 Examples
Ride Sharing App

```Swift

struct RideAttributes: ActivityAttributes {
    struct ContentState: Codable, Hashable {
        var driverName: String
        var currentLocation: String
        var estimatedArrival: Date
        var distance: Double // in miles/km
    }
    
    var rideId: String
    var destination: String
}
Workout Tracking
Swift

struct WorkoutAttributes: ActivityAttributes {
    struct ContentState: Codable, Hashable {
        var elapsedTime: TimeInterval
        var distance: Double
        var heartRate: Int
        var calories: Int
    }
    
    var workoutType: String
    var startTime: Date
}
```
<img width="771" alt="Screenshot 2025-04-22 at 1 41 03 PM" src="https://github.com/user-attachments/assets/9ab18e33-77ba-4b9d-aae2-1760a1641832" />
- Cannot create a provisioning profile for "com.Live.Activity.Live-Activity"
- Personal development teams do not support Push Notifications and iCloud capabilities:

## How to Fix It
#### Change the bundle identifier:

- Use a simpler format (e.g., "com.yourname.LiveActivityDemo")
- Avoid periods within components
  
#### Remove Push Notifications capability:
- Go to "Signing & Capabilities" tab
- Remove "Push Notifications" if present
* Note: Remote updates won't work
  
### Simplify the project:

- Remove widget extension if just testing UI
- Create a basic demo without actual Live Activity integration
* Bypasses signing requirements
Easiest way to test without a paid account

## 📚 Resources
Apple Documentation: Live Activities
WWDC22: Meet ActivityKit
WWDC22: Design guidelines for Live Activities
Human Interface Guidelines: Live Activities
