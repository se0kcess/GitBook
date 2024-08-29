---
description: js -> ts로 변환하기
coverY: 0
---

# 0829 타입스크립트

실습 : 환율 계산 자바스크립트 코드



자바스크립트 코드

```javascript
      const currencyEl_one = document.getElementById('currency-one');
      const amountEl_one = document.getElementById('amount-one');
      const currencyEl_two = document.getElementById('currency-two');
      const amountEl_two = document.getElementById('amount-two');
      const rateEl = document.getElementById('rate');
      const swap = document.getElementById('swap');

      const getList = async () => {
        const res = await fetch('https://open.exchangerate-api.com/v6/latest');
        const rates = await res.json();
        const list = rates.rates;
        return list;
      };

      optionList = async () => {
        const list = await getList();
        for (let key in list) {
          const option1 = document.createElement('option');
          const option2 = document.createElement('option');
          option1.value = key;
          option1.innerText = key;
          option1.setAttribute('data-rate', list[key]);
          option2.value = key;
          option2.innerText = key;
          option2.setAttribute('data-rate', list[key]);
          currencyEl_one.appendChild(option1);
          currencyEl_two.appendChild(option2);

          if (key === 'USD') {
            option1.selected = true;
          }
          if (key === 'KRW') {
            option2.selected = true;
          }
        }
        calculate();
      };

      optionList();

      const calculate = () => {
        const currency_one = currencyEl_one.value;
        const currency_two = currencyEl_two.value;
        const rate_one = parseFloat(currencyEl_one.options[currencyEl_one.selectedIndex].getAttribute('data-rate'));

        const rate_two = parseFloat(currencyEl_two.options[currencyEl_two.selectedIndex].getAttribute('data-rate'));

        const rate = rate_two / rate_one;
        rateEl.innerText = `1 ${currency_one} = ${rate.toFixed(4)} ${currency_two}`;

        amountEl_two.value = (amountEl_one.value * rate).toFixed(2);
      };

      currencyEl_one.addEventListener('change', calculate);
      amountEl_one.addEventListener('input', calculate);
      currencyEl_two.addEventListener('change', calculate);
      amountEl_two.addEventListener('input', calculate);

      swap.addEventListener('click', () => {
        const temp = currencyEl_one.value;
        currencyEl_one.value = currencyEl_two.value;
        currencyEl_two.value = temp;

        const tempAmount = amountEl_one.value;
        amountEl_one.value = amountEl_two.value;
        amountEl_two.value = tempAmount;

        calculate();
      });
```



타입스크립트 코드

```typescript
interface Currency {
  [key: string]: number;
}

interface Res {
  rates: Currency;
}

const currencyEl_one = document.getElementById('currency-one') as HTMLSelectElement;
const amountEl_one = document.getElementById('amount-one') as HTMLInputElement;
const currencyEl_two = document.getElementById('currency-two') as HTMLSelectElement;
const amountEl_two = document.getElementById('amount-two') as HTMLInputElement;
const rateEl = document.getElementById('rate') as HTMLElement;
const swap = document.getElementById('swap') as HTMLElement;

const getList = async (): Promise<Currency> => {
  const res = await fetch('https://open.exchangerate-api.com/v6/latest');
  const rates: Res = await res.json();
  const list: Currency = rates.rates;
  return list;
};

const optionList = async (): Promise<void> => {
  const list = await getList();
  for (let key in list) {
    const option1 = document.createElement('option');
    const option2 = document.createElement('option');
    option1.value = key;
    option1.innerText = key;
    option1.setAttribute('data-rate', list[key].toString());
    option2.value = key;
    option2.innerText = key;
    option2.setAttribute('data-rate', list[key].toString());
    currencyEl_one.appendChild(option1);
    currencyEl_two.appendChild(option2);

    if (key === 'USD') {
      option1.selected = true;
    }
    if (key === 'KRW') {
      option2.selected = true;
    }
  }
  calculate();
};

optionList();

const calculate = () => {
  const currency_one = currencyEl_one.value;
  const currency_two = currencyEl_two.value;
  const rate_one = parseFloat(currencyEl_one.options[currencyEl_one.selectedIndex].getAttribute('data-rate') || '0');

  const rate_two = parseFloat(currencyEl_two.options[currencyEl_two.selectedIndex].getAttribute('data-rate') || '0');

  const rate = rate_two / rate_one;
  rateEl.innerText = `1 ${currency_one} = ${rate.toFixed(4)} ${currency_two}`;

  amountEl_two.value = (parseFloat(amountEl_one.value) * rate).toFixed(2);
};

currencyEl_one.addEventListener('change', calculate);
amountEl_one.addEventListener('input', calculate);
currencyEl_two.addEventListener('change', calculate);
amountEl_two.addEventListener('input', calculate);

swap.addEventListener('click', () => {
  const temp = currencyEl_one.value;
  currencyEl_one.value = currencyEl_two.value;
  currencyEl_two.value = temp;

  const tempAmount = amountEl_one.value;
  amountEl_one.value = amountEl_two.value;
  amountEl_two.value = tempAmount;

  calculate();
});
```

