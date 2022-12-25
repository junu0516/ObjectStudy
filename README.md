# Ch1. 객체, 설계

- “이론이 먼저일까, 실무가 먼저일까?” 라는 질문에 대해 글래스(Robert L. Glass)는 실무가 먼저라고 답함
- 어느 분야가 되었던, 아무것도 없는 상태에서는 이론을 새롭게 정립하기 전에 실무를 통해 관찰한 결과를 바탕으로 이론을 정립해 나가는 것이 최선임
- 소프트웨어 공학은 상대적으로 역사가 짧기 때문에, 소프트웨어 설계 혹은 유지보수에 있어 많은 경우에 이론보다 실무가 더 앞서있음
    - ‘훌륭한 설계’라는 것에 대한 최초의 이론은 1970년대가 되어서야 어느정도 정립되었으며, 이 역시 특정 이론에서 출발한 것이 아닌 수많은 실무에서의 기법들을 이론화하고 정립한 것의 결과임
    - 유지보수의 측면에서는 당장 효과적인 이론이 발표된 적은 거의 없으며, 오히려 ‘소프트웨어 유지보수’라는 키워드는 이론이라는 것과 거리가 멀어보임

# 1. 티켓 판매 어플리케이션 구현하기

## 1-1. 초대장, 티켓, 관람객 객체 정의하기

- 어떤 소극장에서 추첨을 통해 선정된 관람객에게 무료 초대장을 발송하기 위한 이벤트를 진행한 상황을 가정
- 공연 당일에는 이벤트에 당첨된 관람객과, 그렇지 못한 관람객을 다른 방식으로 입장시켜야 할 필요가 있음
    - 이벤트에 당첨된 관람객은 초대장을 티켓으로 교환한 후 입장
    - 이벤트에 당첨되지 않은 관람객은 티켓을 구매한 후 입장
    - 우선적으로 이벤트 당첨 여부를 확인 후, 그렇지 않은 경우에는 티켓을 구매하도록 해야 함
- 위와 같은 상황을 위해 ‘초대장’이라는 개념을 아래와 같은 구조체로 정의

```swift
struct Invitation {

    let when: Date //get-only property
}
```

- 또한 공연 관람을 위해 필요한 티켓을 구조체로 구현

```swift
struct Ticket {
    
    let fee: Int //get-only property
}
```

- 이벤트 당첨자는 티켓으로 교환 가능한 초대장을 가지고 있을 것이고, 이벤트에 당첨되지 않은 관람객은 티켓을 구매하기 위한 현금을 가지고 있을 것임
- 따라서 관람객은 크게 초대장, 현금, 티켓의 3가지 소지품을 가질 수 있는 상황이며, 이를 위해 Bag이라는 구조체를 아래처럼 정의해볼 수 있음

```swift
struct Bag {
    
    let amount: Double
    let invitation: Invitation
    let ticket: Ticket
}
```

- 이후 관람객의 초대장 보유 여부 판단, 티켓 소유여부 판단, 현금(amount)의 증감 이라는 4가지 행위를 아래와 같이 연산프로퍼티와 함수로 구현
    - 구조체에서 self는 불변(immutable)이기 때문에, 프로퍼티값의 변경이 필요한 함수는 mutating 키워드를 선언
    - 초대장과 티켓의 소유 여부 판단은 함수대신 연산프로퍼티를 통해, 특정 속성값이 nil인지를 판단하도록 함
    - 이 외에, 우선 외부에서 현금과 초대장, 티켓 속성에 대해 직접 접근할 일이 없기 때문에 get-only하게 구현하던 것에서 완전히 private하도록 접근제어를 수정

```swift
struct Bag {
    
  private var amount: Int
  private var invitation: Invitation?
  private var ticket: Ticket?
  
  var hasInvitation: Bool {
      invitation == nil
  }
  
  var hasTicket: Bool {
      ticket == nil
  }
  
  mutating func setTicket(_ ticket: Ticket) {
      self.ticket = ticket
  }
  
  mutating func minusAmount(_ amount: Int) {
      self.amount -= amount
  }
  
  mutating func plusAmount(_ amount: Int) {
      self.amount += amount
  }
}
```

