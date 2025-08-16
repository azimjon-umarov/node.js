## Node.js Hodisalar Tsikli: To'liq Qo'llanma

Ushbu qo'llanma Node.js ning asosiy konsepsiyalaridan biri bo'lgan **Hodisalar Tsikli (Event Loop)** ni chuqur tushuntirib beradi. U o'rta darajadagi JavaScript dasturchilari uchun tushunarli bo'lishi bilan birga, tajribali Node.js mutaxassislari uchun ham qimmatli manba bo'lib xizmat qiladi.

-----

### 1\. Node.js Hodisalar Tsikliga Kirish  wprowadzenie

#### Hodisalar Tsikli nima?

Node.js bir vaqtning o'zida bitta operatsiyani bajaradigan, ya'ni **bitta oqimli (single-threaded)** muhitda ishlaydi. Shunga qaramay, u bir vaqtning o'zida minglab ulanishlarga xizmat ko'rsata oladi. Bu qanday amalga oshiriladi? Javob **Hodisalar Tsikli**da yashiringan.

**Hodisalar Tsikli** — bu Node.js ga bloklanmaydigan, asinxron I/O (kirish/chiqish) operatsiyalarini bajarish imkonini beruvchi mexanizmdir. Uning asosiy vazifasi — uzoq davom etadigan operatsiyalarni (masalan, faylni o'qish, ma'lumotlar bazasiga so'rov yuborish, tarmoq so'rovi) operatsion tizimga "yuklash" va ular tugashini kutib o'tirmasdan, boshqa kodlarni bajarishda davom etishdir. Operatsiya tugagach, uning natijasi bilan ishlash uchun mo'ljallangan qayta qo'ng'iroq (callback) funksiyasi maxsus navbatga qo'yiladi va Hodisalar Tsikli uni kerakli vaqtda ishga tushiradi.

#### Nima uchun bu muhim?

Hodisalar Tsikli Node.js-ni, ayniqsa, I/O operatsiyalari ko'p bo'lgan (I/O-bound) ilovalar uchun yuqori unumdorlikka va masshtablashuvchanlikka ega qiladi. Bunga misollar:

  * **Veb-serverlar:** Bir vaqtning o'zida ko'plab HTTP so'rovlariga xizmat ko'rsatish.
  * **API-lar:** Ma'lumotlar bazalari va boshqa xizmatlar bilan doimiy aloqada bo'lish.
  * **Real-time ilovalar:** Chatlar, o'yin serverlari va boshqalar.

An'anaviy bloklanadigan I/O modellarida har bir so'rov alohida oqimni (thread) egallaydi. Minglab foydalanuvchilar minglab oqimlarni talab qiladi, bu esa juda ko'p xotira va resurs sarfiga olib keladi. Node.js esa bitta oqim va Hodisalar Tsikli yordamida bu muammoni samarali hal qiladi.

#### Boshlovchilar uchun o'xshatish 🧑‍🍳

Hodisalar Tsiklini bir restorandagi **yolg'iz, ammo juda chaqqon ofitsiant**ga o'xshatish mumkin:

