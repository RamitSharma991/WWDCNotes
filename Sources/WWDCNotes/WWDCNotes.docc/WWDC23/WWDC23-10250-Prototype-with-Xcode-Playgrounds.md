# Prototype with Xcode Playgrounds

Speed up feature development by prototyping new code with Xcode Playgrounds, eliminating the need to keep rebuilding and relaunching your project to verify your changes. We’ll show you how using a playground in your project or package can help you try out your code in various scenarios and take a close look at the returned values, including complex structures and user interface elements, so you can quickly iterate on a feature before integrating it into your project.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10250", purpose: link, label: "Watch Video (24 min)")

   @Contributors {
      @GitHubUser(multitudes)
   }
}



## Chapters
[3:00 - Exploring the data model and inspecting UI snapshots](https://developer.apple.com/videos/play/wwdc2023/10250/?time=180)  
[14:26 - Becoming familiar with a new package](https://developer.apple.com/videos/play/wwdc2023/10250/?time=866)  
[18:07 - Using the Live View](https://developer.apple.com/videos/play/wwdc2023/10250/?time=1087)  
[22:03 - Running the updated project](https://developer.apple.com/videos/play/wwdc2023/10250/?time=1323)

## Intro
New improvements in Xcode Playgrounds make it easier to prototype new features. 

- Rapid iterations. Allowing to skip rebuilding and relaunching the project whenever we want to prototype a new feature or try out a small change in code. 
- Exercise rare code paths. They also make it much easier to execute code that otherwise would be quite difficult to reach, like the logic responsible for generating an order summary in a shopping app.  
- Trying out new dependencies. It's also a perfect environment for trying out code before introducing new dependencies.

![Rapid iterations][iterations]

[iterations]: WWDC23-10250-iterations

The presenter is working on a small app that's meant to help with wildlife photography. At the moment, the app keeps track of the species already found and photographed. 

We build a new tab for a checklist view. Each bird has a checkbox, which will allow to track progress.

```swift
extension ChecklistView {
	
	var birdsToShow: [Bird] {
		// TODO: Narrow down the list of birds to find. 
		let birdProvider = BirdProvider(region: .northAmerica)
		return birdProvider.birds
	}
}
```

We can adjust the birdsToShow computed property in the custom ChecklistView. At the moment, it simply creates the BirdProvider type configured for North America and returns all the bird species found on the whole continent. 

To avoid the frequent rebuilding and relaunching the project and then navigating to the ChecklistView to see the changes, we will adjust this code in an Xcode Playground.

![Rapid iterations][iterations2]

[iterations2]: WWDC23-10250-iterations2

Switch to "Automatically run" in the menu that shows up when I long click the run button on the bottom bar.

![Rapid iterations][iterations3]

[iterations3]: WWDC23-10250-iterations3

This causes the playground to automatically execute the whole code.

The playgrounds added to projects have two settings enabled by default: Build Active Scheme and Import App Types. They will ensure that the active scheme is built before each playground execution and that the app target modules are automatically imported. This makes it much easier to work with the types defined within the project. 

![Rapid iterations][iterations4]

[iterations4]: WWDC23-10250-iterations4

Declaring a BirdProvider instance like in the birdsToShow property of the ChecklistView.

The result sidebar to the right of the editor shows that this declaration has generated a playground result. Use the inline result toggle to see more details.

The inline result shows the details of this BirdProvider instance along with its two properties: the birds array and the provided region.

![Rapid iterations][iterations5]

[iterations5]: WWDC23-10250-iterations5

Now in Xcode 15, each row also has a type information label, which shows a short summary of the type, and we can use the tooltips for each row to see more details.

For example, the tooltip tells us that the BirdProvider type comes from the app module and that the region enum was defined within that struct. Let's expand the array row to see more details about the birds.

Interacting with the inline result view, Xcode 15 highlights the source code that produced the result. In this case, the view displays the value assigned to the birdProvider constant. Taking a closer look at the array elements.

![Rapid iterations][iterations6]

[iterations6]: WWDC23-10250-iterations6

While the region and birds array properties show nice summaries, by default, the rows representing each bird only tell us about the array indexes. 

That's because the custom Bird type has no description defined. Lets make the Bird type conform to CustomStringConvertible protocol. Add the extension in the file that defines the Bird type.

```swift
extension Bird: CustomStringConvertible {
	public var description: String {
		return "\ (commonName) (\(scientificName))"
	}
}
```

![Rapid iterations][iterations7]

[iterations7]: WWDC23-10250-iterations7

With the new description definition, each row should show the common and scientific name of the bird. In the automatic mode of playground execution, the playground will automatically re-execute when we reopen it.

![Rapid iterations][iterations8]

[iterations8]: WWDC23-10250-iterations8

Let's take a look at other properties of the Bird type.

Some of them already have the photo property, like this Atlantic puffin.

Once I click on its row, the photo is displayed in the new split view-based user interface, which allows to see the structure of the object along with the preview.

![Rapid iterations][iterations9]

[iterations9]: WWDC23-10250-iterations9

By default, there is no such preview for custom Bird types when clicking on its row.

![Rapid iterations][iterations10]

[iterations10]: WWDC23-10250-iterations10

To achieve that, we can use the CustomPlaygroundDisplayConvertible protocol. This conformance only affects the playground representation.
Import the app module and add a simple extension that returns the photo property as the playgroundDescription.

```swift
import Foundation
import SampleBirdingApp

extension Bird: CustomPlaygroundDisplayConvertible {
	public var playgroundDescription: Any {
		return photo as Any
	}
}
```

![Rapid iterations][iterations11]

[iterations11]: WWDC23-10250-iterations11

We are explicitly casting photo to Any in the return statement. Without it, the compiler would warn that we are losing an important piece of information about the value being an optional.

This is fine in this case, as Xcode Playgrounds handles optionals by only creating a custom description for objects that don't return nil in the playgroundDescription property. In Xcode 15, the playgroundDescription returned by types conforming to CustomPlaygroundDisplayConvertible will be displayed in the split view along the object's structure.

Now, the birds that already have a photo will quickly show it without the need of expanding the row. 

![Rapid iterations][iterations12]

[iterations12]: WWDC23-10250-iterations12

Lets focus on birds that don't have the photos yet filtering out all the birds that have a photo already.

Hovering over the inline result toggles highlights their source code ranges. This makes it clear that the array is the result assigned to the birdsToFind constant and true is the latest value produced by the closure passed to the filter function.

![Rapid iterations][iterations13]

[iterations13]: WWDC23-10250-iterations13

The result sidebar tells us the number of all birds. We filter for owls. The array only has five elements now... 
We create a ChecklistView instance and add each bird one by one. As a UIView subclass, it now also shows a few properties along with the snapshot. The Value history mode, now also uses the new split-view-based user interface.

![Rapid iterations][iterations14]

[iterations14]: WWDC23-10250-iterations14

The view incorrectly says "birds" in the header for just one bird. 

![Rapid iterations][iterations15]

[iterations15]: WWDC23-10250-iterations15

To fix this we adjust the strings defined in the new String Catalog.

Bring up the context menu and select Vary By Plural.

![Rapid iterations][iterations16]

[iterations16]: WWDC23-10250-iterations16

And adjust the singular form of this string.

![Rapid iterations][iterations16a]

[iterations16a]: WWDC23-10250-iterations16a

![Rapid iterations][iterations17]

[iterations17]: WWDC23-10250-iterations17

Learn more about the new String Catalogs: 

[Discover String Catalogs](https://developer.apple.com/videos/play/wwdc2023/10155/)

The ChecklistView is now ready to use. 

![Rapid iterations][iterations18]

[iterations18]: WWDC23-10250-iterations18

Each row in the custom Checklist View had a disclosure indicator. Once I select a row in the list, it opens a simple map view. 

Let's fetch the data about the most recent sighting of the selected bird and show it on the map. We will have to adjust the sightingsToShow(for bird:) function in my ChecklistView. 

```swift
extension ChecklistView {
	func sightingToShow(for bird: Bird) -> Sighting? {
		// TODO: Use BirdSightings package to fetch the most recent sighting.
		return nil
	}
}
```

The BirdSightings package makes it easy to fetch the data from one of the citizen-science websites, where people report their sightings.The package includes documentation in form of a playground that shows a few examples.

![Rapid iterations][iterations19]

[iterations19]: WWDC23-10250-iterations19

Import the CoreLocation framework to be able to work with coordinates and the BirdSighting framework to use its API.

For the function arguments, we can simply start with the first bird from the list.

![Rapid iterations][iterations20]

[iterations20]: WWDC23-10250-iterations20

The ability to provide specific coordinates will be great for two things: testing the code and planning all the road trips.
```swift
let appleParkLocation = CLLocationCoordinate2D(latitude: 37.3348655, longitude: 122.0089409)
```

Before to introduce a network call, we will switch to the manual mode of playground execution to make sure to avoid unnecessary calls. Select Manually Run.

![Rapid iterations][iterations21]

[iterations21]: WWDC23-10250-iterations21

Let's add the fetching code.

```swift
let sightingsProvider = SightingsProvider()
let sightings = sightingsProvider.fetchSightings(of: bird.speciesCode, around: appleParkLocation)
```

Let's see the data about the sighting in my SightingMapView. We initialize it with the fetched sighting data.

For such complex user interface elements like map views, we can use the playgrounds live view to see the large, fully interactive preview. To use it, we import the PlaygroundSupport framework.

Looks like we are way too far east. 

![Rapid iterations][iterations22]

[iterations22]: WWDC23-10250-iterations22

Let's close the live view in the Editor Options and try to see where the problem was introduced.

In Xcode 15, playgrounds can now show a preview for CLLocationCoordinate2D, so let's take a look at the location property.

![Rapid iterations][iterations23]

[iterations23]: WWDC23-10250-iterations23

This might just be a matter of mixing west with east. Let's try to fix this by adding the minus sign in front of the longitude and re-executing the playground.

Now this is definitely Apple Park. 

![Rapid iterations][iterations23]

[iterations23]: WWDC23-10250-iterations23

Let's quickly bring the fetching code to the ChecklistView. Copy these three lines to the sightingsToShow function.

Instead of always using the hardcoded location here, replace it with the lastCurrentLocation from CLLocationManager. Also add the return statement with the new mostRecentSighting.

```swift
extension ChecklistView {
	var birdsToShow: [Bird] {
		let birdProvider = BirdProvider (region: .northAmerica)
		let birdsToFind = birdProvider.birds.filter { $0.photo == nil }
		let owlsToFind = birdsToFind.filter { $0.family == .owls }
		return owlsToFind
	}

	func sightingToShow(for bird: Bird) -> Sighting? {
		let sightingsProvider = SightingsProvider ()
		let sightings = sightingsProvider.fetchSightings(of: bird.speciesCode, around: lastCurrentLocation)
		let mostRecentSighting = sightings.first
		return mostRecentSighting
	}
}
```


![Rapid iterations][iterations24]

[iterations24]: WWDC23-10250-iterations24

The app can now show the most recent sighting for the selected bird. 

![Rapid iterations][iterations25]

[iterations25]: WWDC23-10250-iterations25

## Wrap Up
- Customize playground representation. We used CustomStringConvertible and CustomPlaygroundDisplayConvertible protocols to customize the representations of our custom types.  
- Take advantage of execution modes  
- We saw how adjusting the playground execution mode can speed up your workflow.  
- Work with Live Views. Using Value History mode allowed us to quickly see how our classes react to multiple inputs.  
- In this session, we used Xcode Playground's live view to quickly prototype new features of a project.

## Resources
[Have a question? Ask with tag wwdc2023-10250](https://developer.apple.com/forums/create/question?&tag1=266&tag2=237&tag3=665030)  
[Search the forums for tag wwdc2023-10250](https://developer.apple.com/forums/tags/wwdc2023-10250)

## Related Videos 
[Explore Packages and Projects with Xcode Playgrounds - WWDC20](https://developer.apple.com/videos/play/wwdc2020/10096)  
