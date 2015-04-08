---

layout: ribbon

style: |

    #Cover h2 {
        margin:30px 0 0;
        color:#FFF;
        text-align:center;
        font-size:70px;
        }
    #Cover p {
        margin:10px 0 0;
        text-align:center;
        color:#FFF;
        font-style:italic;
        font-size:20px;
        }
        #Cover p a {
            color:#FFF;
            }
    #Picture h2 {
        color:#FFF;
        }
    #SeeMore h2 {
        font-size:100px
        }
    #SeeMore img {
        width:0.72em;
        height:0.72em;
        }
---

# Использование контрактов в Ruby on Rails приложениях (Design by Contract) {#Cover}

*Автор: [Давыденков Михаил](http://github.com/DavydenkovM/)*

![](pictures/contract.jpg)
<!-- photo by John Carey, fiftyfootshadows.net -->

## Основная идея, история и требования к языку:

1. Взаимодействие между объектами осуществляется через контракты, предусматривающие взаимные обязательства и преимущества (отношения клиент - поставщик).
2. Из языка Eiffel (Бертран Мейер). На уровне языка поддерживается в Clojure, Eiffell и других менее известных языках. В большинстве языков поддержка с помощью сторонних библиотек.
3. ЯП должен поддерживать наследование, динамическое связывание, способность обрабатывать исключения и возможноcть автоматического документирования ПО.

## Что такое контракт метода или функции:

1. Обязательства (предусловия), которые дают преимущество для поставщика (он может не проверять выполнение предусловий).
2. Свойства (постусловия).
3. Инварианты (валидации). Свойства/state объекта, который не должен меняться до и после вызова метода/функции.

## Контракты в Ruby

~~~
$ gem install contracts # inspired by contracts.coffee
require 'contracts' # в application.rb
include Contracts

Contract Num => Num
def double(x)
  x * 2
end
puts double("oops")
~~~

## Пример нарушения контракта

~~~
./contracts.rb:34:in 'failure_callback': Contract violation: (RuntimeError)
    Expected: Contracts::Num,
    Actual: "oops"
    Value guarded in: Object::double
    With Contract: Contracts::Num, Contracts::Num
    At: main.rb:6
    ...stack trace...
~~~

## Структура failure_message

~~~
{
  :arg => the argument to the method,
  :contract => the contract that got violated,
  :class => the method's class,
  :method => the method,
  :contracts => the contract object
}
~~~

## Встроенные контракты

contracts.ruby предусматривает большое количество встроенных контрактов:

~~~
Num, Pos, Neg, Nat, Bool, Any, None, 
Or, Xor, Not, 
ArrayOf, HashOf, Maybe, 
RespondTo[:password, :credit_card], Send[:valid?], Exactly[Numeric]
~~~

## Создание собственных контрактов

Контракты очень просто создать. Контрактом может быть:

1. Название класса (String или Fixnum)
2. Константа (nil или 1)
3. Proc, принимающий значение и возвращающий true/false
4. Класс, имеющий класс метод valid? 
5. Объект, имеющий метод valid?

## Кастомизации. Переписывание failure_callback

~~~
# initializer
Contract.override_failure_callback do |data|
  Rails.logger.error format(data)
  Airbrake.notify_or_ignore(error_from_data(data))
end
~~~

## Кастомизации. Переписывание сообщений об ошибках

~~~
def Num.to_s
  "a number please"
end
~~~

## Перегрузка методов и pattern matching

~~~
Contract Num => Num
def fact x
  if x == 1
    x
  else
    x * fact(x - 1)
  end
end
~~~


## Перегрузка методов и pattern matching 2

~~~
Contract 1 => 1
def fact x
  x
end

Contract Num => Num
def fact x
  x * fact(x - 1)
end
~~~

## Перегрузка методов и pattern matching 3

~~~
Contract lambda{|n| n < 12 } => Ticket
def get_ticket(age)
  ChildTicket.new(age: age)
end

Contract lambda{|n| n >= 12 } => Ticket
def get_ticket(age)
  AdultTicket.new(age: age)
end
~~~

## Контракты в модулях

~~~
module M
  include Contracts
  include Contracts::Modules

  Contract String => String
  def self.parse
    # do some hard parsing
  end
end
~~~

## Инварианты

~~~
include Contracts::Invariants

Invariant(:day) { 1 <= day && day <= 31 }
Invariant(:month) { 1 <= month && month <= 12 }

Contract None => Fixnum
def silly_next_day!
  self.day += 1
end
~~~

## Производительность

Заявлено, что контракты имееют минимальный slowdown по производительности.
Бенчмарки метода, возвращающего сумму двух чисел (1_000_000 раз):

~~~
                        total        real
testing read            2.530000 (  2.521314)
testing contracts read  2.900000 (  2.903721)
~~~

## ![](http://shwr.me/pictures/logo.svg) [Пример реализации контрактов в rails приложении](https://github.com/DavydenkovM/rails_and_contracts/)
{:.shout #SeeMore}

