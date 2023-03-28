# Clean-Architecture-MVVM-Study

![](https://velog.velcdn.com/images/qnm83/post/e3f916d6-f97a-4755-b96d-1cfcf799768f/image.png)

## 의존성 규칙

안쪽 layer은 바깥 layer에 의존성을 갖으면 안 됩니다.

![](https://velog.velcdn.com/images/qnm83/post/87a5f075-293a-4e4e-bcb3-6d1aacd2ef6a/image.png)

## Domain Layer(Business Logic)

- 가장 안쪽 layer로 다른 layer들에 의존성이 없습니다.
- Entity(Business Model), Use Case, Repository Interface로 구성 
- 이러한 분리는 테스트하기 용이하게 만듭니다.
- 좋은 아키텍처가 Use Case 중심으로 설계하는 이유는 frame work, tool, eviroment에 전념하지 않게 만들기 위해서 입니다.
  - [Screaming Architecture](https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html)
  
## Presentation Layer
 
- UI가 있습니다.
- Use Case에 의해 실행되는 ViewModel(Presenter)로 View를 coordinate 합니다.
- Presentation layer은 Domain Layer에만 의존합니다.

## Data Layer

- Repository 구현체 그리고 Data Sources로 구성됩니다.
- Repository는 서로 다른 Data Source들을 조정하는 역할을 합니다.
- Data Source는 Remote, Local일 수 있습니다.
- Data Layer은 Domain Layer에만 의존합니다.
- JSON Data를 매핑하는 부분도 추가할 수 있습니다.(DTO?)

<br>

![](https://velog.velcdn.com/images/qnm83/post/b3d3eb10-d3a6-4746-8cbe-e4ad1cf00fcf/image.png)

`의존성 방향`과, `데이터 흐름`을 표시한 그림입니다. Repository Interface(Protocol)을 사용하는 부분에서 `의존성 역전`을 볼 수 있습니다.


## Data Flow

1. **View**(UI)는 **View Model**(Presenter)로 부터 메소드를 실행합니다.

2. **View Model**은 **Use Case**를 실행합니다.

3. **Use Case**는 **User**와 **Repository**로 부터 data를 결합합니다.

4. 각각의 **Repository**는 **Remote Data**(Network), **Persistent DB** 로 부터 데이터를 반환합니다.(Remote or Cached).

5. Repository에서 반환된 정보는 다시 **View**(UI)로 흘러갑니다.

## DependencyDirection

![](https://velog.velcdn.com/images/qnm83/post/87a5f075-293a-4e4e-bcb3-6d1aacd2ef6a/image.png)

### Presentation Layer(MVVM)

- ViewModel(Presenter)
- View(UI)

### Domain Layer

- Entity
- Use Case
- Repositroy Interface

### Data Repository Layer

- Repository Implementation
- API(Network)
- Persistence DB


<br>

---

<br>

## Example Project: Movies App

### Domain Layer

- Entity, 영화를 찾는 Use Case, Data Repository Interface로 구성

#### Note

- UseCase protocol을 사용해서 UseCase 구현체를 구현해야 합니다.
  ```swift
  protocol UseCase {
      @discardableResult
      func start() -> Cancellable?
  }
  ```
- Use Case는 **Interactor**라고도 불립니다.

### Presentation Layer

- View Model, View로 구성
- View Model에서는 UIKit을 import하지 않습니다.
  - UI Framework에 의존하지 않아 refator해도 그대로 사용할 수 있습니다.


#### Note

- **ViewModelInput**, **ViewModelOutput** interface를 사용면 ViewModel을 쉽게 mocking 해서 **View**Controller를 테스트 하기 쉽게 만듭니다.
- **Action** 클로저는 **FlowCoordinator**가 화면 전환하도록 합니다. 
- ation은 struct를 사용해서 구현합니다.

<br>

- presentation logic을 위한 **Flow Coordinator**를 사용해서 View Controller의 책임을 덜어줍니다.
- **Flow**는 **강한 참조**를 가져야합니다.(with action closure, self function) 필요시 해제되지 않고 **Flow**를 유지하기 위해


### Data Layer

- Repository를 포함합니다.
- Domain Layer에서 정의한 interface들을 채택해서 구현체를 만듭니다(Dependency Inversion)
- JSON data를 mapping 하는 것과 CoreDataEntity들도 포함합니다.


#### note 

- Data Transfer Object(DTO)는 JSON response를 Domain으로 mapping하기 위해 사용됩니다.
- reponse를 cache하려면 DTO in persistent storage를 통해 Data를 전달하면 됩니다.(DTO -> NSManagedObject)

### Infrastructure Layer(Network)

- network framework를 둘러싼 wrapper 입니다.
- network parameter들을 정의 합니다.

### MVVM

![](https://velog.velcdn.com/images/qnm83/post/d2717d15-5d42-466f-a347-712ef86c4111/image.png)


- MVVVM를 통해 UI와 Domain을 관심사의 분리를 구현합니다.
- **View*와 **View Model**의 **Data Binding**은 클로저, delegate, Observable(RxSwift, Combine)으로 구현될 수 있습니다.

### MVVMs Communication

![](https://velog.velcdn.com/images/qnm83/post/aedf40c2-e15c-4342-ab34-ebb7109ea45b/image.png)

Delegation과 Closure을 사용해서 소통합니다.

### Dependency Injection Container

- **Dependency Injection**은 한 객체가 다른 객체의 의존성을 제공하는 방법입니다.
- DIContainer는 모든 injection들의 중심 장치입니다.

<br>

- dependency생성을 DiContainer에 위임하는 프로토콜을 선언
  1. Dependency Protocol을 정의하고 이 프로토콜을 채택할 DIContainer을 만듭니다. 
  2. DIContainer을 FlowCoordinator에 주입합니다.
  3. FlowCoordinator로 ViewController을 생성합니다.
  
```swift
// Define Dependencies protocol for class or struct that needs it
protocol MoviesSearchFlowCoordinatorDependencies  {
    func makeMoviesListViewController() -> MoviesListViewController
}

class MoviesSearchFlowCoordinator {
    
    private let dependencies: MoviesSearchFlowCoordinatorDependencies

    init(dependencies: MoviesSearchFlowCoordinatorDependencies) {
        self.dependencies = dependencies
    }
...
}

// Make the DIContainer to conform to this protocol
extension MoviesSceneDIContainer: MoviesSearchFlowCoordinatorDependencies {}

// And inject MoviesSceneDIContainer `self` into class that needs it
final class MoviesSceneDIContainer {
    ...
    // MARK: - Flow Coordinators
    func makeMoviesSearchFlowCoordinator(navigationController: UINavigationController) -> MoviesSearchFlowCoordinator {
        return MoviesSearchFlowCoordinator(navigationController: navigationController,
                                           dependencies: self)
    }
}
```

- closure 사용

```swift
// Define makeMoviesListViewController closure that returns MoviesListViewController
class MoviesSearchFlowCoordinator {
   
    private var makeMoviesListViewController: () -> MoviesListViewController

    init(navigationController: UINavigationController,
         makeMoviesListViewController: @escaping () -> MoviesListViewController) {
        ...
        self.makeMoviesListViewController = makeMoviesListViewController
    }
    ...
}

// And inject MoviesSceneDIContainer's `self`.makeMoviesListViewController function into class that needs it
final class MoviesSceneDIContainer {
    ...
    // MARK: - Flow Coordinators
    func makeMoviesSearchFlowCoordinator(navigationController: UINavigationController) -> MoviesSearchFlowCoordinator {
        return MoviesSearchFlowCoordinator(navigationController: navigationController,
                                           makeMoviesListViewController: self.makeMoviesListViewController)
    }
    
    // MARK: - Movies List
    func makeMoviesListViewController() -> MoviesListViewController {
        ...
    }
}
```


<br>
<br>

## 참고

- [Clean Architecture and MVVM on iOS](https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3)