1.  **Buyurtma olish (So'rovni boshlash):** Ofitsiant (Node.js oqimi) stoldan buyurtma oladi (masalan, `fs.readFile` funksiyasini chaqiradi).
2.  **Buyurtmani oshxonaga berish (Operatsiyani topshirish):** U buyurtmani oshxonaga (operatsion tizim, `libuv`) beradi va oshpazlar (OS kerneli) taomni tayyorlashni boshlaydi.
3.  **Boshqa ishlarni bajarish (Bloklanmaslik):** Ofitsiant oshpazlarni kutib o'tirmaydi. U boshqa stollarga xizmat ko'rsatadi, yangi buyurtmalar oladi yoki hisob-kitob qiladi (boshqa JavaScript kodlarini bajaradi).
4.  **Taom tayyor bo'lishi (Operatsiya yakunlanishi):** Oshxona taom tayyor bo'lganligi haqida qo'ng'iroq chaladi (I/O operatsiyasi tugaydi va callback navbatga qo'yiladi).
5.  **Taomni yetkazib berish (Callback'ni bajarish):** Ofitsiant bo'shashi bilanoq (call stack bo'shaganda), qo'ng'iroqni eshitib, oshxonadan tayyor taomni oladi va uni tegishli stolga yetkazib beradi (Hodisalar Tsikli callback funksiyasini ishga tushiradi).

Bu ofitsiant hech qachon bir joyda qotib qolmaydi, doim harakatda bo'ladi, shuning uchun ham yolg'iz o'zi butun restoranga xizmat ko'rsata oladi.

-----

### 2\. Hodisalar Tsikli Qanday Ishlaydi: Umumiy Ko'rinish 🖼️

#### Katta Rasm

Asinxron operatsiyaning hayot tsikli quyidagicha kechadi:

1.  **Call Stack (Chaqiruvlar Steki):** Dasturingizdagi JavaScript funksiyalari shu yerda bajariladi. `console.log('Salom')` kabi sinxron kod darhol bajariladi.
2.  **Node.js API-lari:** `fs.readFile`, `http.get` kabi asinxron funksiya chaqirilganda, Node.js bu so'rovni o'zining C++ bilan yozilgan API-lariga yuboradi.
3.  **Libuv:** Bu Node.js "oshxonasi" — C tilida yozilgan, operatsion tizimning asinxron imkoniyatlariga kirishni ta'minlaydigan kutubxona. `libuv` I/O operatsiyasini boshqarishni o'z zimmasiga oladi va kerak bo'lsa, ishchi oqimlar (worker threads) pulidan foydalanadi.
4.  **Callback Queue (Qayta Qo'ng'iroqlar Navbati):** `libuv` operatsiyani tugatgach, unga bog'langan callback funksiyasini maxsus navbatga (masalan, Poll Queue) qo'yadi.
5.  **Event Loop (Hodisalar Tsikli):** Hodisalar Tsikli doimiy ravishda **Call Stack** bo'sh yoki bo'sh emasligini tekshiradi. Call Stack bo'shashi bilan, u Callback Queue'dan birinchi callback'ni olib, bajarish uchun Call Stack'ga joylashtiradi.

#### Sinxron va Asinxron Kodning Bajarilishi

  * **Sinxron kod:** Birma-bir, yuqoridan pastga qarab bajariladi. Bir operatsiya tugamaguncha, keyingisi boshlanmaydi. Bu Call Stack'ni "bloklashi" mumkin.
    ```javascript
    console.log('Birinchi');
    // Bu erda 1 soniya kutish bo'ladi
    const start = Date.now();
    while (Date.now() - start < 1000) {}
    console.log('Ikkinchi');
    ```
  * **Asinxron kod:** Operatsiyani boshlaydi va darhol keyingi qatorga o'tadi. Natija kelajakda callback orqali olinadi. Bu vaqtda Call Stack bo'sh bo'lib, boshqa ishlarni bajarishi mumkin.
    ```javascript
    const fs = require('fs');

    console.log('Birinchi');
    fs.readFile('/path/to/file', () => {
      console.log('Fayl o`qildi');
    });
    console.log('Ikkinchi');

    // Natija:
    // Birinchi
    // Ikkinchi
    // Fayl o`qildi
    ```

-----

### 3\. Hodisalar Tsiklining Bosqichlari: Chuqur Tahlil 🔄

Hodisalar Tsikli shunchaki bitta navbat emas. U bir necha bosqichlardan (phases) iborat bo'lib, har bir tsiklda ma'lum bir tartibda ulardan o'tadi. Har bir bosqich o'zining callback'lar navbatiga ega.

Asosiy bosqichlar:

1.  **timers**: `setTimeout()` va `setInterval()` tomonidan rejalashtirilgan callback'larni bajaradi.
2.  **pending callbacks**: Ba'zi tizim operatsiyalari (masalan, TCP xatoliklari) uchun kechiktirilgan callback'larni bajaradi.
3.  **idle, prepare**: Faqat Node.js ichki ishlari uchun ishlatiladi.
4.  **poll**: Yangi I/O hodisalarini qabul qiladi; I/O bilan bog'liq callback'larni (deyarli barchasini) bajaradi.
5.  **check**: `setImmediate()` callback'larini bajaradi.
6.  **close callbacks**: Yopilish hodisalari (`socket.on('close', ...)`) uchun callback'larni bajaradi.

#### 1\. Taymerlar Bosqichi (Timers Phase) ⏳

Bu bosqichda Hodisalar Tsikli `setTimeout()` va `setInterval()` orqali belgilangan va kutish vaqti o'tgan taymerlarning callback'larini ishga tushiradi.

Muhim jihati shundaki, taymerga berilgan vaqt (masalan, 100ms) **kafolatlangan aniq vaqt emas**, balki **minimal kutish vaqti**dir. Agar operatsion tizim boshqa vazifalar bilan band bo'lsa yoki oldingi bosqichda uzoq davom etadigan sinxron kod ishlayotgan bo'lsa, callback'ning bajarilishi kechikishi mumkin.

```javascript
// taymerlar.js
const start = Date.now();

setTimeout(() => {
  const end = Date.now();
  console.log(`setTimeout: ${end - start}ms keyin bajarildi`);
}, 500);

setInterval(() => {
  console.log('setInterval har 1000msda ishlaydi');
}, 1000);

console.log('Taymerlar rejalashtirildi');
```

#### 2\. Kutilayotgan Qayta Qo'ng'iroqlar Bosqichi (Pending Callbacks Phase) ⏸️

Bu bosqichda oldingi tsiklda bajarilishi kechiktirilgan I/O callback'lari chaqiriladi. Masalan, tizim `EAGAIN` kabi TCP xatoligini qaytarganda, Node.js bu callback'ni keyingi tsiklning shu bosqichida qayta urinish uchun navbatga qo'yadi. Bu kamdan-kam uchraydigan holat.

#### 3\. Kutish, Tayyorlash Bosqichi (Idle, Prepare Phase) ⚙️

Bu bosqichlar faqat Node.js'ning ichki mexanizmlari uchun ishlatiladi va odatda dasturchi kodi bilan to'g'ridan-to'g'ri bog'liq emas.

#### 4\. So'rovlar Bosqichi (Poll Phase) ❤️

Bu Hodisalar Tsiklining "yuragi" hisoblanadi. Bu yerda ikkita asosiy vazifa bajariladi:

1.  Taymerlar tomonidan belgilangan vaqt chegarasigacha bloklanib, yangi I/O hodisalarini kutish.
2.  Poll navbatidagi I/O bilan bog'liq callback'larni bajarish.

Poll bosqichi ishlashining ikki xil holati bor:

  * **Agar poll navbati bo'sh bo'lmasa:** Hodisalar Tsikli navbatdagi callback'larni sinxron ravishda, navbat bo'shaguncha yoki tizim chegarasiga yetguncha bajaradi.
  * **Agar poll navbati bo'sh bo'lsa:**
      * Agar `setImmediate()` orqali rejalashtirilgan skriptlar bo'lsa, Hodisalar Tsikli poll bosqichini tugatib, **check** bosqichiga o'tadi va ularni bajaradi.
      * Agar `setImmediate()` skriptlari bo'lmasa, Hodisalar Tsikli navbatga yangi callback'lar qo'shilishini kutadi. Agar bu vaqtda taymerlar muddati kelib qolsa, u keyingi tsiklning **timers** bosqichiga o'tadi.

<!-- end list -->

```javascript
// poll.js
const fs = require('fs');

fs.readFile(__filename, () => {
  // Bu callback poll navbatiga qo'shiladi
  console.log('Fayl o`qildi (poll bosqichi)');

  setTimeout(() => console.log('Ichki setTimeout (keyingi timers)'), 0);
  setImmediate(() => console.log('Ichki setImmediate (check bosqichi)'));
});

console.log('Fayl o`qish boshlandi');
```

#### 5\. Tekshirish Bosqichi (Check Phase) ✅

Bu bosqich faqat `setImmediate()` orqali rejalashtirilgan callback'larni bajarish uchun mo'ljallangan. Bu poll bosqichi tugagandan so'ng darhol ishga tushadi.

```javascript
// check.js
const fs = require('fs');

fs.readFile(__filename, () => {
  console.log('Fayl o`qish tugadi (poll)');

  // Poll bosqichidan keyin check bosqichi keladi, shuning uchun
  // setImmediate har doim birinchi bo'lib ishlaydi.
  setImmediate(() => {
    console.log('setImmediate callback');
  });

  setTimeout(() => {
    console.log('setTimeout callback');
  }, 0);
});
```

Ushbu kodda `setImmediate` har doim I/O callback ichidagi `setTimeout(..., 0)` dan oldin ishlaydi.

#### 6\. Yopish Qayta Qo'ng'iroqlari Bosqichi (Close Callbacks Phase) 🚪

Bu bosqichda yopilish hodisalari bilan bog'liq callback'lar bajariladi. Masalan, `socket.on('close', ...)` yoki `process.exit()` chaqirilganda.

```javascript
// close.js
const net = require('net');

const server = net.createServer((socket) => {
  // ...
  socket.on('close', () => {
    // Bu callback 'close callbacks' bosqichida bajariladi.
    console.log('Socket yopildi.');
  });
  socket.end();
}).listen(8080);

server.on('close', () => {
  console.log('Server yopildi.');
});

// Serverni bir muddatdan keyin yopamiz
setTimeout(() => server.close(), 2000);
```

-----

### 4\. Mikrotasklar va Makrotasklar 🔬

Hodisalar Tsiklining yuqorida sanab o'tilgan bosqichlari **Makrotasklar (Macrotasks)** deb ataladi. Ammo Node.js'da ulardan ham yuqoriroq ustuvorlikka ega bo'lgan **Mikrotasklar (Microtasks)** mavjud.

Mikrotasklar navbati Hodisalar Tsiklining bosqichlari orasida emas, balki **har bir makrotask bajarilgandan so'ng darhol** ishga tushadi. Ya'ni, timers bosqichidagi bitta callback tugashi bilan, Node.js keyingi callback'ga yoki keyingi bosqichga o'tishdan oldin **barcha mikrotasklar navbatini** to'liq bo'shatadi.

Ikki asosiy mikrotask navbati mavjud:

1.  **`process.nextTick()` navbati:** Eng yuqori ustuvorlikka ega. Bu navbatdagi barcha callback'lar bajarilmaguncha, boshqa hech narsa, hatto Promise mikrotasklari ham ishlamaydi.
2.  **Promise mikrotask navbati:** `Promise`'larning `.then()`, `.catch()`, `.finally()` metodlari va `async/await` sintaksisi bilan bog'liq callback'lar shu navbatga qo'yiladi.

**Bajarilish Tartibi:**

1.  Call Stack'dagi sinxron kod.
2.  `process.nextTick()` navbatidagi barcha callback'lar.
3.  Promise mikrotask navbatidagi barcha callback'lar.
4.  Hodisalar Tsiklining bitta makrotaski (masalan, bitta taymer callback'i).
5.  ... va yana 2-3-4-qadamlar takrorlanadi.

<!-- end list -->

```javascript
// microtasks.js
console.log('1: Boshlanish');

setTimeout(() => console.log('5: setTimeout (makrotask)'), 0);

setImmediate(() => console.log('6: setImmediate (makrotask)'));

Promise.resolve().then(() => console.log('4: Promise (mikrotask)'));

process.nextTick(() => console.log('3: nextTick (mikrotask)'));

console.log('2: Tugash');

// Natija:
// 1: Boshlanish
// 2: Tugash
// 3: nextTick (mikrotask)
// 4: Promise (mikrotask)
// 5: setTimeout (makrotask)
// 6: setImmediate (makrotask)
```

Ko'rib turganingizdek, sinxron kod tugagandan so'ng, Hodisalar Tsikli makrotasklarga (timers, check) o'tishdan oldin barcha mikrotasklarni (`nextTick` va `Promise`) bajardi.

-----

### 5\. Murakkab Holatlar va Keng Tarqalgan Xatolar 🤯

#### `setTimeout(fn, 0)` va `setImmediate(fn)`

Bu ikkalasi ham kodni "keyinroq" bajarishga o'xshasa-da, ularning ishlashida muhim farq bor.

  * **Asosiy modulda (I/O'dan tashqarida) chaqirilganda:**
    Ularning bajarilish tartibi **oldindan aytib bo'lmaydi (non-deterministic)**. Sababi, Hodisalar Tsikli boshlanishi uchun minimal vaqt kerak bo'ladi. Agar shu vaqt ichida `setTimeout(..., 0)` taymerining 1ms'lik minimal chegarasi o'tib ketsa, u birinchi ishlaydi. Aks holda, `setImmediate` birinchi bo'ladi.

    ```javascript
    // tartib_kafolatlanmagan.js
    setTimeout(() => console.log('setTimeout'), 0);
    setImmediate(() => console.log('setImmediate'));
    // Bu faylni bir necha marta ishga tushirsangiz, natija har xil bo'lishi mumkin.
    ```

  * **I/O callback ichida chaqirilganda:**
    Bajarilish tartibi **har doim bir xil va kafolatlangan (deterministic)**. `setImmediate` doim birinchi ishlaydi. Chunki I/O callback'lari **poll** bosqichida bajariladi. Poll bosqichidan keyingi bosqich esa **check** (`setImmediate` uchun). `setTimeout` esa keyingi tsiklning **timers** bosqichini kutishi kerak bo'ladi.

    ```javascript
    // tartib_kafolatlangan.js
    const fs = require('fs');

    fs.readFile(__filename, () => {
      setTimeout(() => console.log('setTimeout'), 0);
      setImmediate(() => console.log('setImmediate')); // Bu har doim birinchi ishlaydi
    });
    ```

#### Hodisalar Tsiklini Bloklash

Node.js bitta oqimli bo'lgani uchun, uzoq davom etadigan sinxron kod butun ilovani "muzlatib" qo'yadi. Bu **"Hodisalar Tsiklini bloklash"** deyiladi. Bloklangan vaqtda Node.js hech qanday I/O hodisasini yoki boshqa callback'ni bajara olmaydi.

```javascript
// bloklash.js
const http = require('http');

http.createServer((req, res) => {
  if (req.url === '/block') {
    console.log('Bloklovchi so`rov boshlandi...');
    // BU YOMON AMALIYOT! Hodisalar tsiklini bloklaydi.
    const end = Date.now() + 5000; // 5 soniyaga bloklash
    while (Date.now() < end) {}
    console.log('Bloklovchi so`rov tugadi.');
    res.end('Men bloklangan edim!');
  } else {
    res.end('Salom Dunyo!');
  }
}).listen(8000);
// /block'ga so'rov yuboring, shu paytda boshqa tabda serverga so'rov yuborsangiz,
// u 5 soniya javob bermay turadi.
```

**Yechimlar:**

  * **Worker Threads:** CPU-og'ir vazifalarni alohida oqimlarda bajarish uchun. Bu haqiqiy parallellikni ta'minlaydi.
  * **Vazifani bo'laklarga bo'lish:** Katta sinxron ishni `setTimeout(..., 0)` yoki `setImmediate` yordamida kichikroq qismlarga bo'lib, Hodisalar Tsikliga "nafas olish" imkonini berish.
  * Asinxron API'lardan foydalanish.

#### Callback Jahannami (Callback Hell) va Zamonaviy Yechimlar

Bir-biriga bog'liq ko'plab asinxron operatsiyalar "Callback Jahannami" deb ataluvchi murakkab va o'qish qiyin bo'lgan kodga olib kelishi mumkin.

```javascript
// Callback Hell
fs.readFile('file1.txt', 'utf8', (err, data1) => {
  if (err) throw err;
  fs.readFile('file2.txt', 'utf8', (err, data2) => {
    if (err) throw err;
    console.log(data1, data2);
  });
});
```

**Zamonaviy yechimlar:**

  * **Promises:** Asinxron operatsiyalarni zanjir qilib bog'lash imkonini beradi.
    ```javascript
    const fs = require('fs').promises;

    fs.readFile('file1.txt', 'utf8')
      .then(data1 => {
        return fs.readFile('file2.txt', 'utf8').then(data2 => [data1, data2]);
      })
      .then(([data1, data2]) => {
        console.log(data1, data2);
      })
      .catch(err => console.error(err));
    ```
  * **Async/Await:** Promise'lar ustiga qurilgan sintaktik qand (syntactic sugar) bo'lib, asinxron kodni sinxron kod kabi o'qiladigan qiladi.
    ```javascript
    const fs = require('fs').promises;

    async function readFiles() {
      try {
        const data1 = await fs.readFile('file1.txt', 'utf8');
        const data2 = await fs.readFile('file2.txt', 'utf8');
        console.log(data1, data2);
      } catch (err) {
        console.error(err);
      }
    }
    readFiles();
    ```

-----

### 6\. Asosiy Xulosalar 📌

  * **Node.js bitta oqimli:** Lekin Hodisalar Tsikli yordamida bloklanmaydigan asinxron operatsiyalarni bajaradi.
  * **`libuv` yordamchidir:** I/O operatsiyalarini operatsion tizimda bajarish uchun `libuv` kutubxonasidan foydalaniladi.
  * **Tsikl bosqichlardan iborat:** Hodisalar Tsikli `timers`, `poll`, `check` kabi aniq tartibdagi bosqichlardan o'tadi.
  * **Poll bosqichi markaziy rol o'ynaydi:** Aksariyat I/O callback'lari shu yerda ishga tushadi.
  * **Mikrotasklar ustuvorroq:** `process.nextTick()` va `Promise` callback'lari har bir makrotaskdan keyin va keyingi makrotaskdan oldin bajariladi.
  * **Tsiklni bloklamang:** CPU-og'ir sinxron koddan qoching, aks holda ilovangiz qotib qoladi. Worker Threads'dan foydalaning.
  * **Zamonaviy sintaksisdan foydalaning:** Callback Jahannami'dan qochish uchun `Promises` va `async/await`'ni qo'llang.

## Tricky interveiw questions

### 1-savol: Taymerlarning noaniq tartibi

**Kod parchasi:**

```javascript
setTimeout(() => {
  console.log('setTimeout');
}, 0);

setImmediate(() => {
  console.log('setImmediate');
});

console.log('script start');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
script start
// Keyingi ikki qatorning tartibi kafolatlanmagan:
setTimeout
setImmediate
// YOKI
setImmediate
setTimeout
```

**Chuqur tahlil:**

1.  **Call Stack va sinxron ijro:** Skript ishga tushganda, `setTimeout(..., 0)` taymerlar bosqichi uchun, `setImmediate(...)` esa tekshirish (check) bosqichi uchun rejalashtiriladi. So'ng `console.log('script start')` sinxron ravishda bajariladi va Call Stack bo'shaydi.
2.  **Hodisalar Tsiklining boshlanishi:** Hodisalar Tsikli ishga tushadi. U birinchi bo'lib **Taymerlar** bosqichiga kiradi.
3.  **Noaniqlik sababi:** `setTimeout(..., 0)` ning ishlashi uchun Node.js minimal 1ms kutishi kerak bo'lishi mumkin (bu OS scheduler'iga bog'liq). Agar Hodisalar Tsikli boshlanguncha bu 1ms o'tib ulgurgan bo'lsa, `setTimeout` callback'i birinchi bajariladi. Agar tsikl 1ms dan tezroq boshlansa, Taymerlar bosqichi bo'sh o'tadi, tsikl **Poll** va keyin **Check** bosqichiga o'tadi va `setImmediate` birinchi ishlaydi. Shuning uchun asosiy modulda bu ikkalasining tartibi oldindan aytib bo'lmaydi.

-----

### 2-savol: I/O ichidagi aniq tartib

**Kod parchasi:**

```javascript
const fs = require('fs');

fs.readFile(__filename, () => {
  console.log('I/O callback');

  setTimeout(() => {
    console.log('setTimeout');
  }, 0);

  setImmediate(() => {
    console.log('setImmediate');
  });
});
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
I/O callback
setImmediate
setTimeout
```

**Chuqur tahlil:**

1.  **I/O operatsiyasi:** `fs.readFile` chaqiruvi `libuv`'ga I/O operatsiyasini topshiradi. Uning callback'i fayl o'qib bo'lingach **Poll** bosqichining navbatiga qo'yiladi.
2.  **Poll bosqichi:** Hodisalar Tsikli **Poll** bosqichiga yetib kelganda, I/O callback'ini topadi va uni bajaradi. `console.log('I/O callback')` chiqadi.
3.  **Yangi Macrotask'larni rejalashtirish:** I/O callback'i ichida `setTimeout(..., 0)` keyingi tsiklning **Taymerlar** bosqichi uchun, `setImmediate` esa joriy tsiklning **Check** bosqichi uchun rejalashtiriladi.
4.  **Aniq tartib:** **Poll** bosqichi tugagandan so'ng, Hodisalar Tsikli to'g'ridan-to'g'ri keyingi bosqichga, ya'ni **Check** bosqichiga o'tadi. Shu sababli, `setImmediate` callback'i darhol bajariladi. `setTimeout` esa o'zining **Taymerlar** bosqichini keyingi tsikl aylanishida kutishi kerak. Shuning uchun I/O ichida bu tartib doim aniq.

-----

### 3-savol: `process.nextTick` va `Promise` ustuvorligi

**Kod parchasi:**

```javascript
Promise.resolve().then(() => {
  console.log('promise 1');
});

process.nextTick(() => {
  console.log('nextTick 1');
});

Promise.resolve().then(() => {
  console.log('promise 2');
});

process.nextTick(() => {
  console.log('nextTick 2');
});

console.log('script start');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
script start
nextTick 1
nextTick 2
promise 1
promise 2
```

**Chuqur tahlil:**

1.  **Sinxron ijro:** `console.log('script start')` darhol bajariladi.
2.  **Mikrotask'larni navbatga qo'yish:** Barcha `Promise.resolve().then()` chaqiruvlari **Promise mikrotask navbatiga**, `process.nextTick()` chaqiruvlari esa **`nextTick` mikrotask navbatiga** qo'shiladi.
3.  **Mikrotask'larni bajarish:** Sinxron kod tugab, Call Stack bo'shagach, Hodisalar Tsikli makrotasklarga (masalan, Taymerlar) o'tishdan oldin mikrotask navbatlarini bo'shatadi.
4.  **Ustuvorlik:** Node.js'da **`nextTick` navbati Promise navbatidan yuqoriroq ustuvorlikka ega**. Shuning uchun, avval `nextTick` navbatidagi *barcha* callback'lar (`nextTick 1`, `nextTick 2`) bajariladi. Shundan keyingina Promise navbatidagi callback'lar (`promise 1`, `promise 2`) bajariladi.

-----

### 4-savol: Barchasini aralashtirish

**Kod parchasi:**

```javascript
const fs = require('fs');

console.log('A');

fs.readFile(__filename, () => {
  console.log('C (I/O)');
  setTimeout(() => console.log('E (timer)'), 0);
  setImmediate(() => console.log('F (immediate)'));
  process.nextTick(() => console.log('D (nextTick)'));
});

Promise.resolve().then(() => console.log('B (promise)'));
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
A
B (promise)
C (I/O)
D (nextTick)
F (immediate)
E (timer)
```

**Chuqur tahlil:**

1.  **Sinxron ijro:** `console.log('A')` chiqadi. `fs.readFile` I/O uchun rejalashtiriladi. `Promise` mikrotask navbatiga qo'shiladi.
2.  **Birinchi mikrotask bosqichi:** Asosiy skript tugagach, mikrotask navbati tekshiriladi. `console.log('B (promise)')` bajariladi.
3.  **Poll bosqichi:** Hodisalar Tsikli fayl o'qib bo'linishini kutadi. O'qib bo'lingach, I/O callback'i **Poll** bosqichida bajariladi. `console.log('C (I/O)')` chiqadi.
4.  **I/O ichidagi rejalashtirish:** I/O callback'i ichida `setTimeout` keyingi **Taymerlar** bosqichi uchun, `setImmediate` **Check** bosqichi uchun, `process.nextTick` esa joriy operatsiyadan so'ng darhol bajariladigan **`nextTick` mikrotask** navbatiga qo'shiladi.
5.  **Ikkinchi mikrotask bosqichi:** I/O callback'i tugashi bilan, Hodisalar Tsikli keyingi bosqichga o'tishdan oldin mikrotask navbatini bo'shatadi. `console.log('D (nextTick)')` chiqadi.
6.  **Check va Timers bosqichlari:** Endi tsikl davom etadi. Keyingi bosqich **Check** bo'lgani uchun `console.log('F (immediate)')` chiqadi. `setTimeout` esa keyingi tsiklning **Taymerlar** bosqichida bajariladi, ya'ni `console.log('E (timer)')`.

-----

### 5-savol: `async/await` va uning mohiyati

**Kod parchasi:**

```javascript
async function asyncFunc() {
  console.log('B');
  await Promise.resolve();
  console.log('D');
}

console.log('A');
asyncFunc();
console.log('C');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
A
B
C
D
```

**Chuqur tahlil:**

1.  **Sinxron ijro:** `console.log('A')` chiqadi. `asyncFunc()` chaqiriladi.
2.  **`async` funksiya ichi:** Funksiya ichidagi birinchi `console.log('B')` sinxron ravishda bajariladi.
3.  **`await` kalit so'zi:** `await` operatoriga duch kelganda, `asyncFunc` ning qolgan qismi (`console.log('D')`) **Promise mikrotask navbatiga** qo'yiladi va funksiya ijrosi to'xtatilib, boshqaruv chaqiruvchi tomonga qaytariladi.
4.  **Asosiy skript davomi:** Boshqaruv qaytganidan so'ng, `console.log('C')` sinxron ravishda bajariladi.
5.  **Mikrotask'ni bajarish:** Asosiy skript tugagach, Hodisalar Tsikli mikrotask navbatini tekshiradi va u yerdagi `console.log('D')` ni bajaradi. `async/await` aslida `Promise.then()` uchun sintaktik qobiq ekanligi shu yerda namoyon bo'ladi.

-----

### 6-savol: Rekursiv `process.nextTick` va I/O ochligi

**Kod parchasi:**

```javascript
const fs = require('fs');
let i = 0;

function recursiveNextTick() {
  if (i < 20) { // Cheksiz tsiklni oldini olish uchun
    console.log(`nextTick loop: ${i}`);
    i++;
    process.nextTick(recursiveNextTick);
  }
}

fs.readFile(__filename, () => {
  console.log('I/O Callback - Bu hech qachon chiqmaydi!');
});

recursiveNextTick();

console.log('script start');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
script start
nextTick loop: 0
nextTick loop: 1
...
nextTick loop: 19
```

**Chuqur tahlil:**

1.  **Boshlanish:** `fs.readFile` I/O operatsiyasini rejalashtiradi. `recursiveNextTick()` birinchi marta chaqiriladi. `console.log('script start')` chiqadi.
2.  **`nextTick` tuzog'i:** Birinchi `recursiveNextTick` chaqiruvi `nextTick loop: 0` ni chiqaradi va keyingi iteratsiya uchun yana `process.nextTick(recursiveNextTick)` ni rejalashtiradi.
3.  **I/O ochligi (Starvation):** `nextTick` navbati Hodisalar Tsiklining boshqa har qanday bosqichidan, jumladan I/O callback'larini bajaradigan **Poll** bosqichidan ham yuqori ustuvorlikka ega. Har bir `nextTick` callback'i tugashi bilan, Node.js darhol `nextTick` navbatini qayta tekshiradi. Rekursiv chaqiruv tufayli bu navbat hech qachon bo'shamaydi. Natijada, Hodisalar Tsikli **Poll** bosqichiga yetib bora olmaydi va I/O callback'i hech qachon bajarilmaydi. Bu "I/O starvation" deb ataladi.

-----

### 7-savol: Hodisalar Tsiklini bloklash

**Kod parchasi:**

```javascript
console.log('start');

setTimeout(() => {
  console.log('setTimeout');
}, 10);

Promise.resolve().then(() => console.log('promise'));

// 50ms davomida tsiklni bloklash
const start = Date.now();
while (Date.now() - start < 50) {}

console.log('end');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
start
end
promise
setTimeout
```

**Chuqur tahlil:**

1.  **Sinxron boshlanish:** `console.log('start')` chiqadi. `setTimeout` 10ms dan so'ng, `Promise` esa mikrotask sifatida rejalashtiriladi.
2.  **Bloklash:** `while` tsikli sinxron koddur. U **Call Stack**'ni 50ms davomida band qilib turadi. Bu vaqt ichida Hodisalar Tsikli to'liq to'xtab turadi. Hech qanday taymer yoki I/O ishlay olmaydi. `setTimeout` ning 10ms lik vaqti o'tib ketgan bo'lsa ham, uning callback'i Call Stack bo'shaguncha bajarila olmaydi.
3.  **Bloklashdan so'ng:** 50ms dan keyin `while` tsikli tugaydi va `console.log('end')` chiqadi.
4.  **Mikrotask va Makrotask:** Endi Call Stack bo'sh. Birinchi navbatda mikrotask (`promise`) bajariladi. Shundan so'nggina Hodisalar Tsikli o'z ishini davom ettirib, **Taymerlar** bosqichiga o'tadi va vaqti allaqachon o'tib ketgan `setTimeout` callback'ini bajaradi.

-----

### 8-savol: Bo'sh Poll bosqichi va `setImmediate`

**Kod parchasi:**

```javascript
const fs = require('fs');

// Bu I/O operatsiyasi tsiklni Poll bosqichida ushlab turadi
fs.readFile(__filename, () => {
    // I/O callback
});

setTimeout(() => {
    console.log('timeout');
}, 200);

setImmediate(() => {
    console.log('immediate');
});
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi? (Faylni o'qish juda tez yakunlanadi deb faraz qiling)

**Kutilayotgan natija:**

```
immediate
timeout
```

**Chuqur tahlil:**

1.  **Rejalashtirish:** `fs.readFile` I/O uchun, `setTimeout` 200ms dan so'ng **Taymerlar** uchun, `setImmediate` esa **Check** bosqichi uchun rejalashtiriladi.
2.  **Poll bosqichining mantig'i:** Hodisalar Tsikli **Poll** bosqichiga keladi. U yerda I/O callback'ini bajaradi (bizning holatda u bo'sh). Endi **Poll** navbati bo'sh.
3.  **Keyingi qadamni tanlash:** **Poll** navbati bo'sh bo'lganda, tsikl keyingi qadamni tanlaydi:
      * Agar `setImmediate` rejalashtirilgan bo'lsa, tsikl kutib o'tirmasdan darhol **Check** bosqichiga o'tadi.
      * Agar `setImmediate` bo'lmasa, u eng yaqin taymer kelguncha yoki yangi I/O hodisasi yuz berguncha kutadi.
4.  **Natija:** Bizning holatda `setImmediate` mavjud, shuning uchun tsikl darhol **Check** bosqichiga o'tadi va `console.log('immediate')` ni bajaradi. Shundan so'ng tsikl aylanishda davom etadi va 200ms o'tgach **Taymerlar** bosqichida `console.log('timeout')` ni bajaradi.

-----

### 9-savol: Promise zanjiri va mikrotasklar

**Kod parchasi:**

```javascript
console.log('A');

Promise.resolve()
  .then(() => console.log('B'))
  .then(() => console.log('C'));

console.log('D');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
A
D
B
C
```

**Chuqur tahlil:**

1.  **Sinxron ijro:** `console.log('A')` va `console.log('D')` sinxron ravishda bajariladi.
2.  **Birinchi `.then()`:** `Promise.resolve()` darhol bajarilgan (resolved) promise qaytaradi. Birinchi `.then()` callback'i (`console.log('B')`) **Promise mikrotask navbatiga** qo'yiladi.
3.  **Ikkinchi `.then()`:** Bu yerda eng muhim jihat: ikkinchi `.then()` darhol navbatga qo'yilmaydi. U faqat birinchi `.then()` callback'i bajarilib, o'z navbatida yangi promise qaytarganidan so'ng navbatga qo'yiladi.
4.  **Bajarilish:** Sinxron kod tugagach, mikrotask navbati ishga tushadi. `console.log('B')` bajariladi. Uning bajarilishi tugagach, u qaytargan promise'ga bog'langan ikkinchi `.then()` callback'i (`console.log('C')`) endi mikrotask navbatiga qo'shiladi va darhol bajariladi, chunki Node.js bitta mikrotaskdan so'ng navbatni qayta tekshiradi.

-----

### 10-savol: Promise ichidagi Taymer

**Kod parchasi:**

```javascript
console.log('start');

Promise.resolve().then(() => {
  console.log('promise');
  setTimeout(() => {
    console.log('timeout');
  }, 0);
});

console.log('end');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
start
end
promise
timeout
```

**Chuqur tahlil:**

1.  **Sinxron ijro:** `console.log('start')` va `console.log('end')` chiqadi. Promise callback'i mikrotask navbatiga qo'yiladi.
2.  **Mikrotask ijrosi:** Sinxron kod tugagach, mikrotask navbati ishga tushadi va `console.log('promise')` chiqadi.
3.  **Makrotask rejalashtirilishi:** Mikrotask ichida `setTimeout(..., 0)` chaqiriladi. Bu yangi makrotaskni (taymerni) **keyingi** Hodisalar Tsikli aylanishining **Taymerlar** bosqichi uchun rejalashtiradi. U joriy tsiklda ishlay olmaydi.
4.  **Keyingi tsikl:** Joriy tsiklning barcha bosqichlari va mikrotasklari tugagach, yangi tsikl boshlanadi va **Taymerlar** bosqichida `console.log('timeout')` bajariladi. Bu mikrotask makrotaskni qanday rejalashtirishini ko'rsatadi.

-----

### 11-savol: Taymer ichidagi Promise

**Kod parchasi:**

```javascript
console.log('start');

setTimeout(() => {
  console.log('timeout');
  Promise.resolve().then(() => {
    console.log('promise');
  });
}, 0);

console.log('end');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
start
end
timeout
promise
```

**Chuqur tahlil:**

1.  **Sinxron ijro:** `console.log('start')` va `console.log('end')` chiqadi. `setTimeout` callback'i makrotask sifatida rejalashtiriladi.
2.  **Makrotask ijrosi:** Hodisalar Tsikli **Taymerlar** bosqichiga kelganda, `setTimeout` callback'ini bajaradi. `console.log('timeout')` chiqadi.
3.  **Mikrotask rejalashtirilishi:** Taymer callback'i ichida `Promise.resolve().then()` chaqiriladi. Bu yangi mikrotaskni rejalashtiradi.
4.  **Darhol bajarilish:** Qoida bo'yicha, har bir makrotask bajarilgandan so'ng, Hodisalar Tsikli keyingi makrotaskga yoki keyingi bosqichga o'tishdan oldin **barcha mikrotask navbatini** bo'shatishi kerak. Shuning uchun, `timeout` callback'i tugashi bilan darhol `promise` callback'i bajariladi.

-----

### 12-savol: `async/await` va `nextTick` kombinatsiyasi

**Kod parchasi:**

```javascript
async function test() {
  console.log('A');
  await new Promise(resolve => resolve());
  console.log('C');
}

test();

process.nextTick(() => {
  console.log('B');
});
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
A
B
C
```

**Chuqur tahlil:**

1.  **`async` funksiya chaqiruvi:** `test()` chaqiriladi. `console.log('A')` sinxron ravishda chiqadi.
2.  **`await` va pauza:** `await` operatori funksiyaning qolgan qismini (`console.log('C')`) **Promise mikrotask navbatiga** qo'yadi va ijroni to'xtatadi.
3.  **`nextTick` rejalashtirilishi:** Boshqaruv asosiy skriptga qaytadi. `process.nextTick` o'z callback'ini (`console.log('B')`) **`nextTick` mikrotask navbatiga** qo'yadi.
4.  **Mikrotasklar poygasi:** Asosiy skript tugagach, mikrotask navbatlari ishga tushadi. `nextTick` navbati Promise navbatidan yuqori ustuvorlikka ega bo'lgani uchun, `console.log('B')` birinchi bajariladi. Shundan keyingina Promise navbatidagi `console.log('C')` bajariladi.

-----

### 13-savol: Ikkala mikrotask navbatini ketma-ket to'ldirish

**Kod parchasi:**

```javascript
process.nextTick(() => {
  console.log('nextTick 1');
  Promise.resolve().then(() => console.log('promise 1'));
});

process.nextTick(() => {
  console.log('nextTick 2');
});

Promise.resolve().then(() => {
  console.log('promise 2');
  process.nextTick(() => console.log('nextTick 3'));
});
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
nextTick 1
nextTick 2
promise 2
nextTick 3
promise 1
```

**Chuqur tahlil:**

1.  **Dastlabki rejalashtirish:** Ikkita `nextTick` va bitta `promise` rejalashtiriladi.
2.  **`nextTick` ustuvorligi:** Birinchi bo'lib `nextTick` navbati to'liq bo'shatiladi. `console.log('nextTick 1')` chiqadi. Uning ichida yangi `promise 1` rejalashtiriladi. Keyin `console.log('nextTick 2')` chiqadi.
3.  **`nextTick` navbati bo'shadi.**
4.  **Promise navbati:** Endi Promise navbati ishga tushadi. `console.log('promise 2')` chiqadi. Uning ichida yangi `nextTick 3` rejalashtiriladi.
5.  **Navbatlarni qayta tekshirish:** Bitta mikrotask (`promise 2`) tugagach, tizim navbatlarni qayta tekshiradi. `nextTick` navbatida yangi element (`nextTick 3`) paydo bo'ldi va uning ustuvorligi yuqori bo'lgani uchun `console.log('nextTick 3')` darhol bajariladi.
6.  **Yakunlash:** `nextTick` navbati yana bo'sh. Promise navbatida qolgan `promise 1` bajariladi.

-----

### 14-savol: `setTimeout` ichida `setTimeout`

**Kod parchasi:**

```javascript
setTimeout(() => {
  console.log('A');
  setTimeout(() => {
    console.log('B');
  }, 0);
}, 10);

setTimeout(() => {
  console.log('C');
}, 10);
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
A
C
B
```

**Chuqur tahlil:**

1.  **Dastlabki taymerlar:** Ikkala `setTimeout(..., 10)` ham deyarli bir vaqtda rejalashtiriladi. Ular ikkalasi ham taxminan 10ms dan so'ng **Taymerlar** bosqichining navbatiga tushadi.
2.  **Birinchi Taymerlar bosqichi:** Taxminan 10ms o'tgach, tsikl **Taymerlar** bosqichiga kiradi. U navbatdagi callback'larni birma-bir bajaradi. Birinchi bo'lib 'A' ni chiqaradigan callback ishlaydi.
3.  **Yangi taymerni rejalashtirish:** 'A' callback'i ichida yangi `setTimeout(..., 0)` rejalashtiriladi. Bu taymer **keyingi** tsiklning **Taymerlar** bosqichi uchun navbatga qo'yiladi, joriy bosqich uchun emas.
4.  **Joriy bosqichni davom ettirish:** Tizim joriy **Taymerlar** bosqichidagi navbatni bo'shatishda davom etadi va 'C' ni chiqaradigan callback'ni bajaradi.
5.  **Keyingi tsikl:** Joriy tsikl tugagandan so'ng, yangi tsikl boshlanadi va uning **Taymerlar** bosqichida 'B' callback'i bajariladi.

-----

### 15-savol: `setImmediate` ichida `setImmediate`

**Kod parchasi:**

```javascript
setImmediate(() => {
  console.log('A');
  setImmediate(() => {
    console.log('B');
  });
});

fs.readFile(__filename, () => {
    console.log('I/O');
});

console.log('start');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
start
A
I/O
B
```

**Chuqur tahlil:**

1.  **Sinxron ijro:** `console.log('start')` chiqadi. `setImmediate` ('A') **Check** bosqichi uchun, `fs.readFile` esa **Poll** bosqichi uchun rejalashtiriladi.
2.  **Birinchi tsikl (Check):** Hodisalar tsikli birinchi bo'lib `setImmediate` uchun **Check** bosqichiga kelishi mumkin (agar taymerlar bo'lmasa). `console.log('A')` chiqadi. Uning ichida yangi `setImmediate` ('B') rejalashtiriladi. Bu callback **keyingi** tsiklning **Check** bosqichi uchun navbatga qo'yiladi.
3.  **Birinchi tsikl (Poll):** Tsikl davom etadi va **Poll** bosqichiga keladi. Fayl o'qish tugagan bo'lsa, `console.log('I/O')` chiqadi.
4.  **Ikkinchi tsikl:** Hodisalar tsikli yangi aylanishni boshlaydi. U **Check** bosqichiga kelganda, oldingi qadamda rejalashtirilgan 'B' callback'ini topadi va bajaradi. `console.log('B')` chiqadi.

-----

### 16-savol: Promise va `setImmediate` poygasi

**Kod parchasi:**

```javascript
Promise.resolve().then(() => console.log('promise'));
setImmediate(() => console.log('immediate'));
console.log('start');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
start
promise
immediate
```

**Chuqur tahlil:**
Bu mikrotask va makrotask o'rtasidagi fundamental farqni tekshiradigan klassik savol.

1.  **Rejalashtirish:** `console.log('start')` chiqadi. `Promise.then` callback'i **mikrotask** sifatida, `setImmediate` callback'i esa **makrotask** (Check bosqichi uchun) sifatida rejalashtiriladi.
2.  **Bajarilish qoidasi:** Hodisalar Tsikli qoidasi shunday: joriy sinxron kod (macrotask) tugagach, keyingi macrotask'ga o'tishdan oldin **barcha mikrotasklar navbati** to'liq bo'shatilishi kerak.
3.  **Natija:** Asosiy skript tugagach, Call Stack bo'shaydi. Hodisalar Tsikli **Check** bosqichiga o'tishdan oldin mikrotask navbatini tekshiradi va `promise` callback'ini bajaradi. Shundan keyingina tsikl o'z ishini davom ettirib, **Check** bosqichida `immediate` callback'ini bajaradi.

-----

### 17-savol: Nol soniyali `setInterval`

**Kod parchasi:**

```javascript
let i = 0;
const id = setInterval(() => {
  console.log(`interval: ${i++}`);
  if (i > 2) {
    clearInterval(id);
  }
}, 0);

console.log('start');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
start
interval: 0
interval: 1
interval: 2
```

**Chuqur tahlil:**

1.  **Sinxron ijro:** `console.log('start')` chiqadi. `setInterval(..., 0)` o'z callback'ini imkon qadar tezroq, har bir tsikl aylanishining **Taymerlar** bosqichida bajarish uchun rejalashtiradi.
2.  **Tsikl ishi:** Hodisalar Tsiklining har bir aylanishida, u **Taymerlar** bosqichiga kelganda, `setInterval` callback'i navbatda turgan bo'ladi va bajariladi. `console.log` chiqadi va `i` oshiriladi.
3.  **To'xtatish:** `i` 3 ga yetganda, `clearInterval(id)` chaqiriladi, bu esa keyingi tsikl aylanishlari uchun bu callback'ni rejalashtirishni to'xtatadi. `setTimeout(..., 0)` dan farqli o'laroq, bu har bir tsiklda qayta-qayta rejalashtiriladi.

-----

### 18-savol: I/O, `nextTick` va Promise birgalikda

**Kod parchasi:**

```javascript
const fs = require('fs');

fs.readFile(__filename, () => {
  console.log('I/O');
  Promise.resolve().then(() => console.log('promise'));
});

process.nextTick(() => console.log('nextTick'));
console.log('start');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
start
nextTick
I/O
promise
```

**Chuqur tahlil:**

1.  **Sinxron ijro va rejalashtirish:** `fs.readFile` I/O uchun, `process.nextTick` esa mikrotask uchun rejalashtiriladi. `console.log('start')` chiqadi.
2.  **Birinchi mikrotask:** Asosiy skript tugashi bilan `nextTick` mikrotask navbati bo'shatiladi. `console.log('nextTick')` chiqadi.
3.  **Poll bosqichi:** Hodisalar Tsikli I/O operatsiyasi tugashini kutadi. Tugagach, uning callback'i **Poll** bosqichida bajariladi. `console.log('I/O')` chiqadi.
4.  **Ikkinchi mikrotask:** I/O callback'i ichida yangi `Promise` mikrotaski rejalashtiriladi. I/O callback'i (bu makrotask) tugashi bilan, tsikl keyingi bosqichga o'tishdan oldin mikrotask navbatini bo'shatadi. `console.log('promise')` chiqadi.

-----

### 19-savol: `await` qanday qilib ijroni "bo'ladi"

**Kod parchasi:**

```javascript
async function main() {
  console.log('A');
  await new Promise(res => setTimeout(res, 10));
  console.log('C');
}

console.log('B');
main();
console.log('D');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
B
A
D
C
```

**Chuqur tahlil:**

1.  **Asosiy skript:** `console.log('B')` chiqadi. `main()` chaqiriladi.
2.  **`async` funksiya ichi:** `console.log('A')` sinxron ravishda chiqadi.
3.  **`await` va makrotask:** `await` 10ms kutadigan `setTimeout`'li Promise'ni kutadi. Bu `main()` funksiyasining ijrosini shu yerda to'xtatadi va boshqaruvni asosiy skriptga qaytaradi. `console.log('C')` qismi esa Promise'ning `.then()` bloki ichiga o'ralib, taymer tugagandan so'ng mikrotask sifatida bajarilishini kutadi.
4.  **Asosiy skript davomi:** Boshqaruv qaytganidan so'ng, `console.log('D')` chiqadi.
5.  **Yakunlanish:** Taxminan 10ms o'tgach, `setTimeout` callback'i `Promise`'ni `resolve` qiladi. Bu esa o'z navbatida `await` dan keyingi qismni (`console.log('C')`) mikrotask navbatiga qo'yadi va u darhol bajariladi.

-----

### 20-savol: Chuqur zanjirlangan mikrotasklar

**Kod parchasi:**

```javascript
setImmediate(() => console.log('A (immediate)'));

Promise.resolve().then(() => {
  console.log('B (promise 1)');
  process.nextTick(() => console.log('C (nextTick)'));
});

console.log('D (start)');
```

**Savol:** Konsolga nima va qanday aniq tartibda chiqariladi?

**Kutilayotgan natija:**

```
D (start)
B (promise 1)
C (nextTick)
A (immediate)
```

**Chuqur tahlil:**

1.  **Sinxron ijro:** `console.log('D (start)')` chiqadi. `setImmediate` makrotask, `Promise.then` esa mikrotask sifatida rejalashtiriladi.
2.  **Birinchi mikrotask:** Asosiy skript tugagach, mikrotask navbati ishga tushadi. `console.log('B (promise 1)')` chiqadi.
3.  **Mikrotask ichida mikrotask:** 'B' callback'i ichida `process.nextTick` chaqiriladi. Bu yangi mikrotaskni (`C`) `nextTick` navbatiga qo'yadi.
4.  **Navbatlarni qayta tekshirish:** Bitta mikrotask (`B`) tugagach, tizim navbatlarni yana boshidan tekshiradi. `nextTick` navbatida yangi element (`C`) bor va uning ustuvorligi yuqori. Shuning uchun `console.log('C (nextTick)')` darhol bajariladi.
5.  **Makrotask:** Barcha mikrotasklar tugagach, Hodisalar Tsikli nihoyat o'z aylanishini davom ettirib, **Check** bosqichiga keladi va `console.log('A (immediate)')` ni bajaradi.