#### 인덱스 시그니처&#x20;

동적으로 생성되는 객체의 키를 정의하는 방법

* 객체가 여러개의 Key를 가질 수 있다.
* Key와 매핑되는 value를 가지는 경우 사용한다.

{\[key : T] : U}&#x20;

```typescript
interface User{
  [key: string]: string | number // string 또는 number 타입 가능
  name: string // name은 only string
  age: number // age는 only number
}
const user: User = {
  name: 'seokcess',
  age: 85, // name이나 age 속성 제거시 Error 발생
  email: 'kimminsuk@gmail.com',
  zip: 12345,
  isValid: true // Error
}
```

#### 함수의 명시적  this 타입

```typescript
interface User{
  name: string;
}
function greet(this: User, msg: string){
  return `Hello ${this.name},${msg}`;
}
const seokcess = {
  name: 'seokcess'
  greet;
}
seokcess.greet('Good morning');

const neo = {
  name: 'Neo';
}
greet.call(neo, 'Have a great day!');  
```

#### 옵셔널 체이닝 "?."

* 프로퍼티가 없는 중첩 객체를 에러 없이 안전하게 접근 가능
* ?. 앞의 평가 대상이 undefined나 null 이면 평가를 멈추고 undefined 반환
* 존재하지 않아도 괜찮은 대상에 사용
* ?. 앞의 변수는 꼭 선언되어 있어야 함

#### 단락 평가

* ?.는 왼쪽 평가대상에 값이 없으면 즉시 평가를 멈춤

?.() : 존재 여부가 확실하지 않은 함수를 호출할 때 사용

?\[] : 객체 존재 여부가 확실하지 않은 경우에 프로퍼티를 안전하게 읽을 수 있음



#### nulish 병합 연산자 "??"

* 여러 피연산자 중 값이 확정되어 있는 변수를 찾을 수 있음 (단순한 문법으로)
* 변수에 기본 값 할당 기능

```javascript
x = (a !== null && a !== undefined ? a : b)
// x = a ?? b 와 동일하게 동작
a ?? b
// a가 null도 undefined도 아니면 a, 그 외엔 b
```

#### '??'과 '||'의 차이

* ||는 첫 번째 truthy 값을 반환
* ??는 첫 번째 정의된 값을 반환

?. (옵셔널 체이닝)과 ??(null 병합 연산자)의 차이점

*   ?.(옵셔널 체이닝)

    * 객체의 속성에 안전하게 접근할 때 사용
    * 작동 방식: ?. 앞의 값이 null 또는 undefined일 경우 나머지 부분을 평가하지 않고 undefined 반환
    * 객체의 중첩된 속성에 안전하게 접근할 때 사용


* ?? (null 병합 연산자)
  * 왼쪽 피연산자가 null 또는 undefined일 때 오른쪽 피연산자 반환
  * 작동 방식 : 왼쪽 피연산자가 null 또는 undefined가 아니면 왼쪽 피연산자를 반환하고 그렇지 않으면 오른쪽 피연산자 반환
  * 기본 값을 설정할 때 유용함

주요 차이점

* `?.`는 속성 접근에 사용되고, `??`는 값 자체에 대한 연산에 사용
* `?.`는 `undefined`를 반환하지만, `??`는 지정된 기본값을 반환함
* `?.`는 중간에 평가를 중단할 수 있지만, `??`는 항상 양쪽 피연산자를 평가함

```typescript
const user = { 
  name: "John",
  preferences: null 
};

const theme = user.preferences?.theme ?? "default theme"; 
// user.preferences == null / 왼쪽 값이 null이므로 오른쪽 값 반환
console.log(theme); // "defalt theme"
```