- 이벤트에 당첨된 관람객의 Bag 인스턴스에는 현금과 초대장에 대한 속성값이 있을 것이고, 그렇지 않은 관람객이라면 초대장 속성에 대해서는 속성값이 nil이 될 것임
- 따라서 Bag 인스턴스를 생성할 경우에는, 현금과 초대장을 함께 보관하거나, 혹은 현금만 보관하거나 둘 중 하나일 것이기 때문에 생성자를 2개 정의해볼 수 있음

```swift
struct Bag {
    /*
            속성, 함수는 위와 동일하고 생성자만 아래와 같이 2개 정의
        */
    init(amount: Int) {
        self.amount = amount
    }
    
    init(invitation: Invitation, amount: Int) {
        self.invitation = invitation
        self.amount = amount
    }
    
   //중략
}
```

- 이후 Bag 인스턴스를 소유하게 되는, 관람객이라는 개념을 구현한 Audience 구조체를 아래와 같이 정의
    - 관람객은 오로지 소지품을 보관하는 Bag 인스턴스를 소유하게 되며, 생성자도 이를 주입받기 위한 하나만 존재할 것임

```swift
struct Audience {
    //구조체의 경우 생성자가 자동으로 정의되기 때문에 굳이 명시하지 않음
  var bag: Bag 
}
```

## 1-2. 매표소와 판매원, 소극장 객체 정의하기

- 관람객은 소극장에 입장하기 위해 매표소에서 초대장을 티켓으로 교환하거나, 티켓을 현금을 내고 구매해야 함
- 이를 위해 매표소에서는 관람객에게 판매할 티켓, 티켓 판매 금액의 2가지를 속성으로 가져야 함
    - `getTicket()`의 경우에는, 책에서 java 예제코드는 `ArrayList`의 첫번째 요소를 리턴하는 식으로 구현했지만, swift의 경우에는 배열만 존재하기 때문에 `O(1)`로 처리하기 위해 배열의 `popLast()`를 호출하는 식으로 구현

```swift
struct TicketOffice {
    
  private var amount: Int
  private var tickets:[Ticket]
  
  mutating func getTicket() -> Ticket? {
      tickets.popLast()
  }
  
  mutating func minusAmount(_ amount: Int) {
      self.amount -= amount
  }
  
  mutating func plusAmount(_ amount: Int) {
      self.amount += amount
  }
}
```

- 판매원은 매표소에서 관람객에게 초대장을 티켓으로 교환하거나, 티켓을 현금을 받고 판매하는 2가지의 행위를 수행해야 함
- 또한 판매원은 자신이 속한 매표소(TicketOffice)에 대한 정보를 알고 있어야 함

```swift
struct TicketSeller {
    
  var ticketOffice: TicketOffice 
}
```

- 마지막으로 판매원이 티켓을 주고, 이를 받음 관람객이 입장할 수 있는 소극장 객체를 정의해야 함
- Theater 객체는 관람객이 입장할 수 있도록 하는 행위를 포함해야 함

```swift
struct Theater {
    
    private var ticketSeller: TicketSeller
    
    mutating func enter(audience: Audience) {
        //가능한 티켓을 한 장 pop
        guard let ticket = ticketSeller.ticketOffice.getTicket() else { return }
        var audience = audience
        
        //초대권이 없는 관람객의 경우에는 티켓을 판매
        if !audience.bag.hasInvitation {
            audience.bag.minusAmount(ticket.fee)
            ticketSeller.ticketOffice.plusAmount(ticket.fee)
        }
        
        //관람객에게 티켓 지급
        audience.bag.setTicket(ticket)
    }
}
```

# 2. 무엇이 문제인가

- 로버트 마틴(Robert C. Martin)에 의하면, 소프트웨어 모듈은 아래의 3가지 기능을 가져야 한다고 함
    - 실행 중에 제대로 동작할 수 있어야 함
    - 변경을 위해 존재해야 함. 즉, 생명주기 동안 간단한 작업만으로 변경이 가능해야 함
        
        (변경이 어려우면 안됨)
        
    - 특별한 훈련 없이도 여러 개발자가 쉽게 읽고 이해할 수 있어야 함
- 위의 3가지에 입각해서 1번에서 작성한 코드에 어떤 문제가 있는지 살펴보자

