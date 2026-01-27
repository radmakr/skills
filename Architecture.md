# Architecture

When you are designing the code architecture keep this all in mind.

I like to have a few concepts followed.

No view models but lean heavy into the provided macros and property wrappers available for swiftUI including @Query when dealing with Swift Data.

## Views and Presenters
I like to use the idea of Views and Presenters. A view will have logic for what is displayed on screen, so a typical SwiftUI view and a presenter is also technically a SwiftUI `View` but it deals more with `NavigationStacks`, `sheets`, `fullScreenModals` , `navigationDestination`, etc.

## State
Then we have the idea of `State` objects. These are both `@MainActor` and `@Observable` class objects. Views and presenters will have access to State objects after they are passed into the SwiftUI environment.

States will control sheets, fullScreenCovers, NavigationStack paths, etc.

This enum lives inside a state class,

```swift
enum Sheet: Identifiable {
        case send
        case receive
        case open(Transaction)
        case settings
        case moreActivity

        var id: Int {
            switch self {
            case .send: 1
            case .receive: 2
            case .open: 3
            case .settings: 4
            case .moreActivity: 5
            }
        }
    }
    
    var sheet: Sheet?
}
```

And this pattern will be used for fullScreenCovers and TabView tabs, etc.

The app has an `AppState` like this,

```swift
import SwiftUI

@Observable
@MainActor
public class AppState {
    enum Route: Int, Identifiable {
        case wallet
        case setup
        case config
        
        var id: Int { rawValue }
    }
    
    var route: Route = .setup
    
    @ObservationIgnored
    lazy var walletState = WalletState(parentState: self)
    @ObservationIgnored
    lazy var setupState = SetupState(parentState: self)
```

You can see here it has a Route enum which controls where the app should route to in the `AppPresenter` 

```swift
import SwiftUI

struct AppPresenter: View {
    @Environment(AppState.self) private var state
    @Environment(\.nwc) private var nwc
    
    var body: some View {
        @Bindable var state = state
        
        Group {
            switch state.route {
            case .setup:
                SetupPresenter()
                    .environment(state.setupState)
            case .config:
                ConfigView()
            case .wallet:
                WalletPresenter()
                    .environment(state.walletState)
            }
        }
        .onOpenURL()
        .onAppear(perform: determineRoute)
        .logoutTracker()
    }
    
    private func determineRoute() {
        state.determineRoute()
    }
}

#Preview {
    AppPresenter()
        .setupModel()
        .environment(\.nwc, NWC())
        .environment(AppState())
        .environment(ScanQRCodeState(parentState: SetupState(parentState: .init())))
}
```

That onOpenURL leads back to the state objects eventually,

```swift
import SwiftUI

fileprivate struct OnOpenURL: ViewModifier {
    @Environment(AppState.self) private var state
    @State private var url: URL?
    @State private var onOpenURLTrigger = PlainTaskTrigger()
    
    func body(content: Content) -> some View {
        content
            .onOpenURL { url = $0 }
            .onChange(of: url, triggerOnOpenUrl)
            .task($onOpenURLTrigger) { await openUrl() }
    }
    
    private func triggerOnOpenUrl() {
        onOpenURLTrigger.trigger()
    }
    
    private func openUrl() async {
        guard let url else { return }
        await state.onOpenURL(url)
    }
}

extension View {
    func onOpenURL() -> some View {
        modifier(OnOpenURL())
    }
}
```

You will also notice `@ObservationIgnored` child states,

```swift
import Foundation
import SwiftUI

@Observable
@MainActor
class WalletState {    
   unowned let parentState: AppState
    
    init(parentState: AppState) {
        self.parentState = parentState
    }
```

The important thing here is to give it an unowned parentState property, `unowned let parentState: AppState`

## Misc
Never force unwrap an optional. Ever.