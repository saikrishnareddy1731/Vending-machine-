# Vending Machine System - Design Documentation

## Requirements

### Functional Requirements
1. **Product Selection**: Users can select products using code numbers (101-110)
2. **Payment Processing**: Accept multiple coin denominations (Penny, Nickel, Dime, Quarter)
3. **Change Management**: Return exact change when overpayment occurs
4. **Refund Capability**: Allow full refund of inserted coins before product selection
5. **Inventory Management**: Track product availability and mark items as sold out
6. **Product Dispensing**: Dispense selected product after successful payment
7. **State Transitions**: Machine transitions through appropriate states during transaction

### Non-Functional Requirements
1. **Maintainability**: Easy to add new states or modify existing behavior
2. **Extensibility**: Support adding new product types and coin denominations
3. **Reliability**: Handle insufficient funds and sold-out scenarios gracefully
4. **Simplicity**: Clear and understandable code structure

## Objectives

### Primary Objectives
1. **Implement State Pattern**: Manage vending machine behavior across different states
2. **Separation of Concerns**: Isolate inventory, payment, and dispensing logic
3. **Error Handling**: Gracefully handle exceptional scenarios (insufficient funds, invalid codes, sold out items)
4. **User Experience**: Provide clear feedback at each transaction step

### Design Objectives
1. Create a flexible architecture supporting future enhancements
2. Minimize coupling between components
3. Maximize code reusability
4. Follow SOLID principles

## Design Patterns Used

### 1. State Pattern (Implemented)
**Purpose**: Manage vending machine behavior that changes based on internal state

**Implementation**:
- **Context**: `VendingMachine` class maintains current state
- **State Interface**: Abstract `State` class defines all possible operations
- **Concrete States**: 
  - `IdleState`: Initial state, waiting for user to insert coins
  - `HasMoneyState`: Coins inserted, ready for product selection
  - `SelectionState`: Product being selected, validating payment
  - `DispenseState`: Dispensing product and returning change

**Benefits**:
- Eliminates complex conditional logic (no giant if-else chains)
- Each state encapsulates its own behavior
- Easy to add new states without modifying existing code
- State transitions are explicit and type-safe

**How it works in the code**:
```java
// VendingMachine acts as Context
State vendingState = vendingMachine.getVendingMachineState();
vendingState.clickOnInsertCoinButton(vendingMachine);
// State changes from IdleState to HasMoneyState
```

**State Transitions**:
```
IdleState → HasMoneyState (insert coins)
HasMoneyState → SelectionState (select product)
SelectionState → DispenseState (valid payment)
SelectionState → IdleState (insufficient payment/refund)
DispenseState → IdleState (product dispensed)
```

### 2. Strategy Pattern (Potential Enhancement)
**Where it could be applied**: Payment processing and change calculation

**Current Implementation Limitations**:
- Change calculation is hardcoded in `SelectionState.getChange()`
- Only supports coin-based payment
- Cannot easily add card payment, mobile payment, etc.
- Change return logic is simplified (just returns amount, not actual coins)

**How Strategy Pattern would improve the design**:
```java
interface PaymentStrategy {
    boolean processPayment(int amount);
    List<Coin> calculateChange(int paid, int price);
    void refund();
}

// Different strategies for different payment types
class CoinPaymentStrategy implements PaymentStrategy {
    private List<Coin> insertedCoins;
    
    public List<Coin> calculateChange(int paid, int price) {
        // Algorithm to return optimal coin combination
        // e.g., for 18 cents: [DIME, NICKEL, PENNY, PENNY, PENNY]
    }
}

class CardPaymentStrategy implements PaymentStrategy {
    private String cardNumber;
    // No change calculation needed for cards
}

class MobilePaymentStrategy implements PaymentStrategy {
    private String walletId;
    // Digital payment, no physical change
}
```

**Benefits**:
- Swap payment algorithms at runtime
- Add new payment methods without changing VendingMachine or State classes
- Each strategy is independently testable
- Change calculation can use different algorithms (greedy, dynamic programming, etc.)

**Usage Example**:
```java
class VendingMachine {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
}

// Client can choose strategy
vendingMachine.setPaymentStrategy(new CoinPaymentStrategy());
// or
vendingMachine.setPaymentStrategy(new CardPaymentStrategy());
```

## UML Class Diagram

