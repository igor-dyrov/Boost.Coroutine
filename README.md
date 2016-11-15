# Boost.Coroutine
С помощью [Boost.Coroutine](http://www.boost.org/libs/coroutine) стало возможным использовать сопрограммы в C++.
Сопрограммы - функция в других языках программирования, которые используют ключевое слово ***yield*** для сопрограмм.
В этих языках ***yield*** может использоваться как ***return***.
Когда используется ***yield***, функция запоминает положение, и если функция вызывается снова,
выполнение продолжается с того же места.

В C++ не определено ключевое слово ***yield***. C помощью Boost.Coroutine можно выйти из функции и
продолжить позже с того же места. Boost.Asio также использует выгоды
Boost.Coroutine.

Есть две версии Boost.Coroutine. Эта глава показывает вторую версию, которая является актуальной.
Эта версия была доступна начиная с Boost 1.55.0 и пришла на замену первой.

`Пример 51.1. Использование сопрограмм.`
<a name="example511"></a>
```c++
#include <boost/coroutine/all.hpp>
#include <iostream>

using namespace boost::coroutines;

void cooperative(coroutine<void>::push_type &sink)
{
  std::cout << "Hello";
  sink();
  std::cout << "world";
}

int main()
{
  coroutine<void>::pull_type source{cooperative};
  std::cout << ", ";
  source();
  std::cout << "!\n";
}
```

[Пример 51.1](#example511) определяет функцию, ***cooperative()***, которую вызывают из ***main()*** как сопрограмму. ***cooperative()***
возвращается к ***main()***, не отработав до конца и вызывается во второй раз.
На втором вызове функция продолжает работу с того места, где закончила в прошлый раз.

Чтобы использовать ***cooperative()*** как сопрограмму, надо использовать типы ***pull_type*** и ***push_type***.
Эти типы определены в ***boost::coroutines::coroutine***, которая является шаблоном,
инстанцированном с ***void*** в [Примере 51.1](example511).

Чтобы использовать сопрограммы, Вам нужны типы ***pull_type*** и ***push_type***. Один из этих типов будет использоваться, чтобы создать объект, который будет инициализирован с функцией, использующейся в качестве сопрограммы. Другой тип будет первым параметром функции сопрограммы.

[Пример 51.1](example511) создает объект, названный ***source***, типа ***pull_type*** в функции ***main()***. ***cooperative()*** передается в конструктор. ***push_type*** используется в качестве единственного параметра в ***cooperative()***.

Когда создается ***source***, функция ***cooperative()***, которая передан в конструктор, сразу вызывается как сопрограмма. Это происходит, потому что источник определен как ***pull_type***. Если бы источник был типа ***push_type***, конструктор не вызвал бы ***cooperative()*** как сопрограмму.

***cooperative()*** пишет ***Hello*** в стандартный поток вывода. После этого функция вызывает ***sink***, как функцию. Это возможно потому что ***push_type*** перегружает ***operator()***. В то время как ***source*** в ***main()*** представляет сопрограмму ***cooperative()***, ***sink*** представляет функцию ***main()*** в ***cooperative()***. Вызов ***sink*** заставляет ***cooperative()*** покинуть функцию, ***main()*** продолжается с того места, где был вызван ***cooperative()*** и пишет запятую в стандартный поток вывода.

Затем ***main()*** вызывает ***source***, в качестве функции. Снова, это возможно из-за перегруженного ***operator()***. На этот раз ***cooperative()*** продолжает с точки, где она закончила в прошлый раз и пишет ***World*** в стандартный вывод. Поскольку нет никакого другого кода в ***cooperative()***, сопрограмма завершается. Она возвращается в ***main()***, которая пишет восклицательный знак в стандартный вывод.

[Пример 51.1](example511) выводит ***Hello, world!*** на экран.

Можно представлять сопрограммы как совместные потоки. До некоторого момента ***main()*** и ***cooperative()*** выполняются одновременно. Код выполняется по очереди в ***main()*** и ***cooperative()***. Комманды в каждой функции выполняются последовательно. Благодаря сопрограммам, функциям не требуется завершаться, прежде чем другая может быть запущена.

`Пример 51.2. Возвращение значений из сопрограмм.`
<a name="example512"></a>
```c++
#include <boost/coroutine/all.hpp>
#include <functional>
#include <iostream>

using boost::coroutines::coroutine;

void cooperative(coroutine<int>::push_type &sink, int i)
{
  int j = i;
  sink(++j);
  sink(++j);
  std::cout << "end\n";
}

int main()
{
  using std::placeholders::_1;
  coroutine<int>::pull_type source{std::bind(cooperative, _1, 0)};
  std::cout << source.get() << '\n';
  source();
  std::cout << source.get() << '\n';
  source();
}
```

[Пример 51.2](example512) похож на предыдущий. На этот раз ***boost::coroutines::coroutine*** инициализирована типом ***int***. Это позволяет возвращать ***int*** из сопрограммы.

Направление, в котором ***int*** передается, зависит от где использованы ***pull_type*** и ***push_type***. Пример использует ***pull_type***, чтобы инициализировать объект в ***main()***. у ***cooperative()*** есть доступ к объекту типа ***push_type***. ***push_type*** отправляет значение, и ***pull_type*** принимает значение; таким образом определяется направление передачи данных.

***cooperative()*** вызывает ***sink*** с параметром типа ***int***. Этот параметр требуется, что бы сопрограмма инициализировалась с типом ***int***. Значение, переданное в ***sink***, получено от ***source*** в ***main()*** при помощи члена-функции, ***get()***, являющейся методом ***pull_type***.

[Пример 51.2](example512) также иллюстрирует, как функция с несколькими параметрами может использоваться в качестве сопрограммы. у ***cooperative()*** есть дополнительный параметр типа ***int***, который не может быть передан непосредственно конструктору ***pull_type***. Пример использует ***std::bind()***, чтобы соединить функцию с ***pull_type***.

Пример пишет ***1***, ***2*** и в конце ***end*** в стандартный поток вывода.

`Пример 51.3. Передача двух значений в сопрограмму.`
<a name="example513"></a>
```c++
#include <boost/coroutine/all.hpp>
#include <tuple>
#include <string>
#include <iostream>

using boost::coroutines::coroutine;

void cooperative(coroutine<std::tuple<int, std::string>>::pull_type &source)
{
  auto args = source.get();
  std::cout << std::get<0>(args) << " " << std::get<1>(args) << '\n';
  source();
  args = source.get();
  std::cout << std::get<0>(args) << " " << std::get<1>(args) << '\n';
}

int main()
{
  coroutine<std::tuple<int, std::string>>::push_type sink{cooperative};
  sink(std::make_tuple(0, "aaa"));
  sink(std::make_tuple(1, "bbb"));
  std::cout << "end\n";
}
```

[Пример 51.3](example513) использования ***push_type*** в ***main()*** и ***pull_type*** в ***cooperative()***, это означает, что данные передаются от вызывающей функции в сопрограмму.

Этот пример иллюстрирует, как могут быть переданы несколько значений. Boost.Coroutine не поддерживает передачу нескольких значений, таким образом, должен использоваться кортеж. Вы должны упаковать значения в кортеж или другую структуру.

[Пример 51.3](example513) выводит на экран ***0 aaa***, ***1 bbb***, и ***end***.

`Пример 51.4. Сопрограммы и исключения.`
<a name="example514"></a>
```c++
#include <boost/coroutine/all.hpp>
#include <stdexcept>
#include <iostream>

using boost::coroutines::coroutine;

void cooperative(coroutine<void>::push_type &sink)
{
  sink();
  throw std::runtime_error("error");
}

int main()
{
  coroutine<void>::pull_type source{cooperative};
  try
  {
    source();
  }
  catch (const std::runtime_error &e)
  {
    std::cerr << e.what() << '\n';
  }
}
```

Сопрограмма завершается сразу, как только выкидывается исключение. Исключение передается в вызывающую функцию, где может быть поймано. Таким образом исключения обрабатываются стандартно в сопрограммах, не отличаясь от вызова обычной функции.

[Пример 51.4](example514) показывает, как это работает. Программа пишет ***error*** в стандартный поток вывода.
