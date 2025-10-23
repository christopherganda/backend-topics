# SOLID Principles

[<- Back to Main Menu](../../README.md#1-core-skills)

## 1. What is SOLID?

SOLID is an acronym of five object-oriented principles. These principles help create software development more understandable, flexible and robust. Following them leads to cleaner, more maintainable code.
This principles were introduced by Robert C. Martin in 2000.

SOLID stands for:
1. Single Responsibility Principle (SRP)
2. Open/Closed Principle (OCP)
3. Liskov Substitution Principle (LSP)
4. Interface Segregation Principle (ISP)
5. Dependency Inversion Principle (DIP)

---

## 2. The five principles of SOLID
To understand SOLID much easier, [Gilded Rose Refactoring Kata](https://github.com/emilybache/GildedRose-Refactoring-Kata) is a good example to get more understanding.

### Single Responsibility Principle (SRP)
#### Definition
It states: `A class should have only one reason to change.`

What it actually means is that, a class should only change for the responsibility that it bears. 

#### Example Problem
Let's take a look at the Gilded Rose problem here.
```
class GildedRose

  def initialize(items)
    @items = items
  end

  def update_quality()
    @items.each do |item|
      if item.name != "Aged Brie" and item.name != "Backstage passes to a TAFKAL80ETC concert"
        if item.quality > 0
          if item.name != "Sulfuras, Hand of Ragnaros"
            item.quality = item.quality - 1
          end
        end
      else
        if item.quality < 50
          item.quality = item.quality + 1
          if item.name == "Backstage passes to a TAFKAL80ETC concert"
            if item.sell_in < 11
              if item.quality < 50
                item.quality = item.quality + 1
              end
            end
            if item.sell_in < 6
              if item.quality < 50
                item.quality = item.quality + 1
              end
            end
          end
        end
      end
      if item.name != "Sulfuras, Hand of Ragnaros"
        item.sell_in = item.sell_in - 1
      end
      if item.sell_in < 0
        if item.name != "Aged Brie"
          if item.name != "Backstage passes to a TAFKAL80ETC concert"
            if item.quality > 0
              if item.name != "Sulfuras, Hand of Ragnaros"
                item.quality = item.quality - 1
              end
            end
          else
            item.quality = item.quality - item.quality
          end
        else
          if item.quality < 50
            item.quality = item.quality + 1
          end
        end
      end
    end
  end
end
```
The problem here is that GildedRose takes all the responsibility for `Aged Brie`, `Sulfuras`, etc. So to achieve SRP, we should seperate each item type into separate class.

#### Example Solution
```
class GildedRose

  def initialize(items)
    @items = items
  end

  def update_quality()
    @items.each do |item|
      case item.name
      when "Aged Brie"
        AgedBrieItem.new(item).update_quality
      else
        SulfurasItem.new(item).update_quality
      end
    end
  end
end

class AgedBrieItem
  def initialize(item)
    @item = item
  end

  def update_quality
    # Do something
  end
end

class SulfurasItem
  def initialize(item)
    @item = item
  end

  def update_quality
    # Do Something
  end
end
```
---
### Open/Closed Principle (OCP)
#### Definition
It states: `Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.`

What it means is that if we want to add something(or extend), we won't need to modify the existing code. This is normally achieved through abstraction such as common `Item` class or using factory pattern.

#### Example Problem
On the SRP example solution above, imagine we want to add new item type called `Backstage passes`. We need to modify the GildedRose class. This means the code is not closed for modification.
```
class GildedRose

  def initialize(items)
    @items = items
  end

  def update_quality()
    @items.each do |item|
      case item.name
      when "Aged Brie"
        AgedBrieItem.new(item).update_quality
      when "Backstage passes to a TAFKAL80ETC concert"
        BackstagePassesItem.new(item).update_quality
      else
        SulfurasItem.new(item).update_quality
      end
    end
  end
end

class AgedBrieItem
  def initialize(item)
    @item = item
  end

  def update_quality
    # Do something
  end
end

class SulfurasItem
  def initialize(item)
    @item = item
  end

  def update_quality
    # Do Something
  end
end

class BackstagePassesItem
  def initialize(item)
    @item = item
  end

  def update_quality
    # Do Something
  end
end
```
#### Example Solution

```
class GildedRose

  def initialize(items)
    @wrapped_items = items.map { |i| ItemFactory.build(i) }
  end

  def update_quality()
    @wrapped_items.each(&:update_quality)
  end
end

class BaseItem
  attr_reader :item

  def initialize(item)
    @item = item
  end

  def update_quality
    raise NotImplementedError
  end
end

class AgedBrieItem < BaseItem
  def update_quality
    # Do something
  end
end

class SulfurasItem < BaseItem
end

class BackstagePassesItem < BaseItem
  def update_quality
    # Do Something
  end
end

class ItemFactory
  ITEM_PATTERNS = {
    /Aged Brie/i => AgedBrieItem,
    /Sulfuras/i => SulfurasItem,
    /Backstage Passes/i => BackstagePassesItem
  }.freeze

  def self.build(item)
    base_class = ITEM_PATTERNS.find { |pattern, _| item.name.match?(pattern) }&.last || GenericItem
    base_class.new(item)
  end
end
```
With this, you can extend new class and item patterns without modifying the existing code.
---

### Liskov Substitution Principle (LSP)
#### Definition
It states: `Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.`

To achieve LSP, we normally use inheritance/polymorphism. A good inheritance(Like `AgedBrie` or `Sulfuras` inheriting from `BaseItem`) must ensure that child classes can be use interchangably with parent class without breaking the system.

#### Example Problem
Looking back at OCP's example and Gilded Kata requirements where `Sulfuras` item type doesn't need to do `update_quality`. Our code already breaks LSP because `BaseItem.update_quality` raises `NotImplementedError`. Meanwhile we can leave `Sulfuras` with empty implementation.
```
class BaseItem
  attr_reader :item

  def initialize(item)
    @item = item
  end

  def update_quality
    raise NotImplementedError
  end
end

class SulfurasItem < BaseItem
end
```

#### Example Solution
There are multiple solutions to achieve LSP actually.

##### First Solution
```
class BaseItem
  attr_reader :item

  def initialize(item)
    @item = item
  end

  def update_quality
  end
end

class SulfurasItem < BaseItem
end
```
##### Second Solution
```
class BaseItem
  attr_reader :item

  def initialize(item)
    @item = item
  end

  def update_quality
    raise NotImplementedError
  end
end

class SulfurasItem < BaseItem
  def update_quality
    # This empty code override the parent's update quality method and we also achieve LSP
  end
end
```
---

### Interface Segregation Principle (ISP)
It states: `Clients should not be forced to depend on methods they do not use.`

This principle emphasize that large, general-purpose interface should be broken down into smaller and specific interfaces.

#### Example Problem
Since our example is in Ruby and Ruby doesn't have explicit interface. But since we already use simple `update_quality` through all class, this doesn't violate ISP. But let's take a look on another example using Java.

```
interface Notification {
  void sendEmail();
  void sendSMS();
  void sendPushNotification();
}

class EmailService implements Notification {
  @override
  public void sendEmail() {
    # Do something
  }

  @override
  public void sendSMS() {
    throw new UnsupportedOperationException();
  }

  @override
  public void sendPushNotification() {
    throw new UnsupportedOperationException();
  }
}
```
This `Notification` interface violates ISP as `EmailService` is forced to implements `sendSMS` and `sendPushNotification` which it doesn't need.

#### Example Solution
```
interface EmailNotification {
  void sendEmail();
}

interface SMSNotification {
  void sendSMS();
}

interface PushNotification {
  void sendPushNotification();
}

class EmailService implements EmailNotifications {
  @override
  public void sendEmail() {
    # Do something
  }
}

class SMSService implements SMSNotification {
  @override
  public void sendSMS() {
    # Do something
  }
}

class PushNotificationService implements PushNotification {
  @override
  public void sendPushNotification() {
    # Do something
  }
}
```
---

### Dependency Inversion Principle (DIP)
It states: `High-level modules should not depend on low-level modules. Both should depend on abstractions.`

What it means is that, when a class(High level) needs something from other class, the high level class should depend on the abstraction of the depended one.

#### Example Problem
Let's take a look at the example on SRP's solution. The `GildedRose` class depends directly on the low-level class(`AgedBrieItem`, `BackstagePassesItem`, `SulfurasItem`). This violates DIP.

```
class GildedRose

  def initialize(items)
    @items = items
  end

  def update_quality()
    @items.each do |item|
      case item.name
      when "Aged Brie"
        AgedBrieItem.new(item).update_quality
      when "Backstage passes to a TAFKAL80ETC concert"
        BackstagePassesItem.new(item).update_quality
      else
        SulfurasItem.new(item).update_quality
      end
    end
  end
end
```

#### Example Solution
Our solution on OCP that use `BaseItem` is the solution to achieve DIP. `BaseItem` acts as the abstaction and `ItemFactory` calls the abstraction class rather than the concrete class(`AgedBrieItem`, `BackstagePassesItem`, `SulfurasItem`).
