# Swift Data

If I am asking you to modify Swift Data models, you can assume I  do not care about a migration (unless I specifically mention it).

## Setup
First we always need a way to setup our model,

```swift
import SwiftUI
@preconcurrency import SwiftData

typealias MySchema = MySchemaV1

enum MySchemaV1: VersionedSchema, Sendable {
    static let versionIdentifier = Schema.Version(1, 0, 0)
    
    static var models: [any PersistentModel.Type] {
        [
            SampleType1.self,
            SampleType2.self,
        ]
    }
}


struct MyDataContainerViewModifier: ViewModifier {
    let container: ModelContainer
    let schema = Schema(MySchema.models)
    
    init(inMemory: Bool) {
        do {
            let modelConfiguration = ModelConfiguration(schema: schema, isStoredInMemoryOnly: inMemory)
            container = try ModelContainer(for: schema, configurations: modelConfiguration)
        } catch {
            fatalError("Failed to create ModelContainer")
        }
    }
    
    func body(content: Content) -> some View {
        content
            .modelContainer(container)
    }
}

class MyModelOptions {
    static let inMemoryPersistence = false
}


extension View {
    func setupModel(inMemory: Bool = MyModelOptions.inMemoryPersistence) -> some View {
        modifier(MyDataContainerViewModifier(inMemory: inMemory))
    }
}

```

And then you create all models like this `SampleType1` example,

```swift
import Foundation
@preconcurrency import SwiftData

typealias SampleType1 = MySchema.SampleType1

extension MySchema {
    
    @Model
    class SampleType1 {
       var property1: String
       var property2: Int
   }
}
```

## ModelActor
If we ever need to take action on Swift Data data outside of a swiftUI view we need to use a model actor (which you can read about here - https://sosumi.ai/documentation/swiftdata/modelactor),

```swift
@preconcurrency import Foundation
import SwiftData

@ModelActor
actor MyModelActor {
}
```
