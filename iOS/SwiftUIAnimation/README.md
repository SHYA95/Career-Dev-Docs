# SwiftUI Animation  🎬
## 🔄 Basic Animations

```Swift
// Basic animation with default parameters
.animation(.default)

// Animation with duration
.animation(.easeInOut(duration: 1.0))

// Animation with explicit value change tracking
.animation(.default, value: isExpanded)
```

## 🌊 Spring Animations

```Swift
// Basic spring
animation(.spring())

// Custom spring
animation(.spring(response: 0.5, dampingFraction: 0.5, blendDuration: 0))

// Interpolating spring
animation(.interpolatingSpring(stiffness: 50, damping: 1))
```
## ⏱️ Timing Curve Animations

```Swift
// Ease In
animation(.easeIn)

// Ease Out
animation(.easeOut)

// Ease In Out
animation(.easeInOut)

// Linear
animation(.linear)

// Custom timing curve
animation(.timingCurve(0.2, 0.8, 0.2, 1.0))
```
## 🔁 Repeating Animations

```Swift
// Basic repeat
animation(.linear(duration: 2).repeatForever())

// Repeat with count
animation(.linear(duration: 2).repeatCount(3))

// Repeat with auto-reverse
animation(.linear(duration: 2).repeatForever(autoreverses: true))
```
## ⏳ Delayed Animations

```Swift
// Add delay to any animation
animation(.easeIn.delay(1))
```
## 🔗 Combined Animations

```Swift
// Chain multiple animation modifiers
animation(
    .spring(response: 0.5, dampingFraction: 0.5)
    .delay(1)
    .repeatForever()
)
```

## 🔄 Basic Transitions

```Swift
// Basic transition
transition(.slide)

// Common transitions
transition(.scale)
transition(.move(edge: .leading))
transition(.opacity)
transition(.offset(x: 100, y: 0))

// Combined transitions
transition(.asymmetric(
    insertion: .scale,
    removal: .opacity
))
```
## 🧩 Matching Geometry Effect

```Swift
// Create a namespace
@Namespace private var namespace

// Use in view
RoundedRectangle(cornerRadius: 10)
    .matchedGeometryEffect(id: "shape", in: namespace)
```
# 🛠️ Core Animation
## 📝 Basic Concepts

```Swift
// CALayer - The basic building block
let layer = CALayer()
layer.frame = CGRect(x: 0, y: 0, width: 100, height: 100)
layer.backgroundColor = UIColor.red.cgColor

// Basic Animation
let animation = CABasicAnimation(keyPath: "position.x")
animation.fromValue = 0
animation.toValue = 100
animation.duration = 1.0
layer.add(animation, forKey: "basic")
```
## 🧰 Main Animation Types
### CABasicAnimation

```Swift
// Simple position animation
let basic = CABasicAnimation(keyPath: "position")
basic.fromValue = NSValue(cgPoint: fromPoint)
basic.toValue = NSValue(cgPoint: toPoint)
basic.duration = 1
layer.add(basic, forKey: nil)
```
### CAKeyframeAnimation

```Swift
// Animation through multiple points
let keyframe = CAKeyframeAnimation(keyPath: "position")
keyframe.values = [point1, point2, point3]
keyframe.keyTimes = [0, 0.5, 1]
keyframe.duration = 2
layer.add(keyframe, forKey: nil)
```

### CAAnimationGroup

```Swift
// Combining multiple animations
let group = CAAnimationGroup()
group.animations = [animation1, animation2]
group.duration = 2
layer.add(group, forKey: nil)
```
## ⚙️ Common Animation Properties

```Swift
animation.duration = 1.0           // Duration in seconds
animation.beginTime = CACurrentMediaTime() + 1.0  // Delayed start
animation.repeatCount = Float.infinity  // Infinite repetition
animation.autoreverses = true      // Reverse animation
animation.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
```
## 🎨 Animatable Properties
- position
- bounds
- backgroundColor
- cornerRadius
- transform
- opacity
- path
- strokeEnd
- shadowOffset
- contents
### ⏲️ Animation Timing Functions

```Swift
// Built-in timing functions
CAMediaTimingFunction(name: .linear)
CAMediaTimingFunction(name: .easeIn)
CAMediaTimingFunction(name: .easeOut)
CAMediaTimingFunction(name: .easeInEaseOut)

// Custom timing function
CAMediaTimingFunction(controlPoints: 0.5, 0.1, 0.75, 0.9)
```
## 🧱 Layer Properties

```Swift
layer.shadowOpacity = 0.5
layer.shadowRadius = 5.0
layer.shadowOffset = CGSize(width: 2, height: 2)
layer.cornerRadius = 10
layer.masksToBounds = true
```
## 📋 Examples
### Rotation Animation

```Swift
let rotationAnimation = CABasicAnimation(keyPath: "transform.rotation")
rotationAnimation.fromValue = 0
rotationAnimation.toValue = CGFloat.pi * 2
rotationAnimation.duration = 2
rotationAnimation.repeatCount = .infinity
layer.add(rotationAnimation, forKey: "rotation")
```
### Scaling Animation

```Swift
let scaleAnimation = CABasicAnimation(keyPath: "transform.scale")
scaleAnimation.fromValue = 1.0
scaleAnimation.toValue = 1.5
scaleAnimation.duration = 1.0
scaleAnimation.autoreverses = true
layer.add(scaleAnimation, forKey: "scale")
```
## 📚 Resources
## 📖 Apple Documentation
Animations
View Transitions
# 🔍 Tutorials and Guides
Hacking With Swift - Animations
SwiftUI Lab - Animations
Apple WWDC Videos on Animations
Paul Hudson's YouTube Channel
## 🎓 Advanced Animation Concepts
Custom Animations
GeometryEffect
## 🌟 Key Benefits of Core Animation

### Performance:
Hardware accelerated
Runs on a separate thread
Efficient memory usage
Fine Control:
Precise timing control
Complex animation paths
Multiple animation coordination
Rich Features:
3D transformations
Particle effects
Complex layer hierarchies

## 🚀 When to Use Core Animation
Complex animations requiring precise control
Performance-critical animations
Custom drawing and effects
Layer-based animations
3D transformations
