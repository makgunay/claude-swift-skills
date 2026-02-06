# UIKit Liquid Glass Patterns

## UIGlassEffect with UIVisualEffectView
```swift
import UIKit

let glassEffect = UIGlassEffect()
glassEffect.tintColor = UIColor.systemBlue.withAlphaComponent(0.3)
glassEffect.isInteractive = true

let visualEffectView = UIVisualEffectView(effect: glassEffect)
visualEffectView.frame = CGRect(x: 50, y: 100, width: 300, height: 200)
visualEffectView.layer.cornerRadius = 20
visualEffectView.clipsToBounds = true

// Add content to contentView
let label = UILabel()
label.text = "Liquid Glass"
visualEffectView.contentView.addSubview(label)
view.addSubview(visualEffectView)
```

## UIGlassContainerEffect — Multiple Elements
```swift
let containerEffect = UIGlassContainerEffect()
containerEffect.spacing = 40.0

let containerView = UIVisualEffectView(effect: containerEffect)
containerView.frame = CGRect(x: 50, y: 400, width: 300, height: 200)

let glass1 = UIVisualEffectView(effect: UIGlassEffect())
glass1.frame = CGRect(x: 20, y: 20, width: 100, height: 100)
glass1.layer.cornerRadius = 20
glass1.clipsToBounds = true

let glass2 = UIVisualEffectView(effect: UIGlassEffect())
glass2.frame = CGRect(x: 80, y: 60, width: 100, height: 100)
glass2.layer.cornerRadius = 20
glass2.clipsToBounds = true

containerView.contentView.addSubview(glass1)
containerView.contentView.addSubview(glass2)
```

## Scroll View Edge Effects
```swift
let scrollView = UIScrollView(frame: view.bounds)
scrollView.topEdgeEffect.style = .automatic
scrollView.bottomEdgeEffect.style = .hard
scrollView.leftEdgeEffect.isHidden = true
scrollView.rightEdgeEffect.isHidden = true
```

Edge effect styles: `.automatic` (system-determined), `.hard` (hard cutoff with dividing line)

## Scroll Edge Element Container Interaction
```swift
// Make overlay buttons affect scroll edge shape
let interaction = UIScrollEdgeElementContainerInteraction()
interaction.scrollView = scrollView
interaction.edge = .bottom
buttonContainer.addInteraction(interaction)
```

## Toolbar Integration
```swift
// Control shared glass background per toolbar item
let favoriteButton = UIBarButtonItem(image: UIImage(systemName: "heart"), style: .plain, target: self, action: #selector(favoriteAction))
favoriteButton.hidesSharedBackground = true  // Opt out of shared glass
```
