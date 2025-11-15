classDiagram
    direction TB

    %% =================== MAIN CONTEXT ===================
    class VendingMachine {
        - State vendingMachineState
        - Inventory inventory
        - List~Coin~ coinList
        + VendingMachine()
        + getVendingMachineState() State
        + setVendingMachineState(State) void
        + getInventory() Inventory
        + setInventory(Inventory) void
        + getCoinList() List~Coin~
        + setCoinList(List~Coin~) void
    }

    %% =================== STATE PATTERN ===================
    class State {
        <<abstract>>
        + clickOnInsertCoinButton(VendingMachine) void
        + clickOnStartProductSelectionButton(VendingMachine) void
        + insertCoin(VendingMachine, Coin) void
        + chooseProduct(VendingMachine, int) void
        + getChange(int) int
        + dispenseProduct(VendingMachine, int) Item
        + refundFullMoney(VendingMachine) List~Coin~
        + updateInventory(VendingMachine, Item, int) void
    }

    class IdleState {
        + IdleState()
        + IdleState(VendingMachine)
        + clickOnInsertCoinButton(VendingMachine) void
        + updateInventory(VendingMachine, Item, int) void
    }

    class HasMoneyState {
        + HasMoneyState()
        + clickOnStartProductSelectionButton(VendingMachine) void
        + insertCoin(VendingMachine, Coin) void
        + refundFullMoney(VendingMachine) List~Coin~
    }

    class SelectionState {
        + SelectionState()
        + chooseProduct(VendingMachine, int) void
        + getChange(int) int
        + refundFullMoney(VendingMachine) List~Coin~
    }

    class DispenseState {
        + DispenseState(VendingMachine, int)
        + dispenseProduct(VendingMachine, int) Item
    }

    %% =================== INVENTORY ===================
    class Inventory {
        - ItemShelf[] inventory
        + Inventory(int)
        + getInventory() ItemShelf[]
        + setInventory(ItemShelf[]) void
        + initialEmptyInventory() void
        + addItem(Item, int) void
        + getItem(int) Item
        + updateSoldOutItem(int) void
    }

    class ItemShelf {
        - int code
        - Item item
        - boolean soldOut
        + getCode() int
        + setCode(int) void
        + getItem() Item
        + setItem(Item) void
        + isSoldOut() boolean
        + setSoldOut(boolean) void
    }

    class Item {
        - ItemType type
        - int price
        + getType() ItemType
        + setType(ItemType) void
        + getPrice() int
        + setPrice(int) void
    }

    %% =================== ENUMS ===================
    class ItemType {
        <<enumeration>>
        COKE
        PEPSI
        JUICE
        SODA
    }

    class Coin {
        <<enumeration>>
        PENNY : 1
        NICKEL : 5
        DIME : 10
        QUARTER : 25
        + int value
    }

    %% =================== RELATIONSHIPS ===================

    %% State inheritance
    State <|-- IdleState
    State <|-- HasMoneyState
    State <|-- SelectionState
    State <|-- DispenseState

    %% Composition / Aggregation
    VendingMachine *-- State : "current state"
    VendingMachine *-- Inventory : "contains"
    VendingMachine o-- Coin : "0..* coins"

    Inventory *-- ItemShelf : "10 slots"
    ItemShelf o-- Item : "0..1 item"
    Item --> ItemType

    %% State transitions
    IdleState ..> HasMoneyState : "clickOnInsertCoinButton()"
    HasMoneyState ..> SelectionState : "start product selection"
    HasMoneyState ..> IdleState : "refund money"
    SelectionState ..> DispenseState : "product ready"
    SelectionState ..> IdleState : "refund money"
    DispenseState ..> IdleState : "return to idle"

    %% Notes
    note for State "Defines behavior for each machine state.\nDefault methods do nothing."
    note for VendingMachine "Context class holding state,\ninventory and coin list."
    note for Inventory "Holds 10 ItemShelves\nwith codes 101–110."
