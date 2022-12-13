## Конспект по сетевому взаимодействию при помощи библиотеки Dlib
Подробно о устройстве можно прочитать в конспекте по сетевому взаимодействию 1.
Эта библиотека использует пространство имен dlib.
Сначала начнём работать с сервером.
Чтобы начать работать с библиотекой надо создать класс сервера который будет наследоваться от класса server_iostream.
В этом классе должен быть метод.
```C++
void on_connect(
            istream &in,
            ostream &out,
            const string &foreign_ip,
            const string & /*local_ip*/,
            unsigned short /*foreign_port*/,
            unsigned short /*local_port*/,
            uint64 connection_id
    ) {}
```
Этот метод будет вызваться при подключении клиента к серверу. У него есть параметры in и out из параметра in можно получить данные через знак >> , в параметр out можно записать отправляемые данные через знак <<.
пример:
```C++
string s;
in>>s;
out<<s;
out.flush();
```
Все данные записываемые в out временно не отправляются поэтому существует функция flush() который отправляет все данные которые содержатся в out-е.
Для того чтобы бы понять когда клиент отключился от сервера есть метод in.peek() и ключевое слово EOF.
in.peek() возвращает последний пришедший символ, а EOF это специальный символ который отправляется при прекращении передачи данных клиентом.
пример:
```C++
while (in.peek() != EOF) {
    string s;
    in>>s;
    out<<s;
    out.flush();
}
```
Для того чтобы мы могли запустить сервер надо задать порт на котором мы работаем.
```C++
serv our_server; //создаём класс сервера
our_server.set_listening_port(50005); //задаём порт на котором мы работаем их всего их 65536 при этом первые 2000 заняты.
our_server.start_async(); //запускаем сервер
```
вся программа:
```C++
#include <dlib/server.h>
#include <iostream>

using namespace dlib;
using namespace std;

class serv
{
    void on_connect(
            istream &in,
            ostream &out,
            const string &foreign_ip,
            const string & /*local_ip*/,
            unsigned short /*foreign_port*/,
            unsigned short /*local_port*/,
            uint64 connection_id
    ) {
        while (in.peek() != EOF) {
        string s;
        in>>s;
        out<<s<<' ';
        out.flush();
}
    }
};
int main()
{
    serv our_server; //создаём класс сервера
    our_server.set_listening_port(50005); //задаём сокет на котором мы работаем их всего их 65536 при этом первые  2000 заняты.
    our_server.start_async(); //запускаем сервер
}
```

#### Клиент
Для того чтобы подключится к серверу нам надо знать IP сервера и порт на котором мы работаем это делается одной строчкой:
iosockstream stream("localhost:50005");
localhost это наш компьютер.
Далее мы можем отправить сообщение серверу через << также получить сообщение через >>.
пример:
```C++
string s;
cin>>s;
stream<<s;
stream.flush();
stream>>s;
```
Здесь тоже все отправляемые данные отправляются не сразу чтобы решить это у нас есть stream.flush().
Ещё мы можем получить последний полученный символ через знакомый нам peek().
весь код клиента:
```C++
#include <dlib/server.h>
#include <iostream>

using namespace dlib;
using namespace std;

int main()
{
    iosockstream stream("localhost:50005");
    string s;
    cin>>s;
    stream<<s;
    stream.flush();
    stream>>s;
    cout<<s;
}
```