## 2-1. 예상을 빗나가는 코드

- Theater 구조체의 enter 메소드의 역할은 아래와 같이 정리할 수 있음
    - 소극장(Theater)은 관람객(Audience)의 가방(Bag)에서 초대장(Invitation)이 있는 지 확인
    - 가방에 초대장이 있으면 판매원(TicketSeller)은 매표소(TicketOffice)에 있는 티켓(Ticket)을 관람객의 가방으로 옮김(setTicket)
    - 가방안에 초대장이 없으면 관람객의 가방에서 티켓 금액(amount)만큼의 현금을 꺼내서(minusAmount), 매표소에 적립한 후(plusAmount), 매표소에 있는 티켓을 관람객의 가방으로 옮김

- 위의 관점에서 볼 때, 우선적으로 Audience 객체와 TicketSeller 객체는 Theater에 의해 통제를 받는 ‘수동적인 존재’라는 한계가 있음
    - enter을 호출할 때마다, Audience, TicketSeller 인스턴스에 의한 값 변화가 발생
- 이를 그대로 현실에 반영한다면 관람객이 직접 초대장이나 현금을 판매원에게 주지도 않았는데, 입장하자마자 판매원이 관람객의 동의 혹은 다른 행위 없이 바로 가방을 들여다보는 상황이 될 수 있음
- 현실에서 만일 소극장의 티켓 판매원이나 제 3자가 초대장을 확인하고자 관람객의 가방을 멋대로 들여다보는 것은 당연히 문제되는 상황임.

- 또한, enter 메소드 자체의 코드에도 문제가 있는데, 우선적으로 티켓 판매원이 매표소에서 티켓을 가져오거나, 돈을 받는 등의 행위 자체가 2개 이상의 호출 깊이를 가짐
- 이는 다시 말해, enter 메소드 자체의 흐름을 이해하기 위해서는, 단순히 Theater의 하위 속성인 TicketSeller 인스턴스 뿐만 아니라, 이것의 하위 속성들도 이해해야 함을 의미함
    - 예를 들어 매소에서 티켓을 가져오는 행위 하나를 이해하기 위해서, Theater, TicektSeller, TicketOffice 3개의 객체를 이해해야 함

```swift
guard let ticket = ticketSeller.ticketOffice.getTicket() else { return }
```

- 좀 더 복잡한 코드라면, 이런 식으로 알아야 할 세부사항이 많아질수록 코드를 읽고 이해해야 하는 동료 개발자들에게 큰 부담을 주는 상황이 됨

## 2-2. 변경에 취약한 코드

- 만일 관람객이 현금을 가방(Bag)이 아닌 다른 곳에 보관하거나, 현금 대신 다른 방식으로 결제해야 할 경우 앞에서 작성한 Audience 구조체의 변화는 필연적임
- 아래와 같은 상황에서 audience에 대한 변화가 발생하면 해당 코드를 포함하는 Theater 자체에 대한 변화도 같이 발생하게 됨. 다시 말해 객체 하나의 변경이 다른 객체들의 변경을 초래하는 것

```swift
//가능한 티켓을 한 장 pop
guard let ticket = ticketSeller.ticketOffice.getTicket() else { return }
var audience = audience

//초대권이 없는 관람객의 경우에는 티켓을 판매
if !audience.bag.hasInvitation {
    audience.bag.minusAmount(ticket.fee)
    ticketSeller.ticketOffice.plusAmount(ticket.fee)
}

//관람객에게 티켓 지급
audience.bag.setTicket(ticket)
```

- 책에서는 위와 같은 문제를 가리켜 `의존성(dependency)`와 관련된 문제라 언급함. 객체 간의 의존성은, 어느 객체 하나의 변경에 대해, 의존관계에 있는 다른 객체도 함께 변경될 수 있음을 암시함
- 변화를 최소화하고자 의존성을 없애는 것이 답은 아니지만, 가능하면 변화의 정도를 줄이기 위해 최소한의 의존성만 유지하고 불필요한 의존성은 제거하는 것이 좋음
- 여기서 객체 간의 의존성이 과한 경우를 가리켜 `결합도(coupling)`이 높다고 표현하는데, 결합도 또한 의존성과 관련있는 개념이기 때문에, 자연스럽게 변경과도 관련이 있음
- 다시 말해, 두 객체 간의 결합도가 높으면 그만큼 어느 하나의 변화로 인한 다른 하나의 변화가 발생할 확률이 매우 높으며, 이는 유지보수를 그만큼 어렵게 만드는 코드라는 것을 의미

