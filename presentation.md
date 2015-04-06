class: center, middle

# Call Me Maybe

## Frank West
---

# Command methods

--

* Usually don't return values

--

* Cause side effects

--

* Requires spying to properly test

---
# An example

```ruby
class SubscriptionManager
  def initialize(processor, user)
    @processor = processor
    @user = user
  end

  def subscribe
    # Do some subscribing stuff here and then charge
    @processor.charge
  end
end

class PaymentProcessor
  def initialize(credit_card_details)
    @credit_card_details = credit_card_details
  end

  def charge
    # Charge someone some amount of money
  end
end

pp = PaymentProcessor.new(credit_card_details)
sm = SubscriptionManager.new(pp, current_user)
sm.subscribe

```
---

# And the tests

```ruby
describe SubscriptionManager do
  it 'should subscribe the user to our service and charge them' do
    pp = spy('PaymentProcessor')
    user = create(:user)
    sm = SubscriptionManager.new(pp, user)

    sm.subscribe

    # Expect some subscription stuff
    expect(pp).to have_received(:charge).once
  end
end

```
---
# And all's good, until...

--
## Someone wants to add a new feature
--

### How about a trial account

--

###  Given our existing code, how would you solve it?

```ruby
class SubscriptionManager
  def initialize(processor, user)
    @processor = processor
    @user = user
  end

  def subscribe
    # Do some subscribing stuff here and then charge
    @processor.charge
  end
end
```
---

## Maybe like this?

```ruby
class SubscriptionManager
  def initialize(processor, user, is_trial)
    @processor = processor
    @user = user
    @is_trial = is_trial
  end

  def subscribe
    # Do some subscribing stuff here and then charge
    return if @is_trial # Guard for trial
    @processor.charge
  end
end
```

Passing in booleans to make decisions later tends to lead to more booleans and
more decisions

```ruby
def initialize(processor, user, is_trial, is_employee, is_fellow_brony)
```

---

## or like this?

```ruby
class TrialProcessor
  def charge # Charge, really?
    # NOOP
  end
end

tp = TrialProcessor.new
sm = SubscriptionManager.new(tp, current_user)
sm.subscribe
```

Now we are just forcing classes to comply without a care in the world.

---
## or maybe even this?

```ruby
class SubscriptionManager
  def initialize(user)
    @user = user
  end

  def subscribe
    # Do some subscribing stuff here and then charge
  end
end

pp = PaymentProcessor.new(credit_card_details)
sm = SubscriptionManager.new(current_user)
if sm.subscribe
  pp.charge
end
```

But now you have to remember to call the appropriate "charge" code everywhere
you touch the subscription code.

---
# How would I solve it?

--

```ruby
class SubscriptionManager
  def initialize(processor, user)
    @processor = processor
    @user = user
  end

  def subscribe
    # Do some subscribing stuff here and then charge
    @processor.call
  end
end

class PaymentProcessor
  def initialize(credit_card_details)
    @credit_card_details = credit_card_details
  end

  def call
    # Charge someone some amount of money
  end
end

pp = PaymentProcessor.new(credit_card_details)
sm = SubscriptionManager.new(pp, current_user)
sm.subscribe
```
---

# Use .call in all of my command classes

* Call is a generic enough method name to be used in any class

--

* Now we can pass in anything that responds to call

--

### Like a new payment processor:

```ruby
class StripeProcessor
  def intialize(credit_card_details)
    @credit_card_details = credit_card_details
  end

  def call
    Stripe::Charge.create(
      :customer    => @credit_card_details.customer,
      :amount      => @credit_card_details.amount,
      :description => 'New Subscriber',
      :currency    => 'usd')
  end
end

pp = StripeProcessor.new(credit_card_details)
sm = SubscriptionManager.new(pp, current_user)
sm.subscribe
```

---

# or for a trial subscriber

```ruby
class TrialProcessor
  def call; end
end

tp = TrialProcessor.new
sm = SubscriptionManager.new(tp, current_user)
sm.subscribe
```
---
## But wait there's more...
--

### With stabby lambdas

```ruby
sm = SubscriptionManager.new(-> { puts "I'm a stabby lambda" }, current_user)
sm.subscribe
# => I'm a stabby lambda
```
--
### or with regular lambdas
```ruby
sm = SubscriptionManager.new(lambda { puts "I'm a lambda" }, current_user)
sm.subscribe
# => I'm a regular lambda
```
--

### or with procs
```ruby
sm = SubscriptionManager.new(proc { puts "I'm a proc" }, current_user)
sm.subscribe
# => I'm a proc
```
---
class: center, middle

# All of these respond to the call method

---
## and we can remove mocks from our tests

```ruby
describe SubscriptionManager do
  it 'should subscribe the user to our service and charge them' do
    value = 0
    pp = -> { value = 42 }
    user = create(:user)
    sm = SubscriptionManager.new(pp, user)

    sm.subscribe

    # Expect some subscription stuff
    expect(value).to eq(42)
  end
end
```
---

class: center, middle

# Thank you
