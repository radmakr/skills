# Preview Traits

All previews must always build and have sample data.\
This sample data will come from Swift Data’s @Query macro.\
This gets data in swiftUI previews via a preview trait.\

Here is some sample code to demonstrate how that works.\


```swift
import Foundation
@preconcurrency import SwiftData
import SwiftUI

struct SampleData: PreviewModifier {
    
    static func makeSharedContext() throws -> ModelContainer {
        let container = try ModelContainer(for: FMPodcast.self, configurations: ModelConfiguration(isStoredInMemoryOnly: true))
        
        if let podcastResponse: PodcastResponse = object(resourceName: "podcastresponse"),
           let podcast = podcastResponse.feed {
            container.mainContext.insert(FMPodcast(podcast: podcast))
        }
        
        try container.mainContext.save()
        return container
    }
    
    func body(content: Content, context: ModelContainer) -> some View {
        content.modelContainer(context)
    }
}

extension PreviewTrait where T == Preview.ViewTraits {
    @MainActor static var sampleData: Self = .modifier(SampleCompositePodcastAndEpisodeData())
}

fileprivate func object<c: Codable>(resourceName: String) -> c? {
    guard let file = Bundle.main.url(forResource: resourceName, withExtension: "json"),
          let data = try? Data(contentsOf: file) else { return nil }
    let decoder = JSONDecoder()
    decoder.keyDecodingStrategy = .convertFromSnakeCase
    return try? decoder.decode(c.self, from: data)
}

```


In the example above there must be a swift data model named `FMPodcast` and a JSON file named `podcastresponse.json` which in this case decodes to a `PodcastResponse` swift codable struct.

Next in the swiftUI preview that needs the sample data you need your macro to be like this,

```swift
#Preview(traits: .sampleData) {
   MyView()
}
```

And in `MyView` we would have a `@Query` macro,

```swift
struct MyView: View {
   @Query private var podcasts: [FMPodcast]
   
   var body: some View {
      ZStack{}
   }
}
```


I think the rest of this pattern (like decoding the struct from the JSON should be something you already understand)