# 3. 설계 개선하기

- 현재 Theater 객체 안에서, ticketSeller을 통해서 Audience 객체의 속성에 직접 접근하고 있음
- 위에서 언급했듯이 관람객이 가방을 가지고 있던, 어디서 돈을 꺼내던 이러한 세부적인 사실은 굳이 소극장의 판매원이 굳이 알 필요가 없음
- 중요한 것은, 소극장의 역할은 관람객이 소극장에 입장했다는 행위를 주재하기만 하면 되는 것
- 따라서 Audience 객체와 TicketSeller 객체를 각각 자율적인 존재로 만드는 식으로 개선할 수 있음

## 3-1. 캡슐화를 통한 객체의 자율성 높이기

- Audience와 TicketSeller이 각각 Bag과 TicketOffice를 자율적으로 처리하도록 코드를 수정해볼 수 있음
- 우선 기존의 Theater의 enter 메소드 안에서, TicketOffice 인스턴스에 접근하던 코드를 TicketSeller 내부로 숨겨야 함
    - 아래와 같이 sellTo(_:) 메소드를 정의해서, 기존의 Theater의 enter 메소드가 하던 행위를 옮김
    - ticketOffice 속성 자체는 외부에서 직접 접근하지 않기 때문에 private하게 수정 가능
    - 이런 식으로 굳이 드러낼 필요가 없는 속성을 물리적으로 외부로부터 감추는 `캡슐화` 를 통해, Theater과 TicketSeller 객체 간의 결합도를 낮추고, 좀 더 유연한 코드가 되도록 수정!

```swift
struct TicketSeller {

  private var ticketOffice: TicketOffice
  
  mutating func sellTo(_ audience: Audience) {
      //가능한 티켓을 한 장 pop
      guard let ticket = ticketOffice.getTicket() else { return }
      var audience = audience
      
      //초대권이 없는 관람객의 경우에는 티켓을 판매
      if !audience.bag.hasInvitation {
          audience.bag.minusAmount(ticket.fee)
          ticketOffice.plusAmount(ticket.fee)
      }
      
      //관람객에게 티켓 지급
      audience.bag.setTicket(ticket)
  }
}
```

- 기존의 enter 메소드는 아래와 같이 간단해지며, Theater 자체는 ticketSeller의 속성에 대해 더 이상 알 필요가 없어짐

```swift
mutating func enter(audience: Audience) {
    ticketSeller.sellTo(audience)
}
```

- 두번째로 TicketSeller은 여전히 audience 속성 내부의 bag 속성에 직접 접근하는데, 이 역시 Audience 내부로 bag을 감추는 캡슐화를 통해 개선 가능(= Audience 자체는 여전히 자율적이지 않은 상태)
- sellTo 메소드 내부의 audience.bag에 접근하는 부분을 아래와 같이 Audience 내부에 buy 메소드를 정의하고 이동시킴
    - bag 속성 역시 더 이상 외부에서 직접 접근하지 않기 때문에 private으로 처리 가능(캡슐화)

```swift
import Foundation

struct Audience {
  
  private var bag: Bag
    
  mutating func buy(_ ticket: Ticket) -> Int {
      
      //return 후 관람객에게 티켓 지급
      defer {
          bag.setTicket(ticket)
      }
      
      //초대권이 없을 경우 돈을 지불
      if !bag.hasInvitation {
          bag.minusAmount(ticket.fee)
          return ticket.fee
      }
      
      return 0
  }
}
```

- 기존의 sellTo는 아래와 같이 간소화

```swift
mutating func sellTo(_ audience: Audience) {
    //가능한 티켓을 한 장 pop
    guard let ticket = ticketOffice.getTicket() else { return }
    var audience = audience
    
    let fee = audience.buy(ticket)
    ticketOffice.plusAmount(fee)
}
```

