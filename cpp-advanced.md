# Lecture 1. rvalue-ссылки

+: позволяет уменьшать число копирований

А когда они вообще возникают?
1. При передаче параметров

    ```
        struct mytype {
            ...
        }
        void f(mytype); 
        mytype g();
        
        f(g());  // не будет вызван конструктор копирования mytype, т.к. g() - это rvalue (см. дальше)
    ```
    Во что вообще транслируется вызов при передачи параметров?
    ```
    void f(mytype r) {
        r.a = 42;
    }
    
    mytype y;
    f(y)
    ```
    
    *
        ```
        void f(mytype const& x) {
            mytype x_copy = x;
            x.a = 42;
        }
        
        mytype y;
        f(y);
        ```
        
    *   ```
        void f(mytype& x) {
            x.a = 42;
        }
        
        mytype y;
        {
            mytype y_copy = y;
            f(y_copy);
        }
        ```
        
    Заметим, что иногда мы можем не делать копирование: в 1, если не присваивается ничего; во 2, если передается rvalue (можем заисопльзовать только один раз):
    ```
    f(mytype(1, 2, 3));
    ```
    С++ компиляторы устроены вторым вариантом, и копируют при lvalue, не копируют при rvalue.
    
2. При возвращении значения
    **Return-value optimization (RVO)**
    ```
    mytype g() {
        return mytype(1, 2, 3);
    }
    
    mytype r = g();  // резервирует место под результат, а g() на этом месте коструирует объект 
    ```
    Поэтому транслируется в примерно такое:
    ```
    void g(void* result) {
        // а тут как?
        // (*): конструтор - фукция, принимающая void* this и набор аргументов
        //      деструктор - аналогично, функция, принимающая mytype* this
        // можно подумать тут такой код:
        char tmp[sizeof(mytype)];
        mytype_ctor(tmp, 1, 2, 3);
        mytype_ctor(result, tmp);
        mytype_dtor(tmp);
        // но тут копирование bruh, поэтому так по причине (*):
        mytype_ctor(result, 1, 2, 3);
    }
    
    char x[sizeof(mytype)];  // align кривой, но для примера похуй
    g(x);
    ```
    Раньше на RVO не было требований, с С++17 по стандарту нужно реализовывать.
    Как следствие, рекурсивные коллы не плодят множество копирований, т.к поинтер на result у них равный у всех.
    А если возвращаем локальную переменную?
    ```
    std::string foo() {
        std::string tmp;
        for (;;) {
            tmp += ...;
        }
        return tmp;  // все крупные компиляторы умеют не копировать тут, думая что tmp лежит в result
    }
    ```
    **Named return-value optimization (NRVO)**
    ```
    void foo(std::string* result) {
        std::string_ctor(result);
        for (;;) {
            *result += ...;
        }
    }
    ```
    Такое можно невсегда сделать, например, если в момент создания переменной, которую мы вернем существуют пути исполнения, в результате которых мы вернем не эту переменную, то :( :
    ```
    std::string bar() {
        std::string a("abc");
        std::string b("cde");
        
        if (flag) {
            return a;
        } else {
            return b;
        }
    }
    ```
    
Иногда копирование сохдает проблемы:
```
    vector<string> v;
    string s;
    v.push_back(s);
```
Тут для произвольного типа Т вектор будет вынужден делать копирования при переаллокации вектора на другуя память, а для некоторых можно mem_cpy.
Для некоторых типов это свойство очень пиздово, если например vector<fstream>, то как копировать?
В С++03 так нельзя было писать, люди хранили указатели. А это сложно писать exception-safity и не кэш-френдли. Или через vector<shared_ptr<fstream>>.
unique_ptr тоже не копируемый, т.к. в деструкторе у него delete. ПОэтмоу можем придумать следующую операцию:
```
unique_ptr(unique_ptr& other)  // move
    : ptr(other.ptr) {
    pther.ptr = nullptr;
}

void move(unique_ptr& oither) {
    delete ptr;
    ptr = other.ptr;
    other.ptr = nullptr;
}
```
Это позволило бы держать vector<fstream, даже для тех классов, где нельзя предоставить копирование.
но такая сигнатура мува не очень:
- rvalue не биндитвя в lvalue
-   ```
    vector<auto_ptr<mytype>> v;
    sort(v.begin(), v.end());
    ```
    Все указатели занулятся, т.к. там есть часть quick sort, который не копирует теперь, а мувает.
    
    
    
    
    
# Lecture 2. rvalue-ссылки
Непосредственно про ссылки
```
void push_back(T const&);
vois push_back(T&&); // push_back_move;
```
```
struct mytype {
    mytype(mytype const&);
    mytype(mytype&&); // кэйс в autoptr сюда не подходит, т.к. rvalue не биндит от lvalue
    
    mytype& operato=(mytype const&);
    mytype& operator=(mytype&&);
};
```

В целом это всё.
lvalue -> rvalue:
```
vector<mytype> v;
mytype obj;
// v.push_back(static_cast<mytype&&>(obj));
v.push_back(std::move(obj));
```
Чтобы не писать каст, в стандартной библиотеки есть мув:
```
template <typename T>
T&& std::move(T& obj) { // тут про бинд & -> && не говорили , оригинальный мув другой
    return static_cast<T&&>(obj);
}
```
Мув это не приказ, а совет, будет ли там отбирание владения - зависит от реализации.

Нюансы:
1.  ```
    struct person {
        person(person const& name) 
            : name(name) {}  
        
        person(person&& name)
            : name(name) {}  // тут name это lvalue, т.к. он именнованный
            : name(move(name)) {}  // тут все ок, как ожидается
    
        // Не хочется писать кучу перегрузок, альтернатива:
        person(string name) 
            : name(move(name)) {}
        // так лучше писать, но тут больше операций, т.к. 1 копирование + 1 мув

    };
    ```
    ```
    return move(local_var); // ломает NRVO, да и в целом хуйня которая не работает, очевидно
    ```
    Как проверить, выражение lvalue/rvalue - две перегрузки через явный тип:
    1.  Перегрузки
        ```
        void foo(mytype const&);  // 1
        void foo(mytype&&);  // 2
        
        mytype& lvalue();
        mytype&& rvalue();
        
        foo(lvalue());  // 1
        foo (rvalue());  // 2
        ```
    2.  Copy elision + RVO
        ```
        mytype test() {
            return lvalue();  // no RVO
            return rvalue();  // RVO
        }
        
        mytype x = lvalue();  // copy
        mytype x = rvalue();  // no copy
        ```
    
    ```
    mytype&& wtf();  // результат вызова это какая ссылка?
    foo(wtf());  // 2
    mytype test() {
        return wtf();  //  no RVO
    }
    mytype x = wtf();  // copy
    ```
    Ситуация коненчо мемная получилась. wtf := xvalue - функция которая возвращает &&
    lvalue := lvalue
    rvalue := prvalue
    
    xvalue:
        1. результат функции, возвращающий &&
        2. a.b где a это prvalue
    
    После мува правая часть остается в "неопределенном, но валидном для испольщования состоянии".
    Если бы мув был деструктивным, то есть удалял правую часть, то объект бы разрушался. Но сложнотсь в муве полей структур, т.к. какой-то один компонент тоьлко мувнулся, а развалилась вся структура.
    
    const на ретурн типе запрешает мувать, не хорошо так делать.
        
