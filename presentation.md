class: center, middle

# Call Me Maybe

## Frank West
---

# Command methods

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
## Someone wants to add a trial
--

### How would you solve it?

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
--
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

--

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
--

But now you have to remember to call the appropriate "charge" code everywhere
you touch the subscription code.

---
We start by creating a class that processes a csv

```ruby
describe CsvProcessor do

  it 'should process each line of the csv string' do

    csv_string = <<-CSV
frank,hanford,ca
joe,tulare,ca
jim,bakersfield,ca
CSV
    processor = spy('Processor')
    csv_processor = CsvProcessor.new(processor, csv_string)

    csv_processor.read

    expect(processor).to have_received(:process).exactly(3).times
    expect(processor).to have_received(:process).with(['frank','hanford','ca'])
    expect(processor).to have_received(:process).with(['joe','tulare','ca'])
    expect(processor).to have_received(:process).with(['jim','bakersfield','ca'])

  end
end
```
---

```ruby
class CsvProcessor
  def initialize(processor, csv_string)
    @processor = processor
    @csv_string = csv_string
  end

  def read
    CSV.parse(@csv_string) do |row|
      @processor.process row
    end
  end
end
```