```mermaid
classDiagram
    class VendingMachine {
        -State vendingMachineState
        -Inventory inventory
        -List~Coin~ coinList
        +VendingMachine()
        +getVendingMachineState() State
        +setVendingMachineState(State) void
        +getInventory() Inventory
        +setInventory(Inventory) void
        +getCoinList() List~Coin~
        +setCoinList(List~Coin~) void
    }

    class State {
        <<abstract>>
        +clickOnInsertCoinButton(VendingMachine) void
        +clickOnStartProductSelectionButton(VendingMachine) void
        +insertCoin(VendingMachine, Coin) void
        +chooseProduct(VendingMachine, int) void
        +getChange(int) int
        +dispenseProduct(VendingMachine, int) Item
        +refundFullMoney(VendingMachine) List~Coin~
        +updateInventory(VendingMachine, Item, int) void
    }

    class IdleState {
        +IdleState()
        +IdleState(VendingMachine)
        +clickOnInsertCoinButton(VendingMachine) void
        +updateInventory(VendingMachine, Item, int) void
    }

    class HasMoneyState {
        +HasMoneyState()
        +clickOnStartProductSelectionButton(VendingMachine) void
        +insertCoin(VendingMachine, Coin) void
        +refundFullMoney(VendingMachine) List~Coin~
    }

    class SelectionState {
        +SelectionState()
        +chooseProduct(VendingMachine, int) void
        +getChange(int) int
        +refundFullMoney(VendingMachine) List~Coin~
    }

    class DispenseState {
        +DispenseState(VendingMachine, int)
        +dispenseProduct(VendingMachine, int) Item
    }

    class Inventory {
        -ItemShelf[] inventory
        +Inventory(int)
        +getInventory() ItemShelf[]
        +setInventory(ItemShelf[]) void
        +initialEmptyInventory() void
        +addItem(Item, int) void
        +getItem(int) Item
        +updateSoldOutItem(int) void
    }

    class ItemShelf {
        -int code
        -Item item
        -boolean soldOut
        +getCode() int
        +setCode(int) void
        +getItem() Item
        +setItem(Item) void
        +isSoldOut() boolean
        +setSoldOut(boolean) void
    }

    class Item {
        -ItemType type
        -int price
        +getType() ItemType
        +setType(ItemType) void
        +getPrice() int
        +setPrice(int) void
    }

    class ItemType {
        <<enumeration>>
        COKE
        PEPSI
        JUICE
        SODA
    }

    class Coin {
        <<enumeration>>
        PENNY
        NICKEL
        DIME
        QUARTER
        +int value
    }

    State <|-- IdleState
    State <|-- HasMoneyState
    State <|-- SelectionState
    State <|-- DispenseState
    VendingMachine *-- State
    VendingMachine *-- Inventory
    VendingMachine o-- Coin
    Inventory *-- ItemShelf
    ItemShelf o-- Item
    Item --> ItemType
    IdleState ..> HasMoneyState
    HasMoneyState ..> SelectionState
    HasMoneyState ..> IdleState
    SelectionState ..> DispenseState
    SelectionState ..> IdleState
    DispenseState ..> IdleState
```

## State Transition Diagram

```mermaid
stateDiagram-v2
    [*] --> IdleState
    IdleState --> HasMoneyState: clickOnInsertCoinButton()
    HasMoneyState --> HasMoneyState: insertCoin()
    HasMoneyState --> SelectionState: clickOnStartProductSelectionButton()
    HasMoneyState --> IdleState: refundFullMoney()
    SelectionState --> DispenseState: chooseProduct() [sufficient payment]
    SelectionState --> IdleState: chooseProduct() [insufficient payment]
    DispenseState --> IdleState: dispenseProduct()
```

## Sequence Diagram - Successful Purchase Flow

```mermaid
sequenceDiagram
    participant User
    participant VM as VendingMachine
    participant IS as IdleState
    participant HMS as HasMoneyState
    participant SS as SelectionState
    participant DS as DispenseState
    participant Inv as Inventory

    User->>VM: clickOnInsertCoinButton()
    VM->>IS: clickOnInsertCoinButton()
    IS->>VM: setVendingMachineState(HasMoneyState)
    
    User->>VM: insertCoin(NICKEL)
    VM->>HMS: insertCoin(NICKEL)
    HMS->>VM: coinList.add(NICKEL)
    
    User->>VM: insertCoin(QUARTER)
    VM->>HMS: insertCoin(QUARTER)
    HMS->>VM: coinList.add(QUARTER)
    Note over VM: Total: 30 cents
    
    User->>VM: clickOnStartProductSelectionButton()
    VM->>HMS: clickOnStartProductSelectionButton()
    HMS->>VM: setVendingMachineState(SelectionState)
    
    User->>VM: chooseProduct(102)
    VM->>SS: chooseProduct(102)
    SS->>Inv: getItem(102)
    Inv-->>SS: Item(COKE, 12)
    SS->>SS: validate payment (30 >= 12)
    SS->>SS: calculate change (18)
    SS->>SS: getChange(18)
    SS->>VM: setVendingMachineState(DispenseState)
    
    VM->>DS: DispenseState(VM, 102)
    DS->>Inv: getItem(102)
    DS->>Inv: updateSoldOutItem(102)
    DS->>VM: setVendingMachineState(IdleState)
    Note over User: Product dispensed!
```

