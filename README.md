# BeginningRxcocoa

## RxSwift: Reactive Programming with Swift | raywenderlich.com
![image](https://user-images.githubusercontent.com/47273077/185172130-b3557025-c636-4a1b-8490-c900c8312b77.png)

![image](https://user-images.githubusercontent.com/47273077/190855575-bc0bcad3-eac6-4db2-8734-ba6ff14589fb.png)

# Using RxCocoa with basic UIKit controls

## Displaying the data using RxCocoa

<img width="490" alt="スクリーンショット 2022-09-17 21 05 12" src="https://user-images.githubusercontent.com/47273077/190855767-8a84487d-4604-4255-8648-bb90a05c05ad.png">

### 1. Dummy Data
ApiController
```swift
    
    func currentWeather(for city: String) -> Observable<Weather> {
      // Placeholder call
      return Observable.just(
        Weather(
          cityName: city,
          temperature: 20,
          humidity: 90,
          icon: iconNameToChar(icon: "01d"))
      )
    }
```

ViewController
```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        
        ApiController.shared.currentWeather(for: "RxSwift")
            .observeOn(MainScheduler.instance)
            .subscribe(onNext: { data in
                self.tempLabel.text = "\(data.temperature)° C"
                self.iconLabel.text = data.icon
                self.humidityLabel.text = "\(data.humidity)%"
                self.cityNameLabel.text = data.cityName
            })
            .disposed(by: bag)
```

### 2. Real Data
<img width="490" alt="スクリーンショット 2022-09-17 21 23 30" src="https://user-images.githubusercontent.com/47273077/190856519-c99f45b7-05d5-4db0-a3e2-825cdd62bf36.png">

ApiController
```swift
  // MARK: - Api Calls
  func currentWeather(for city: String) -> Observable<Weather> {
    buildRequest(pathComponent: "weather", params: [("q", city)])
      .map { data in
        try JSONDecoder().decode(Weather.self, from: data)
      }
  }
  
    private func buildRequest(method: String = "GET", pathComponent: String, params: [(String, String)]) -> Observable<Data> {
    let url = baseURL.appendingPathComponent(pathComponent)
    var request = URLRequest(url: url)
    let keyQueryItem = URLQueryItem(name: "appid", value: apiKey)
    let unitsQueryItem = URLQueryItem(name: "units", value: "metric")
    let urlComponents = NSURLComponents(url: url, resolvingAgainstBaseURL: true)!

    if method == "GET" {
      var queryItems = params.map { URLQueryItem(name: $0.0, value: $0.1) }
      queryItems.append(keyQueryItem)
      queryItems.append(unitsQueryItem)
      urlComponents.queryItems = queryItems
    } else {
      urlComponents.queryItems = [keyQueryItem, unitsQueryItem]

      let jsonData = try! JSONSerialization.data(withJSONObject: params, options: .prettyPrinted)
      request.httpBody = jsonData
    }

    request.url = urlComponents.url!
    request.httpMethod = method

    request.setValue("application/json", forHTTPHeaderField: "Content-Type")

    let session = URLSession.shared

    return session.rx.data(request: request)
  }

```

ViewController
```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        
        searchCityName.rx.text.orEmpty
          .filter { !$0.isEmpty }
          .flatMap { text in
            ApiController.shared
              .currentWeather(for: text)
              .catchErrorJustReturn(.empty)
          }
          .observeOn(MainScheduler.instance)
          .subscribe(onNext: { data in
              self.tempLabel.text = "\(data.temperature)° C"
              self.iconLabel.text = data.icon
              self.humidityLabel.text = "\(data.humidity)%"
              self.cityNameLabel.text = data.cityName
          })
          .disposed(by: bag)
          
```

![image](https://user-images.githubusercontent.com/47273077/190856552-b57b302c-20ae-43cc-aebd-bfdaf2936378.png)



--------

## Binding observables

![image](https://user-images.githubusercontent.com/47273077/190857849-1d3e551f-e8cb-457d-95ea-238223cd8376.png)

```swift

    override func viewDidLoad() {
        super.viewDidLoad()
        
    let search = searchCityName.rx.text.orEmpty
          .filter { !$0.isEmpty }
          .flatMapLatest { text in // flatMapLatest will cancel any previous newtowork requests when a new one starts
            ApiController.shared
              .currentWeather(for: text)
              .catchErrorJustReturn(.empty)
          }
          .share(replay: 1) // share makes your stream reusable and transforms a single use data source into a muti use Observable
          .observeOn(MainScheduler.instance)


        search.map { "\($0.temperature)° C" }
          .bind(to: tempLabel.rx.text)
          .disposed(by: bag)
        
        search.map(\.icon)
          .bind(to: iconLabel.rx.text)
          .disposed(by: bag)

        search.map { "\($0.humidity)%" }
          .bind(to: humidityLabel.rx.text)
          .disposed(by: bag)

        search.map(\.cityName)
          .bind(to: cityNameLabel.rx.text)
          .disposed(by: bag)
```

-----------

## Improving the code with Traits

### Improving the project with Driver and ControlProperty

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        
//        let search = searchCityName.rx.text.orEmpty
        let search = searchCityName.rx
          .controlEvent(.editingDidEndOnExit)
          .map { self.searchCityName.text ?? "" }
          .filter { !$0.isEmpty }
          .flatMapLatest { text in
            ApiController.shared
              .currentWeather(for: text)
              .catchErrorJustReturn(.empty)
          }
          .asDriver(onErrorJustReturn: .empty)


        search.map { "\($0.temperature)° C" }
          .drive(tempLabel.rx.text)
          .disposed(by: bag)

        search.map(\.icon)
          .drive(iconLabel.rx.text)
          .disposed(by: bag)

        search.map { "\($0.humidity)%" }
          .drive(humidityLabel.rx.text)
          .disposed(by: bag)
        search.map(\.cityName)
          .drive(cityNameLabel.rx.text)
          .disposed(by: bag)
          
```

![image](https://user-images.githubusercontent.com/47273077/190879682-513ad2fb-c816-4b0c-be45-e08e575c8b81.png)


