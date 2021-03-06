---
layout: default
---
## Ducalis::BlackListSuffix

Please, avoid using of class suffixes like `Manager`, `Client` and so on. If it has no parts, change the name of the class to what each object is managing.

It's ok to use Manager as subclass of Person, which is there to refine a type of personal that has management behavior to it.
Related [article](<http://www.carlopescio.com/2011/04/your-coding-conventions-are-hurting-you.html>)

raises on classes with suffixes from black list

```ruby
# bad
class ListSorter
end
```

better to have names which map on business-logic

```ruby
# good
class SortedList
end
```

## Ducalis::CallbacksActiverecord

Please, avoid using of callbacks for models. It's better to keep models small ("dumb") and instead use "builder" classes/services: to construct new objects.
You can read more [here](https://medium.com/planet-arkency/a61fd75ab2d3).

raises on ActiveRecord classes which contains callbacks

```ruby
# bad
class Product < ActiveRecord::Base
  before_create :generate_code
end
```

better to use builder classes for complex workflows

```ruby
# good
class Product < ActiveRecord::Base
end

class ProductCreation
  def initialize(attributes)
    @attributes = attributes
  end

  def create
    Product.create(@attributes).tap do |product|
      generate_code(product)
    end
  end

  private

  def generate_code(product)
    # logic goes here
  end
end
```

## Ducalis::CaseMapping

Try to avoid `case when` statements. You can replace it with a sequence of `if... elsif... elsif... else`.
For cases where you need to choose from a large number of possibilities, you can create a dictionary mapping case values to functions to call by `call`. It's nice to have prefix for the method names, i.e.: `visit_`.
Usually `case when` statements are using for the next reasons:

I. Mapping between different values.
`("A" => 1, "B" => 2, ...)`

This case is all about data representing. If you do not need to execute any code it's better to use data structure which represents it. This way you are separating concepts: code returns corresponding value and you have config-like data structure which describes your data.

```ruby
  %w[A B ...].index("A") + 1
  # or
  { "A" => 1, "B" => 2 }.fetch("A")
```

II. Code execution depending of parameter or type:

  - a. `(:attack => attack, :defend => defend)`
  - b. `(Feet => value * 0.348, Meters => `value`)`

In this case code violates OOP and S[O]LID principle. Code shouldn't know about object type and classes should be open for extension, but closed for modification (but you can't do it with case-statements). This is a signal that you have some problems with architecture.

 a.

```ruby
attack: -> { execute_attack }, defend: -> { execute_defend }
# or
call(:"execute_#{action}")
```

b.

```ruby
class Meters; def to_metters; value;         end
class Feet;   def to_metters; value * 0.348; end
```

III. Code execution depending on some statement.

```ruby
(`a > 0` => 1, `a == 0` => 0, `a < 0` => -1)
```

This case is combination of I and II -- high code complexity and unit-tests complexity. There are variants how to solve it:

 a. Rewrite to simple if statement

```ruby
return 0 if a == 0
a > 0 ? 1 : -1
```

 b. Move statements to lambdas:

```ruby
 ->(a) { a > 0 }  =>  1,
 ->(a) { a == 0 } =>  0,
 ->(a) { a < 0 }  => -1
```

This way decreases code complexity by delegating it to lambdas and makes it easy to unit-testing because it's easy to test pure lambdas.

Such approach is named [table-driven design](<https://www.d.umn.edu/~gshute/softeng/table-driven.html>). Table-driven methods are schemes that allow you to look up information in a table rather than using logic statements (i.e. case, if). In simple cases, it's quicker and easier to use logic statements, but as the logic chain becomes more complex, table-driven code is simpler than complicated logic, easier to modify and more efficient.

raises on case statements

```ruby
# bad
case grade
when "A"
  puts "Well done!"
when "B"
  puts "Try harder!"
when "C"
  puts "You need help!!!"
else
  puts "You just making it up!"
end
```

better to use mapping

```ruby
# good
{
  "A" => "Well done!",
  "B" => "Try harder!",
  "C" => "You need help!!!",
}.fetch(grade) { "You just making it up!" }
```

## Ducalis::ComplexRegex

It seems like this regex is a little bit complex. It's better to increase code readability by using long form with "\x".

raises for regex with a lot of quantifiers

```ruby
# bad
PASSWORD_REGEX = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)./
AGE_RANGE_MATCH = /^(\d+)(?:-)(\d+)$/
FLOAT_NUMBER_REGEX = /(\d+,\d+.\d+|\d+[.,]\d+|\d+)/
```

better to use long form with comments

```ruby
# good
COMPLEX_REGEX = %r{
  start         # some text
  \s            # white space char
  (group)       # first group
  (?:alt1|alt2) # some alternation
  end
}x
LOG_FORMAT = %r{
  (\d{2}:\d{2}) # Time
  \s(\w+)       # Event type
  \s(.*)        # Message
}x
```

## Ducalis::ControllersExcept

Prefer to use `:only` over `:except` in controllers because it's more explicit and will be easier to maintain for new developers.

raises for `before_filters` with `except` method as array

```ruby
# bad
class ProductsController < ApplicationController
  before_filter :update_cost, except: [:index]

  def index; end
  def edit; end

  private

  def update_cost; end
end
```

better use `only` for `before_filters`

```ruby
# good
class ProductsController < ApplicationController
  before_filter :update_cost, only: [:edit]

  def index; end
  def edit; end

  private

  def update_cost; end
end
```

## Ducalis::DataAccessObjects

It's a good practice to move code related to serialization/deserialization out of the controller. Consider of creating Data Access Object to separate the data access parts from the application logic. It will eliminate problems related to refactoring and testing.

raises on working with `session` object

```ruby
# bad
class ProductsController < ApplicationController
  def edit
    session[:start_time] = Time.now
  end

  def update
    @time = Date.parse(session[:start_time]) - Time.now
  end
end
```

better to use DAO objects

```ruby
# good
class ProductsController < ApplicationController
  def edit
    session_time.start!
  end

  def update
    @time = session_time.period
  end

  private

  def session_time
    @_session_time ||= SessionTime.new(session)
  end
end

class SessionTime
  KEY = :start_time

  def initialize(session)
    @session = session
    @current_time = Time.now
  end

  def start!
    @session[KEY] = @current_time
  end

  def period
    Date.parse(@session[KEY]) - @current_time
  end
end
```

## Ducalis::DescriptiveBlockNames

Please, use descriptive names as block arguments. There is no any sense to save on letters.

raises for blocks with one/two chars names

```ruby
# bad
employees.map { |e| e.call(some, word) }
cards.each    { |c| c.date = dates[c.id] }
Tempfile.new("name.pdf").tap do |f|
  f.binmode
  f.write(code)
  f.close
end
```

better to use descriptive names

```ruby
# good
employees.map { |employee| employee.call(some, word) }
cards.each    { |card| card.date = dates[card.id] }
Tempfile.new("name.pdf").tap do |file|
  file.binmode
  file.write(code)
  file.close
end
```

## Ducalis::EnforceNamespace

Too improve code organization it is better to define namespaces to group services by high-level features, domains or any other dimension.

raises on classes without namespace

```ruby
# bad
class MyService; end
```

better to add a namespace for classes

```ruby
# good
module Namespace
  class MyService
  end
end
```

## Ducalis::EvlisOverusing

Seems like you are overusing safe navigation operator. Try to use right method (ex: `dig` for hashes), null object pattern or ensure types via explicit conversion (`to_a`, `to_s` and so on).
Related article: https://karolgalanciak.com/blog/2017/09/24/do-or-do-not-there-is-no-try-object-number-try-considered-harmful/

better to use NullObjects

```ruby
# good
class NullManufacturer
  def contact
    "No Manufacturer"
  end
end

def manufacturer
  product.manufacturer || NullManufacturer.new
end

manufacturer.contact
```

raises on multiple try callings

```ruby
# bad
product.try(:manufacturer).try(:contact)
```

## Ducalis::FacadePattern

There are too many instance variables for one controller action. It's beetter to refactor it with Facade pattern to simplify the controller.
Good article about [Facade](<https://medium.com/kkempin/facade-design-pattern-in-ruby-on-rails-710aa88326f>).

raises on working with `session` object

```ruby
# bad
class DashboardsController < ApplicationController
  def index
    @group = current_group
    @relationship_manager = @group.relationship_manager
    @contract_signer = @group.contract_signer

    @statistic = EnrollmentStatistic.for(@group)
    @tasks = serialize(@group.tasks, ' \
serializer: TaskSerializer)
    @external_links = @group.external_links
  end
end
```

better to use facade pattern

```ruby
# good
class Dashboard
  def initialize(group)
    @group
  end

  def external_links
    @group.external_links
  end

  def tasks
    serialize(@group.tasks, serializer: TaskSerializer)
  end

  def statistic
    EnrollmentStatistic.for(@group)
  end

  def contract_signer
    @group.contract_signer
  end

  def relationship_manager
    @group.relationship_manager
  end
end

class DashboardsController < ApplicationController
  def index
    @dashboard = Dashboard.new(current_group)
  end
end
```

## Ducalis::FetchExpression

You can use `fetch` instead:

```ruby
%<source>s
```

If your hash contains `nil` or `false` values and you want to treat them not like an actual values you should preliminarily remove this values from hash.
You can use `compact` (in case if you do not want to ignore `false` values) or `keep_if { |key, value| value }` (if you want to ignore all `false` and `nil` values).

raises on using [] with default

```ruby
# bad
params[:to] || destination
```

better to use fetch operator

```ruby
# good
params.fetch(:to) { destination }
```

## Ducalis::KeywordDefaults

Prefer to use keyword arguments for defaults. It increases readability and reduces ambiguities.
It is ok if an argument is single and the name obvious from the function declaration.

raises if method definition contains default values

```ruby
# bad
def calculate(step, index, dry = true); end
```

better to pass default values through keywords

```ruby
# good
def calculate(step, index, dry: true); end
```

## Ducalis::ModuleLikeClass

Seems like it will be better to define initialize and pass %<args>s there instead of each method.

raises for class without constructor but accepts the same args

```ruby
# bad
class TaskJournal
  def initialize(customer)
    # ...
  end

  def approve(task, estimate, options)
    # ...
  end

  def decline(user, task, estimate, details)
    # ...
  end

  private

  def log(record)
    # ...
  end
end
```

better to pass common arguments to the constructor

```ruby
# good
class TaskJournal
  def initialize(customer, task, estimate)
    # ...
  end

  def approve(options)
    # ...
  end

  def decline(user, details)
    # ...
  end

  private

  def log(record)
    # ...
  end
end
```

raises for class with only one public method with args

```ruby
# bad
class TaskJournal
  def approve(task)
    # ...
  end

  private

  def log(record)
    # ...
  end
end
```

## Ducalis::MultipleTimes

You should avoid multiple time-related calls to prevent bugs during the period junctions (like Time.now.day called twice in the same scope could return different values if you called it near 23:59:59). You can pass it as default keyword argument or assign to a local variable.
Compare:

```ruby
def period
  Date.today..(Date.today + 1.day)
end
# vs
def period(today: Date.today)
  today..(today + 1.day)
end
```

raises if method contains more then one Time.current calling

```ruby
# bad
def initialize(plan)
  @year = plan[:year] || Date.current.year
  @quarter = plan[:quarter] || quarter(Date.current)
end
```

better to inject time as parameter to the method or constructor

```ruby
# good
def initialize(plan, current_date: Date.current)
  @year = plan[:year] || current_date.year
  @quarter = plan[:quarter] || quarter(current_date)
end
```

## Ducalis::OnlyDefs

Prefer object instances to class methods because class methods resist refactoring. Begin with an object instance, even if it doesn’t have state or multiple methods right away. If you come back to change it later, you will be more likely to refactor. If it never changes, the difference between the class method approach and the instance is negligible, and you certainly won’t be any worse off.
Related article: https://codeclimate.com/blog/why-ruby-class-methods-resist-refactoring/

raises error for class with ONLY class methods

```ruby
# bad
class TaskJournal

  def self.call(task)
    # ...
  end

  def self.find(args)
    # ...
  end
end
```

better to use instance methods

```ruby
# good
class TaskJournal
  def call(task)
    # ...
  end

  def find(args)
    # ...
  end
end
```

## Ducalis::OptionsArgument

Default `options` (or `args`) argument isn't good idea. It's better to explicitly pass which keys are you interested in as keyword arguments. You can use split operator to support hash arguments.
Compare:

```ruby
def generate_1(document, options = {})
  format = options.delete(:format)
  limit = options.delete(:limit) || 20
  # ...
  [format, limit, options]
end

options = { format: 'csv', limit: 5, useless_arg: :value }
generate_1(1, options) #=> ["csv", 5, {:useless_arg=>:value}]
generate_1(1, format: 'csv', limit: 5, useless_arg: :value) #=> ["csv", 5, {:useless_arg=>:value}]

# vs

def generate_2(document, format:, limit: 20, **options)
  # ...
  [format, limit, options]
end

options = { format: 'csv', limit: 5, useless_arg: :value }
generate_2(1, **options) #=> ["csv", 5, {:useless_arg=>:value}]
generate_2(1, format: 'csv', limit: 5, useless_arg: :value) #=> ["csv", 5, {:useless_arg=>:value}]
```

raises if method accepts default options argument

```ruby
# bad
def generate(document, options = {})
  format = options.delete(:format)
  limit = options.delete(:limit) || 20
  [format, limit, options]
end
```

better to pass options with the split operator

```ruby
# good
def generate(document, format:, limit: 20, **options)
  [format, limit, options]
end
```

## Ducalis::ParamsPassing

It's better to pass already preprocessed params hash to services. Or you can use `arcane` gem.

raises if user pass `params` as argument from controller

```ruby
# bad
class ProductsController < ApplicationController
  def index
    Record.new(params).log
  end
end
```

better to pass permitted params

```ruby
# good
class ProductsController < ApplicationController
  def index
    Record.new(record_params).log
  end
end
```

## Ducalis::PossibleTap

Consider of using `.tap`, default ruby [method](<https://apidock.com/ruby/Object/tap>) which allows to replace intermediate variables with block, by this you are limiting scope pollution and make method scope more clear.
If it isn't possible, consider of moving it to method or even inline it.
[Related article](<http://seejohncode.com/2012/01/02/ruby-tap-that/>).

raises for methods with scope variable return

```ruby
# bad
def load_group
  group = channel.groups.find(params[:group_id])
  authorize group, :edit?
  group
end
```

better to use tap to increase code readability

```ruby
# good
def load_group
  channel.groups.find(params[:group_id]) do |group|
    authorize group, :edit?
  end
end
```

## Ducalis::PreferableMethods

Prefer to use %<alternative>s method instead of %<original>s because of %<reason>s.
Dangerous methods are:
`toggle!`, `save`, `delete`, `delete_all`, `update_attribute`, `update_column`, `update_columns`.

raises for `delete` method calling

```ruby
# bad
User.where(id: 7).delete
```

better to use callback-calling methods

```ruby
# good
User.where(id: 7).destroy
```

raises `update_column` method calling

```ruby
# bad
User.where(id: 7).update_column(admin: false)
```

## Ducalis::PrivateInstanceAssign

Don't use controller's filter methods for setting instance variables, use them only for changing application flow, such as redirecting if a user is not authenticated. Controller instance variables are forming contract between controller and view. Keeping instance variables defined in one place makes it easier to: reason, refactor and remove old views, test controllers and views, extract actions to new controllers, etc.
If you want to memoize variable, please, add underscore to the variable name start: `@_name`.

raises for instance variables in controllers private methods

```ruby
# bad
class EmployeesController < ApplicationController
  private

  def load_employee
    @employee = Employee.find(params[:id])
  end
end
```

better to implicitly assign variables in public methods

```ruby
# good
class EmployeesController < ApplicationController
  def index
    @employee = load_employee
  end

  private

  def load_employee
    Employee.find(params[:id])
  end
end
```

raises for memoization variables in controllers private methods

```ruby
# bad
class EmployeesController < ApplicationController
  private

  def catalog
    @catalog ||= Catalog.new
  end
end
```

better to mark private methods memo variables with "_"

```ruby
# good
class EmployeesController < ApplicationController
  private

  def catalog
    @_catalog ||= Catalog.new
  end
end
```

## Ducalis::ProtectedScopeCop

Seems like you are using `find` on non-protected scope. Potentially it could lead to unauthorized access. It's better to call `find` on authorized resources scopes.
Example:

```ruby
current_group.employees.find(params[:id])
# better then
Employee.find(params[:id])
```

raises if somewhere AR search was called on not protected scope

```ruby
# bad
Group.find(8)
```

better to search records on protected scopes

```ruby
# good
current_user.groups.find(8)
```

## Ducalis::PublicSend

You should avoid of using `send`-like method in production code. You can rewrite it as a hash with lambdas and fetch necessary actions or rewrite it as a module which you can include in code.

raises if send method used in code

```ruby
# bad
user.send(action)
```

better to use mappings for multiple actions

```ruby
# good
{
  bark: ->(animal) { animal.bark },
  meow: ->(animal) { animal.meow }
}.fetch(actions)
# or ever better
animal.voice
```

## Ducalis::RaiseWithoutErrorClass

It's better to add exception class as raise argument. It will make easier to catch and process it later.

raises when `raise` called without exception class

```ruby
# bad
raise "Something went wrong"
```

better to `raise` with exception class

```ruby
# good
raise StandardError, "Something went wrong"
```

## Ducalis::Recursion

It seems like you are using recursion in your code. In common, it is not a bad idea, but try to keep your business logic layer free from refursion code.

raises when method calls itself

```ruby
# bad
def set_rand_password
  password = SecureRandom.urlsafe_base64(PASSWORD_LENGTH)
  return set_rand_password unless password.match(PASSWORD_REGEX)
end
```

better to use lazy enumerations

```ruby
# good
def repeatedly
  Enumerator.new do |yielder|
    loop { yielder.yield(yield) }
  end
end

repeatedly { SecureRandom.urlsafe_base64(PASSWORD_LENGTH) }
     .find { |password| password.match(PASSWORD_REGEX) }
```

## Ducalis::RegexCop

It's better to move regex to constants with example instead of direct using it. It will allow you to reuse this regex and provide instructions for others.

Example:

```ruby
CONST_NAME = %<constant>s # "%<example>s"
%<fixed_string>s
```
Available regexes are:
      `/[[:alnum:]]/`, `/[[:alpha:]]/`, `/[[:blank:]]/`, `/[[:cntrl:]]/`, `/[[:digit:]]/`, `/[[:graph:]]/`, `/[[:lower:]]/`, `/[[:print:]]/`, `/[[:punct:]]/`, `/[[:space:]]/`, `/[[:upper:]]/`, `/[[:xdigit:]]/`, `/[[:word:]]/`, `/[[:ascii:]]/`

raises if somewhere used regex which is not moved to const

```ruby
# bad
name = "john"
puts "hi" if name =~ /john/
```

better to move regexes to constants with examples

```ruby
# good
FOUR_NUMBERS_REGEX = /\d{4}/ # 1234
puts "match" if number =~ FOUR_NUMBERS_REGEX
```

## Ducalis::RestOnlyCop

It's better for controllers to stay adherent to REST:
http://jeromedalbert.com/how-dhh-organizes-his-rails-controllers/.
[About RESTful architecture](<https://confreaks.tv/videos/railsconf2017-in-relentless-pursuit-of-rest>)

raises for controllers with non-REST methods

```ruby
# bad
class ProductsController < ApplicationController
  def index; end
  def order; end
end
```

better to use only REST methods and create new controllers

```ruby
# good
class ProductsController < ApplicationController
  def index; end
end

class OrdersController < ApplicationController
  def create; end
end
```

## Ducalis::RubocopDisable

Please, do not suppress RuboCop metrics, may be you can introduce some refactoring or another concept.

raises on RuboCop disable comments

```ruby
# bad
# rubocop:disable Metrics/ParameterLists
def calculate(five, args, at, one, list); end
```

better to follow RuboCop comments

```ruby
# good
def calculate(five, context); end
```

## Ducalis::StandardMethods

Please, be sure that you really want to redefine standard ruby methods.
You should know what are you doing and all consequences.

raises if use redefines default ruby methods

```ruby
# bad
def to_s
  "my version"
end
```

better to define non-default ruby methods

```ruby
# good
def present
  "my version"
end
```

## Ducalis::StringsInActiverecords

Please, do not use strings as arguments for %<method_name>s argument. It's hard to test, grep sources, code highlighting and so on. Consider using of symbols or lambdas for complex expressions.

raises for string if argument

```ruby
# bad
before_save :set_full_name, 
 if: 'name_changed? || postfix_name_changed?'
```

better to use lambda as argument

```ruby
# good
validates :file, if: -> { remote_url.blank? }
```

## Ducalis::TooLongWorkers

Seems like your worker is doing too much work, consider of moving business logic to service object. As rule, workers should have only two responsibilities:
- __Model materialization__: As async jobs working with serialized attributes it's nescessary to cast them into actual objects.
- __Errors handling__: Rescue errors and figure out what to do with them.

raises for a class with more than 5 lines

```ruby
# bad
class UserOnboardingWorker
  def perform(user_id, group_id)
    user = User.find_by(id: user_id)
    group = Group.find(id: group_id)

    return if user.nil? || group.nil?

    GroupOnboard.new(user).process
    OnboardingMailer.new(user).dliver_later
    GroupNotifications.new(group).onboard(user)
  end
end
```

better to use workers only as async primitive and use services

```ruby
# good
class UserOnboardingWorker
  def perform(user_id, group_id)
    user = User.find_by(id: user_id)
    group = Group.find(id: group_id)

    return if user.nil? || group.nil?

    OnboardingProcessing.new(user).call
  end
end
```

## Ducalis::UncommentedGem

Please, add comment why are you including non-realized gem version for %<gem>s.
It will increase [bus-factor](<https://en.wikipedia.org/wiki/Bus_factor>).

raises for gem from github without comment

```ruby
# bad
gem 'pry', '~> 0.10', '>= 0.10.0'
gem 'rake', '~> 12.1'
gem 'rspec', git: 'https://github.com/rspec/rspec'
```

better to add gems from github with explanatory comment

```ruby
# good
gem 'pry', '~> 0.10', '>= 0.10.0'
gem 'rake', '~> 12.1'
gem 'rspec', github: 'rspec/rspec' # new non released API
```

## Ducalis::UnlockedGem

It's better to lock gem versions explicitly with pessimistic operator (~>).

raises for gem without version

```ruby
# bad
gem 'pry'
```

better to lock gem versions

```ruby
# good
gem 'pry', '~> 0.10', '>= 0.10.0'
gem 'rake', '~> 12.1'
gem 'thor', '= 0.20.0'
gem 'rspec', github: 'rspec/rspec'
```

## Ducalis::UselessOnly

Seems like there is no any reason to keep before filter only for one action. Maybe it will be better to inline it?
Compare:

```ruby
before_filter :do_something, only: %i[index]
def index; end

# to

def index
  do_something
end

```

raises for `before_filters` with only one method

```ruby
# bad
class ProductsController < ApplicationController
  before_filter :update_cost, only: [:index]

  def index; end

  private

  def update_cost; end
end
```

better to inline calls for instead of moving it to only

```ruby
# good
class ProductsController < ApplicationController
  def index
    update_cost
  end

  private

  def update_cost; end
end
```
