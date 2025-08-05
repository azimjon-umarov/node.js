 
1. **Thread vs Process – asosiy farqlar**
2. **Worker Threads – ko‘p oqimli ishlov**
3. **Child Process – farzand jarayonlar**
4. **Code Quality Tools – kod sifati uchun vositalar**
5. **Misollar bilan amaliy ko‘rsatmalar**

---

## 1. 📌 **Thread vs Process (Oqim va Jarayon)**

### 🔹 Process (Jarayon):

* Dastur bajarilayotgan mustaqil operatsion tizim muhitidir.
* Har bir **Node.js ilova**si bitta **process**da ishlaydi.
* Har bir process o‘z xotirasiga ega, boshqa processlar bilan to‘g‘ridan-to‘g‘ri ma’lumot ulasha olmaydi (IPC orqali muloqot qiladi).

### 🔹 Thread (Oqim):

* Jarayon ichida ishlovchi kichik bajariladigan birlik.
* **Threadlar** xotirani jarayon ichida bo‘lishadi.
* Node.js default holatda **Single-threaded** (bitta oqimda ishlaydi), lekin **I/O** operatsiyalarni **libuv** yordamida asinxron boshqaradi.

### Farqlar:

| Xususiyat       | Process          | Thread                               |
| --------------- | ---------------- | ------------------------------------ |
| Xotira          | Har biri alohida | Xotirani bo‘lishadi                  |
| Mustaqillik     | To‘liq mustaqil  | Jarayon ichida bog‘langan            |
| Ishga tushirish | Og‘irroq         | Yengilroq                            |
| Xatolik ta’siri | Faqat o‘ziga     | Boshqa threadlarga ham ta’sir qiladi |
| Muloqot         | IPC orqali       | Memory orqali                        |

---

## 2. 🧵 **Worker Threads (Ko‘p Oqimli Ishlov)**

Node.js 10.5+ versiyadan boshlab, **CPU-bound** (og‘ir hisob-kitobli) ishlarni bajarish uchun **`worker_threads`** moduli joriy qilingan.

### ⚠️ Qachon ishlatiladi?

* Ko‘p CPU resursini talab qiladigan ishlar (kripto, ma'lumotlarni siqish, statistikalar)
* Asosiy oqimni bloklamaslik uchun

### 📄 Misol:

```js
// main.js
const { Worker } = require('worker_threads');

function runService(workerData) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js', { workerData });
    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', code => {
      if (code !== 0)
        reject(new Error(`Worker stopped with exit code ${code}`));
    });
  });
}

runService(10).then(result => {
  console.log('Faktorial:', result); // 3628800
});
```

```js
// worker.js
const { parentPort, workerData } = require('worker_threads');

function factorial(n) {
  return n === 0 ? 1 : n * factorial(n - 1);
}

const result = factorial(workerData);
parentPort.postMessage(result);
```

---

## 3. 👶 **Child Process (Farzand Jarayonlar)**

**`child_process`** moduli yordamida boshqa Node.js skriptlarni yoki OS buyruqlarini ishga tushirish mumkin.

### 4 usuli mavjud:

* `exec`
* `execFile`
* `spawn`
* `fork`

### 📄 Misol (`fork` bilan):

```js
// parent.js
const { fork } = require('child_process');

const child = fork('child.js');
child.send(5);

child.on('message', (msg) => {
  console.log('Childdan natija:', msg); // Masalan, 120 (5!)
});
```

```js
// child.js
process.on('message', (msg) => {
  const faktorial = (n) => n === 0 ? 1 : n * faktorial(n - 1);
  process.send(faktorial(msg));
});
```

**Farq WorkerThread va Child Process o‘rtasida:**

| Xususiyat    | Worker Threads | Child Process             |
| ------------ | -------------- | ------------------------- |
| Xotira       | Bir xil        | Alohida                   |
| Muloqot      | Shared memory  | IPC (message passing)     |
| Ishlatilishi | CPU-bound      | I/O-bound yoki OS komanda |
| Tezlik       | Tezroq         | Sekinroq                  |

---

## 4. ✅ **Code Quality Tools (Kod sifatini nazorat qilish vositalari)**

Node.js loyihalarida kod sifati muhim! Quyidagi vositalar bu ishda yordam beradi:

### 🔧 1. **ESLint**

* JavaScript linting (kod xatolarini tahlil qiladi)
* Coding standardni majburlaydi
* Konfiguratsiya: `.eslintrc.js` yoki `.eslintrc.json`

```bash
npm install eslint --save-dev
npx eslint --init
```

### 🔧 2. **Prettier**

* Kodni avtomatik formatlaydi
* ESLint bilan birga ishlashi mumkin

```bash
npm install --save-dev prettier
npx prettier --write .
```

### 🔧 3. **Husky + lint-staged**

* Git hooklarida avtomatik lint qilish
* `pre-commit` hook

```bash
npm install husky lint-staged --save-dev
npx husky install
```

### 🔧 4. **Jest / Mocha / Chai**

* Unit test framework
* Kodning to‘g‘riligini avtomatik tekshiradi

### 🔧 5. **SonarQube**

* Katta loyihalar uchun – kod sifati, duplication, security issues ko‘rsatadi

---

## 🔚 Xulosa

| Tushuncha            | Maqsad                                                     |
| -------------------- | ---------------------------------------------------------- |
| `Process`            | Mustaqil ishchi jarayon                                    |
| `Thread`             | Jarayon ichidagi oqim                                      |
| `Worker Thread`      | CPU-heavy ishlar uchun Node.jsda ko‘p oqimli ishlash       |
| `Child Process`      | Tashqi skriptlar va OS komandalarni ishga tushurish        |
| `Code Quality Tools` | Kodni toza, tushunarli, va professional saqlash vositalari |
 