## Class Responsibilities

### VendingMachine (Context)
- **Purpose**: Central coordinator for the vending machine
- **Responsibilities**:
  - Maintain current state
  - Manage inventory
  - Track inserted coins
  - Delegate operations to current state

### State (Abstract)
- **Purpose**: Define interface for all possible operations
- **Responsibilities**:
  - Declare all state-specific operations
  - Provide default implementations (do nothing)
  - Allow subclasses to override only relevant methods

### IdleState
- **Purpose**: Initial waiting state
- **Responsibilities**:
  - Transition to HasMoneyState when coin button clicked
  - Allow inventory updates
  - Reset coin list when entering from other states
- **Valid Operations**: `clickOnInsertCoinButton()`, `updateInventory()`

### HasMoneyState
- **Purpose**: Accept coins and prepare for selection
- **Responsibilities**:
  - Accept additional coins
  - Transition to SelectionState
  - Handle refund requests
- **Valid Operations**: `insertCoin()`, `clickOnStartProductSelectionButton()`, `refundFullMoney()`

### SelectionState
- **Purpose**: Process product selection and validate payment
- **Responsibilities**:
  - Validate product code
  - Check item availability
  - Verify sufficient payment
  - Calculate and return change
  - Handle insufficient payment with refund
- **Valid Operations**: `chooseProduct()`, `getChange()`, `refundFullMoney()`

### DispenseState
- **Purpose**: Dispense product and complete transaction
- **Responsibilities**:
  - Retrieve selected item
  - Mark item as sold out
  - Return to IdleState
  - Auto-execute dispensing in constructor
- **Valid Operations**: `dispenseProduct()` (called automatically)

### Inventory
- **Purpose**: Manage product storage and availability
- **Responsibilities**:
  - Initialize 10 item shelves (codes 101-110)
  - Add items to shelves
  - Retrieve items by code
  - Update sold-out status

### ItemShelf
- **Purpose**: Represent single storage location
- **Responsibilities**:
  - Store unique code (101-110)
  - Hold item reference
  - Track availability status

### Item
- **Purpose**: Represent vendable product
- **Responsibilities**:
  - Store product type
  - Store product price

### ItemType (Enum)
- **Purpose**: Define available product types
- **Values**: COKE, PEPSI, JUICE, SODA

### Coin (Enum)
- **Purpose**: Define coin denominations
- **Values**: PENNY(1), NICKEL(5), DIME(10), QUARTER(25)

## Key Design Insights

### Strengths
1. **Clean State Management**: Each state handles only its valid operations
2. **No Conditional Complexity**: No large if-else chains for state-dependent behavior
3. **Automatic Transitions**: DispenseState automatically transitions to IdleState
4. **Fail-Safe Defaults**: Invalid operations are no-ops by default
5. **Clear Separation**: Inventory, payment, and state logic are decoupled

### Current Limitations
1. **Simplified change calculation**: Just returns amount, doesn't break into coins
2. **No concurrent transaction support**: Single-threaded design
3. **Limited error handling**: Generic exceptions without specific types
4. **No persistence**: All data lost when application ends
5. **Hard-coded inventory**: 10 items, codes 101-110

### Potential Enhancements with Strategy Pattern
1. **Payment Processing**: Different payment methods (coins, cards, mobile wallets)
2. **Change Calculation**: Optimal coin selection algorithms (greedy, dynamic programming)
3. **Pricing Strategies**: Dynamic pricing, discounts, promotions, time-based pricing
4. **Inventory Restocking**: Different restocking strategies (FIFO, LIFO, priority-based)

## Testing Scenarios

### Happy Path
1. Machine starts in IdleState
2. User clicks insert coin button → HasMoneyState
3. User inserts NICKEL (5) and QUARTER (25) → Total: 30 cents
4. User clicks product selection button → SelectionState
5. User selects code 102 (COKE, $12) → Sufficient payment
6. Machine calculates change: 30 - 12 = 18 cents
7. Machine transitions to DispenseState → Product dispensed
8. Machine returns to IdleState

### Error Cases
1. **Insufficient Payment**: User pays less than product price → Refund and IdleState
2. **Invalid Code**: User selects non-existent code → Exception thrown
3. **Sold Out**: User selects already sold item → Exception thrown
4. **Refund Request**: User requests refund in HasMoneyState → Return coins, IdleState

## Conclusion

This vending machine implementation effectively demonstrates the **State Pattern**, providing a clean and maintainable solution for managing state-dependent behavior. The architecture could be enhanced with the **Strategy Pattern** for flexible payment processing and change calculation algorithms, making it a solid foundation for a production system.