- 결과적으로 여전히 동일한 기능을 수행하지만, 기존과 달리 Audience 혹은 TicketSeller 둘 중 하나가 변하더라도 Theater은 해당 객체의 내부 속성을 직접 바라보지 않기 때문에 변화로부터 자연스러워졌음
- 만일 돈의 소지 방식, 티켓 배부 방식 등을 변경하고 싶으면 각각 Audience와 TicketSeller의 내부만 바꿔주면 됨
- Theater은 오로지 TicketSeller의 sellTo 메소드의 호출만을 알면되고, TicketSeller 역시 Audience의 buy 메소드가 리턴하는 값만을 알면되며, 각각 해당 메소드들이 내부적으로 어떤 식으로 동작하는지는 신경쓸 필요 없음
- 보통 밀접하게 연관된 작업만 수행하고, 연관성 없는 작업은 다른 객체에게 위임하는 것을 가리켜 `응집도(cohesion)`가높다고 하는데, 위의 경우는 결과적으로 객체들 간의 자율성을 높임으로써 결합도를 낮추고 응집도는 높인 케이스임

## 3-2. 절차지향과 객체지향, 책임의 이동

- 맨 처음 1번에서 구현한 코드는 전형적인 `절차적 프로그래밍` 관점에서 작성된 것이라 할 수 있는데, Theater의 enter 메소드가 `프로세스(Process)`가 되고, Theater이 내부적으로 처리하던 Audience, TicketSeller, Bag, TicketOffice의 인스턴스가 모두 `데이터(Data)`에 대응되는 개념임
- 데이터에 해당되는 객체들은 모두 프로세스를 담당하는 객체에 의존적일 수 밖에 없는데, 이런 식의 절차적 프로그래밍 코드는 결국 변경 및 유지보수에 취약한 코드를 양산한다는 한계가 있음
- 이를 개선하고자 3-1에서는 각각의 객체가 자율적으로 존재하도록, 특정 데이터와 이를 사용하는 프로세스가 하나의 모듈 안에 위치하도록 캡슐화를 적용했는데, 이것이 `객체지향 프로그래밍`의 관점에서 작성된 코드임
- 제대로된 객체지향 설계는 변화의 정도를 줄이고 유지보수가 용이한 코드를 위해,  `캡슐화` 를 통해 의존성의 정도를 적절히 관리해서, 객체 간의 결합도를 낮추는 것이라 할 수 있음

- 3-1에서의 코드 개선은 `책임(responsibility)의 이동` 관점에서 봤을 때, 기존에는 Theater에 모든 책임이 몰려 있던 것을, 캡슐화를 거치면서 TicketSeller, Audience라는 모듈에 Theater이 과도하게 가지고 있던 책임을 적절히 분배시키는 과정으로도 볼 수 있음
- 결과적으로 설계 및 유지보수를 어렵게 만드는 핵심은 의존성에 있는데, 이를 위해 우선적으로 불필요한 의존성을 제거함으로써 객체 간의 결합도를 낮춰야 함
    - (계속 반복되지만) 위의 코드에서는 결합도를 낮추고자 각각의 객체가 외부에서 몰라도 되는 속성을 감추는 캡슐화를 적용함으로써, 자기 자신의 자율성을 높였음
- 이렇게 객체들의 자율성이 높아지면 결과적으로 응집도가 높아지고, `높은 응집도`와 `낮은 결합도` 가 훌륭한 객체지향 설계의 핵심임

## 3-3. 좀 더 개선해보기

- TicketSeller, Audience를 개선한 것과 비슷하게 Bag 객체 역시 자율적인 객체가 되도록 개선할 수 있음
- 아래와 같이 개선된 Audience의 buy 메소드 내부에서, Bag와의 결합도를 낮추기 위해 Bag의 속성에 직접 접근하는 로직을 제거해보는 것

```swift
struct Audience {
    
    private var bag: Bag
    
    mutating func buy(_ ticket: Ticket) -> Int {
        
        //return 후 관람객에게 티켓 지급
        defer {
            bag.setTicket(ticket)
        }
        
        //초대권이 없을 경우 돈을 지불
        if !bag.hasInvitation {
            bag.minusAmount(ticket.fee)
            return ticket.fee
        }
        
        return 0
    }
}
```

