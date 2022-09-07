#	Const, static, работа с файлами

* toc
{:toc}

##	Ключевое слово const

###		const переменные

При использовании с обычными переменными ключевое слово const определяет константы, которые нельзя переопределить. При попытке переопределения получаем ошибку компиляции.

```c++
int const j = 1;
int const k = 1;
k = 5; // ошибка компиляции 
j = 2; // ошибка компиляции 
```

###	const и указатели

Важное правило: const применяется к тому типу, который стоит слева от него.

```c++
const int* var1;
int const* var2;
int* const var3;
int const * const var4;
```

Таким образом, для переменной var1 константным является типа int, для var2 - то же самое. Таким образом, var1 и var2 разные способы записи одного и того же.

В третьем случае у нас константный указатель, то есть само значение мы можем менять, но при этом работаем всегда с одной и той же ячейкой памяти. 

В четвертом случае константным является и само значение, и его указатель, то есть имеем дело с полной константностью.

###	const методы

```c++
class Date
{
public:
   Date( int mn, int dy, int yr );
   int getMonth() const;     // A read-only function
   void setMonth( int mn );   // A write function; can't be const
private:
   int month;
};

int Date::getMonth() const
{
   return month;        // Doesn't modify anything
}
void Date::setMonth( int mn )
{
   month = mn;          // Modifies data member
}
```

Методы объявляются константными в случае, если они не должны менять никакие значения полей классов.

###	const параметры функций

```c++
void square(const int, const int);
 
int main()
{
    const int a = 4;
    int b = 5;
    square(a, b);   // 20
    return 0;
}
void square(const int a, const int b)
{
    //a = a * a;     так нельзя сделать
    //b = b * b;     так нельзя сделать
    std::cout << "In square: a * b = " << a * b << std::endl;
}
```

Параметры могут быть константными - значения таких параметров не могут меняться. 

##	Ключевое слово static

###	Статические глобальные переменные

```c++
static int global = 10;
void f() {
	++global;
}
```

Статическая глобальная переменная — это глобальная переменная, доступная только в пределах модуля (текущей единицы трансляции, header+cpp).

###	Статические локальные переменные 

```c++
int next (int start = 0) {
	static int k = start;
	return k++;
}
```

Статическая локальная переменная — это переменная, доступная только в пределах функции. 

Время жизни такой переменной — от первого вызова функции next до конца программы.

###	Статические функции

Должна быть ошибка?

```c++
1.cpp:
static void test () {
	cout << " A \n";
}

2.cpp:
static void test () {
	cout << " B \n";
}
```

Статические глобальные переменные и статические функции проходят *внутреннюю линковку*, поэтому они видны только в рамках одного модуля, и конфликта имен **не возникнет**.

###	Статические поля класса

Статические поля класса — это глобальные переменные, определённые внутри класса.

Для доступа к таким полям не требуется создание объекта. Значение такой переменной будет **общим** для **всех** объектов класса.

```c++
class A 
{
    private:
    static int mInstanceNumber = 0; // Переменная хранит в себе количество "живых" объектов
    public:
    A()
    {
        A::mInstanceNumber++;
    }
    ~A()
    {
        A::mInstanceNumber--;
    }
}
```

###	Статические методы класса

Статические методы — это функции, определённые внутри класса и имеющие доступ к закрытым полям и методам.

```c++
class A 
{
    private:
    static int mInstanceNumber = 0; // Переменная хранит в себе количество "живых" объектов
    public:
    static int getCurrentIntencesNumber()
    {
        return mInstanceNumber;
    }
    A()
    {
        A::mInstanceNumber++;
    }
    ~A()
    {
        A::mInstanceNumber--;
    }
}

...
    
std::cout << A::getCurrentInstancesNumber();
```

##	Работа с файлами

В процессе разработки, вам, скорее всего, понадобится умение читать и записывать информацию в файлы. Это может быть полезно при чтении конфигурации, работой с локалью, сериализация/десериализация объектов и т.д. 

В STL работа с файлами представлена с помощью класса `std::fstream`. [Дока](https://en.cppreference.com/w/cpp/io/basic_fstream)

С его помощью из файла можно читать подобно потоку ввода.

Список [режимов](https://en.cppreference.com/w/cpp/io/ios_base/openmode)

```c++
#include <iostream>
#include <fstream>
#include <string>
 
int main() {
  std::string filename = "test.bin";
  std::fstream file(filename, file.binary | file.trunc | file.in | file.out); // указывается имя файла и режим работы с ним
  if (!file.is_open()) {
    std::cout << "failed to open " << filename << '\n';
  } else {
    // write
    double d = 3.14;
    file.write(reinterpret_cast<char*>(&d), sizeof d); // binary output
    file << 123 << "abc"; // text output
 
    // сдвигаем указатель чтения и записи файла на 0 позицию
    file.seekp(0);
 
    // read
    file.read(reinterpret_cast<char*>(&d), sizeof d); // binary input
    int n;
    std::string str;
    if (file >> n >> str)// text input
      std::cout << "read back from file: " << d << ' ' << n << ' ' << str << '\n';
  }
}
```

Если не указывать binary режим, то файл откроется для чтения/записи как обычный набор строк и при чтении пробельные символы будут пропускаться.

