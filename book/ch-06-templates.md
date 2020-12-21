# 6 Կաղապարներ

Your quote here.
– B. Stroustrup

* Introduction
* Parameterized Types
  * Constrained Template Arguments; Value Template Arguments; Template Argument Deduction
* Parameterized Operations
  * Function Templates; Function Objects; Lambda Expressions
* Template Mechanisms
  * Variable Templates; Aliases; Compile-Time if
* Advice


## 6.1 Ներածություն

Միշտ չէ, որ `vector` օգտագործողն ուզում է օգտագործել `double` արժեքների վեկտոր։ Վեկտորն ընդհանուր հասկացություն է՝ անկախ սահող կետով թվի ներկայացումից։ Հետևաբար վեկտորի տարրի տիպը պետք է ներկայացվի վեկտորից անկախ։ _Կաղապարը_ (template) դաս կամ ֆունկցիա է, որը մենք պարամետրիզացնում ենք տիպերի կամ արժեքների հավաքածույով։ Կաղապարներն օգտագործում ենք այնպիսի գաղափարների ներկայացման համար, որոնք ավելի հստակ են ընկալվում որպես ինչ֊որ ընդհանուր բան, որից կարող ենք գեներացնել կոնկրետ տիպեր ու ֆունկցիաներ՝ նշելով արգումենտները, ինչպես վեկտորի տարրի `double` տիպը։ ??

## 6.2 Պարամետրիզացված տիպեր

«`double`֊երի վեկտոր» տիպը կարող ենք ընդհանրացնել որպես «ինչ֊որ բաների վեկտոր»՝ այն կաղապար դարձնելով և `double`֊ը տիպի պարամետրով փոխարինելով։ Օրինակ.

```C++
template<typename T>
class Vector {
private:
    T* elem;  // elem֊ը ցույց է տալիս T տիպի sz հատ տարրերի զանգվածի
    int sz;
public:
    explicit Vector(int s);          // կոնստրուկտոր: հաստատել ինվարիանտը, ձեռք բերել ռեսուրսները
    ~Vector() { delete[] elem; }     // դեստրուկտոր: ազատել ռեսուրսները

    // ... պատճենի ու տեղափոխման գործողություններ ...

    T& operator[](int i);               // ոչ-const Vector֊ների համար
    const T& operator[](int i) const;   // const Vector֊ների համար (§4.2.1)
    int size() const { return sz; }
};
```

`template<typename T>` նախդիրը `T`֊ն դարձնում է իրեն հաջորդող հայտարարության պարամետր։ Սա մաթեմատիկական _∀T_, կամ, ավելի ճիշտ, _∀T, որտեղ T-ն տիպ է_ գրառման C++֊ական համարժեքն է։ Եթե ուզում եք գրառել մաթեմատիկական _∀T այնպիսիք, որ P(T)_ պայմանը, ապա ձեզ պետ կգան կոնցեպտները (§6.2.1, §7.2)։ Տիպի պարամետրի ներկայացման համար `class` բառի օգտագործումը համարժեք է `typename`֊ի օգտագործմանը, և հին կոդերում հաճախ ենք տեսնում `template<class T>` նախդիրը։

Անդամ ֆունկցիաները կարող են սահմանվել նման եղանակով․

```C++
template<typename T>
Vector<T>::Vector(int s)
{
    if (s<0)
        throw Negative_size{};
    elem = new T[s];
    sz = s;
}

template<typename T>
const T& Vector<T>::operator[](int i) const
{
    if (i<0 || size()<=i)
        throw out_of_range{"Vector::operator[]"};
    return elem[i];
}
```

Այս սահմանումներն ունենալով կարող ենք սահմանել այսպիսի `Vector`֊ներ․

```C++
Vector<char> vc(200);      // 200 նիշերի վեկտոր
Vector<string> vs(17);     // 17 տողերի վեկտոր
Vector<list<int>> vli(45); // 45 int֊երի ցուցակների վեկտոր
```

`Vector<list<int>>` գրառման մեջ `>>` նիշերի զույգը ցույց է տալիս կաղապարի արգումենտների ցուցակի վերջը․ դա սխալ օգտագործված արտածման օպերատորը չէ։ Այլևս անհրաժեշտություն չկա, ինչպես C++98֊ում, երկու `>` նիշերի միջև բացատ թողնել։ 

`Vector`֊ները կարող ենք օգտագործել այսպես․

```C++
void write(const Vector<string>& vs) // մի քանի տողերի Vector
{
    for (int i = 0; i != vs.size(); ++i)
        cout << vs[i] << '\n';
}
```

Մեր `Vector`֊ի հետ միջակայքերով-for (range-for) կառուցվածքն աշխատեցնելու համար պետք է սահմանենք համապատասխան `begin()` և `end()` ֆունկցիաները․

```C++
template<typename T>
T∗ begin(Vector<T>& x)
{
    return x.size() ? &x[0] : nullptr; // ցուցիչ առաջին տարրին, կամ nullptr
}

template<typename T>
T∗ end(Vector<T>& x)
{
    return begin(x)+x.size(); // ցուցիչ վերջին հաջորդող տարրին
}
```

Սրանք ունենալով կարող ենք գրել․

```C++
void f2(Vector<string>& vs) // մի քանի տողերի Vector
{
    for (auto& s : vs)
        cout << s << '\n';
}
```

Նման եղանակով որպես կաղապարներ կարող ենք սահմանել ցուցակները, վեկտորները, արտապատկերումները (այսնքն՝ ասոցեատիվ զանգվածները), չկարգավորված արտապատկերումները (այսինքն՝ հեշավորվող աղյուսակները) և այլն (Chapter 11)։

Կաղապարները կոպիլյացիայի ժամանակի մեխանիզմ են, այսինքն դրանց օգտագործումը, համեմատած ձեռքով գրված կոդի հետ, կատարման ժամանակի որևէ լրացուցիչ ծախս չի պահանջում։ ?? Փաստորեն `Vector<double>`֊ի համար գեներացված կոդը նույնական է 4֊րդ գլխում բերված `Vector`֊ի կոդի հետ։ Ավելին, ստանդարտ գրադարանի `vector<double>`֊ի համար գեներացված կոդն ակնհայտորեն շատ ավելի լավն է լինելու (քանի որ դրա իրականացման վրա շատ ավեիլ ջանք է գործադրված)։

A template plus a set of template arguments is called an instantiation or a specialization. ?? Late in the compilation process, at instantiation time, code is generated for each instantiation used in a program (§7.5). The code generated is type checked so that the generated code is as type safe as handwritten code. Unfortunately, that type check often occurs late in the compilation process, at instantiation time.


### 6.2.1 Կաղապարների սահմանափակումներով արգումենտներ (C++20)

Հաճախ կաղապարն իմաստ է ստանում միայն այնպիսի արգումենտների համար, որոնք բավարարում են որոշակի պայմանների։ Օրինակ, `Vector`-ը սովորաբար ունենում է պատճենման գործողություն, և այդ դեպքում այն պետք է պահանջի, որ տարրերն էլ պատճենման գործողություն ունենան։ Այսինքն, պետք է պահանջենք, որ `Vector`֊ի կաղապարի արգումենտը լինի ոչ միայն `typename`, այլ `Element`, որտեղ «Element»֊ը սահմանում է այն պահանջները, որոնց պետք է բավարարի վեկտորի տարրը․

```C++
template<Element T>
class Vector {
private:
    T* elem;  // elem֊ը T տիպի sz տարրերի զանգվածի ցուցիչ է
    int sz;
    // ...
};
```