- buy 내부의 bag의 하위 속성 및 메소드에 접근하는 로직을 모두, Bag 내부의 별도 메소드로 이동시킨 후 속성 및 메소드를 모두 은닉

```swift
import Foundation

struct Bag {
    
    private var amount: Int
    private var invitation: Invitation?
    private var ticket: Ticket?
    
    init(amount: Int) {
        self.amount = amount
    }
    
    init(invitation: Invitation, amount: Int) {
        self.invitation = invitation
        self.amount = amount
    }
    
    private var hasInvitation: Bool {
        invitation == nil
    }
    
    var hasTicket: Bool {
        ticket == nil
    }
    
    mutating func hold(_ ticket: Ticket) -> Int {

        defer {
            setTicket(ticket)
        }
        
        if !hasInvitation {
            minusAmount(ticket.fee)
            return ticket.fee
        }
        
        return 0
    }
    
    mutating private func setTicket(_ ticket: Ticket) {
        self.ticket = ticket
    }
    
    mutating private func minusAmount(_ amount: Int) {
        self.amount -= amount
    }
    
    mutating private func plusAmount(_ amount: Int) {
        self.amount += amount
    }
}
```

- 위와 비슷하게 TicketOffice와 TickSeller 역시, TicketOffice의 메소드 혹은 속성을 은닉시켜서 의존성을 줄일 수 있음

```swift
struct TicketOffice {
    
    private var amount: Int
    private var tickets:[Ticket]
    
        //TicketOffice - Audience 간 의존성이 새로 생김
    mutating func sellTicketTo(_ audience: Audience) {
        guard let ticket = getTicket() else { return }
        var audience = audience
        
        let fee = audience.buy(ticket)
        plusAmount(fee)
    }
    
    mutating private func getTicket() -> Ticket? {
        tickets.popLast()
    }
    
    mutating private func minusAmount(_ amount: Int) {
        self.amount -= amount
    }
    
    mutating private func plusAmount(_ amount: Int) {
        self.amount += amount
    }
}
```

- 기존의 TicketSeller 내부의 sellTo 메소드 안에서, getTicket, minusAmount, plusAmount 메소드를 직접 호출하던 부분을 모두 TicketOffice의 sellTicketTo 내부로 이동시킨 후, sellTo는 아래와 같이 간소화

```swift
mutating func sellTo(_ audience: Audience) {
    ticketOffice.sellTicketTo(audience)
}
```

- 하지만, sellTicketTo 메소드를 구현하면서 TicketOffice가 해당 메소드의 파라미터로 Audience 인스턴스를 전달받기 때문에, 결과적으로 기존에는 없던 TicketOffice와 Audience간에 의존성이 생겨버렸음
- 의존성이 생겼다는 것은 그만큼 결합도가 높아졌다는 뜻인데, TicketSeller과 TicketOffice 간의 의존성을 낮추려다가 TicketOffice와 Audience 간 의존성이 생겨버렸기 때문에, 결과적으로 어떤 상황이 더 나을 지 선택해야 함(Traide-Off)

# 4. 객체지향 설계의 필요성

- 결과적으로 훌륭한 객체지향 설계는 곧 소프트웨어를 구성하는 모든 객체가 자율적으로 행동하는 설계를 의미함
- 좋은 설계를 위해서는, 우선 필요로 하는 기능을 구현할 수 있는 코드를 짜야함과 동시에, 이후에 변경하기 쉬운 코드가 되어야 함
- 변경하기 쉬워야 하는 이유는, 소프트웨어에 대한 요구사항은 항상 바뀌기 때문에 항상 변경을 유연하게 수용할 수 있어야 하기 때문
- 이러한 점에서 변경에 유연하게 대처하기 위한 대안이 바로 객체지향 프로그래밍인데, 앞서 언급한대로 의존성을 효율적으로 통제함으로써 요구사항에 더욱 수월하게 대처할 수 있는 길을 제시해줌
- 객체들은 서로 협력하고 메시지를 주고받기 때문에, 의존관계가 생기는 것은 필연적임. 하지만 지나친 의존성은 곧 변경에 취약한 코드가 됨을 의미하기 때문에, 항상 적절한 수준의 의존성을 유지할 수 있어야 함
