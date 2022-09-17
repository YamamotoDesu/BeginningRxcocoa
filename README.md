# BeginningRxcocoa

## RxSwift: Reactive Programming with Swift | raywenderlich.com
![image](https://user-images.githubusercontent.com/47273077/185172130-b3557025-c636-4a1b-8490-c900c8312b77.png)

![image](https://user-images.githubusercontent.com/47273077/190855575-bc0bcad3-eac6-4db2-8734-ba6ff14589fb.png)

# Using RxCocoa with basic UIKit controls

## Displaying the data using RxCocoa

<img width="490" alt="スクリーンショット 2022-09-17 21 05 12" src="https://user-images.githubusercontent.com/47273077/190855767-8a84487d-4604-4255-8648-bb90a05c05ad.png">

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
