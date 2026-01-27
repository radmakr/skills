# Networking

Always use `NetworkingKit` https://github.com/SparrowTek/NetworkingKit

This is my code and I want to use it.  


Here is a sample,

```swift
import Foundation

@NetworkingKitActor
public struct MyService: Sendable {
    private let router: NetworkRouter<MyAPI> = {
        let router = NetworkRouter<MyAPI>(decoder: .myDecoder)
        router.delegate = MyEnvironment.current.routerDelegate
        return router
    }()
    
    public init() {}
    
	public func list(pretty: Bool = false) async throws -> CategoriesResponse {
        try await router.execute(.list(pretty: pretty))
	}
}

enum MyAPI {
    case list(pretty: Bool)
}

extension MyAPI: EndpointType {
    public var baseURL: URL {
        get async {
            let environmentURL = await MyEnvironment.current.indexURL
            guard let url = URL(string: environmentURL) else { fatalError("baseURL not configured.") }
            return url
        }
    }
    
    var path: String {
        switch self {
        case .list: "/categories/list"
        }
    }
    
    var httpMethod: HTTPMethod {
        switch self {
        case .list: .get
        }
    }
    
    var task: HTTPTask {
        switch self {
        case .list(let pretty):
            var parameters: Parameters = []
            appendNil(toParameters: &parameters, withKey: "pretty", forBool: pretty)
            
            return .requestParameters(encoding: .urlEncoding(parameters: parameters))
        }
    }
    
    var headers: HTTPHeaders? {
        nil
    }
}
```

With some helper functions,

```swift
import Foundation

func append(_ any: Any?, toParameters parameters: inout Parameters, withKey key: String) {
    guard let any else { return }
    updateParamters(&parameters, with: (key, "\(any)"))
}

func appendNil(toParameters paramters: inout Parameters, withKey key: String, forBool bool: Bool) {
    guard bool else { return }
    updateParamters(&paramters, with: (key, nil))
}

fileprivate func updateParamters(_ paramters: inout Parameters, with dataToAppend: (String, String?)) {
    paramters.append(URLQueryItem(name: dataToAppend.0, value: dataToAppend.1))
}
```

I try to separate Service files into API endpoints that make sense to be together. I don’t like 1 file with all endpoints. It gets too large.

