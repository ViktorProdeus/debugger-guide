# Руководство по отладке в React

## Содержание

1. [Инструменты браузера](#инструменты-браузера)
2. [Отладка с помощью Debugger](#отладка-с-помощью-debugger)
3. [React DevTools](#react-devtools)
4. [Отладка хуков](#отладка-хуков)
5. [Асинхронные операции](#асинхронные-операции)
6. [Типичные ошибки React](#типичные-ошибки-react)
7. [Продвинутые техники консоли](#продвинутые-техники-консоли)
8. [Отладка React в различных фреймворках](#отладка-react-в-различных-фреймворках)
9. [Видеоматериалы по отладке сложных случаев](#видеоматериалы-по-отладке-сложных-случаев)

---

## Инструменты браузера

### Консоль разработчика (F12 или Command+Option+I)

1. **console.log** — базовый инструмент для просмотра значений:

   ```javascript
   console.log('Переменная:', variable);
   ```

2. **console.dir** — отображение объекта как интерактивное дерево:

   ```javascript
   console.dir(document.body);
   ```

3. **console.table** — представление данных в табличном формате:

   ```javascript
   console.table([
     { name: 'John', age: 25 },
     { name: 'Anna', age: 28 },
   ]);
   ```

4. **Точки останова (Breakpoints)**

   - В панели Sources можно установить точки останова, кликнув на номер строки.
   - Временные точки останова: `debugger;` в коде.

5. **Просмотр сетевых запросов**
   - Вкладка Network для отслеживания запросов API.
   - Фильтрация по типу запроса (XHR, Fetch).

### Пример использования инструментов отладки браузера

```javascript
// Установка точки останова в коде
function fetchUserData(userId) {
  console.log('Загрузка данных для пользователя:', userId);

  // Эта команда приостановит выполнение
  debugger;

  fetch(`/api/users/${userId}`)
    .then((response) => {
      console.log('Статус ответа:', response.status);
      return response.json();
    })
    .then((data) => {
      console.table(data); // Вывод данных в таблице
      console.dir(data); // Интерактивное дерево для объекта
      return data;
    });
}
```

---

## Отладка с помощью Debugger

JavaScript Debugger — это мощный инструмент для отладки кода, позволяющий приостанавливать выполнение, исследовать переменные и пошагово проходить через код.

### Основы использования Debugger

1. **Способы установки точек останова (breakpoints)**

   - **Прямо в коде через оператор `debugger`**:

     ```javascript
     function calculateTotal(items) {
       let total = 0;

       debugger; // Выполнение приостановится здесь

       for (let item of items) {
         total += item.price * item.quantity;
       }

       return total;
     }
     ```

   - **В панели Sources DevTools браузера**:

     - Открыть вкладку Sources (F12 -> Sources)
     - Найти нужный файл
     - Кликнуть по номеру строки для установки точки останова

   - **Условные точки останова**:
     - В DevTools кликните правой кнопкой на номер строки
     - Выберите "Add conditional breakpoint"
     - Введите условие, например: `user.id === 5`

2. **Навигация при отладке**

   - **Step Over (F10)** — выполнить текущую строку и перейти к следующей
   - **Step Into (F11)** — войти внутрь функции на текущей строке
   - **Step Out (Shift+F11)** — выйти из текущей функции
   - **Resume (F8)** — продолжить выполнение до следующей точки останова

3. **Панель отладки DevTools**

   - **Call Stack** — показывает цепочку вызовов функций
   - **Scope** — отображает локальные и глобальные переменные
   - **Breakpoints** — список всех установленных точек останова
   - **Watch** — добавление выражений для отслеживания значений

### Отладка React-компонентов

1. **Отладка при рендеринге**

   ```javascript
   function UserProfile({ user }) {
     debugger; // Остановка при каждом рендере

     if (!user) {
       return <div>Загрузка...</div>;
     }

     return (
       <div>
         <h2>{user.name}</h2>
         <p>{user.email}</p>
       </div>
     );
   }
   ```

2. **Отладка обработчиков событий**

   ```javascript
   function LoginForm() {
     const [username, setUsername] = useState('');
     const [password, setPassword] = useState('');

     function handleSubmit(e) {
       e.preventDefault();

       debugger; // Остановка при отправке формы

       // Здесь можно проверить значения полей
       if (!username || !password) {
         alert('Заполните все поля!');
         return;
       }

       loginUser(username, password);
     }

     return <form onSubmit={handleSubmit}>{/* ... поля формы ... */}</form>;
   }
   ```

3. **Отладка хуков**

   ```javascript
   function Counter() {
     const [count, setCount] = useState(0);

     useEffect(() => {
       debugger; // Остановка при каждом срабатывании эффекта
       document.title = `Счетчик: ${count}`;
     }, [count]);

     return (
       <button onClick={() => setCount(count + 1)}>Нажатий: {count}</button>
     );
   }
   ```

### Примеры отлова типичных ошибок

1. **Отлов ошибок рендеринга**

   ```javascript
   function ProductList({ products }) {
     debugger;

     // Отлов ошибки: products может быть undefined или null
     if (!products) {
       console.error('Продукты не были переданы!');
       return <div>Ошибка загрузки продуктов</div>;
     }

     // Отлов ошибки: products должен быть массивом
     if (!Array.isArray(products)) {
       console.error('Products не является массивом:', products);
       return <div>Некорректные данные продуктов</div>;
     }

     return (
       <ul>
         {products.map((product) => (
           <li key={product.id}>
             {product.name} - {product.price} руб.
           </li>
         ))}
       </ul>
     );
   }
   ```

2. **Отлов ошибок в обработчиках событий**

   ```javascript
   function SearchForm() {
     const [query, setQuery] = useState('');
     const [results, setResults] = useState([]);
     const [isLoading, setIsLoading] = useState(false);
     const [error, setError] = useState(null);

     async function handleSearch(e) {
       e.preventDefault();
       setIsLoading(true);
       setError(null);

       try {
         debugger; // Остановка перед запросом

         // Проверка корректности ввода
         if (!query.trim()) {
           throw new Error('Поисковый запрос не может быть пустым');
         }

         const response = await fetch(
           `/api/search?q=${encodeURIComponent(query)}`
         );

         debugger; // Остановка после получения ответа

         // Проверка ответа сервера
         if (!response.ok) {
           throw new Error(`Ошибка сервера: ${response.status}`);
         }

         const data = await response.json();

         // Проверка структуры данных
         if (!Array.isArray(data.results)) {
           throw new Error('Некорректный формат данных от сервера');
         }

         setResults(data.results);
       } catch (err) {
         debugger; // Остановка при ошибке
         console.error('Ошибка поиска:', err);
         setError(err.message);
       } finally {
         setIsLoading(false);
       }
     }

     return (
       <div>
         <form onSubmit={handleSearch}>
           <input value={query} onChange={(e) => setQuery(e.target.value)} />
           <button type="submit" disabled={isLoading}>
             {isLoading ? 'Поиск...' : 'Искать'}
           </button>
         </form>

         {error && <div className="error">{error}</div>}

         <ul>
           {results.map((item) => (
             <li key={item.id}>{item.title}</li>
           ))}
         </ul>
       </div>
     );
   }
   ```

3. **Отлов ошибок в хуках**

   ```javascript
   function UserDashboard({ userId }) {
     const [user, setUser] = useState(null);
     const [isLoading, setIsLoading] = useState(true);
     const [error, setError] = useState(null);

     useEffect(() => {
       let isMounted = true;

       async function fetchUserData() {
         debugger; // Остановка в начале эффекта

         // Проверка userId
         if (!userId) {
           setError('ID пользователя не указан');
           setIsLoading(false);
           return;
         }

         try {
           const response = await fetch(`/api/users/${userId}`);

           // Проверка, что компонент еще смонтирован
           if (!isMounted) return;

           if (!response.ok) {
             throw new Error(`Ошибка API: ${response.status}`);
           }

           const userData = await response.json();

           debugger; // Остановка при получении данных

           // Валидация данных
           if (!userData || !userData.name) {
             throw new Error('Получены некорректные данные пользователя');
           }

           setUser(userData);
         } catch (err) {
           if (isMounted) {
             console.error('Ошибка загрузки пользователя:', err);
             setError(err.message);
           }
         } finally {
           if (isMounted) {
             setIsLoading(false);
           }
         }
       }

       fetchUserData();

       // Функция очистки для предотвращения утечек памяти
       return () => {
         isMounted = false;
       };
     }, [userId]);

     if (isLoading) return <div>Загрузка...</div>;
     if (error) return <div>Ошибка: {error}</div>;
     if (!user) return <div>Пользователь не найден</div>;

     return (
       <div>
         <h1>Профиль: {user.name}</h1>
         <p>Email: {user.email}</p>
         <p>Роль: {user.role}</p>
       </div>
     );
   }
   ```

### Отладка сложных состояний и потока данных

1. **Использование watch expressions**

   В панели отладки DevTools можно добавить выражения для постоянного отслеживания:

   - Сложные вычисления: `items.filter(i => i.isActive).length`
   - Проверки состояния: `isLoading && !error && items.length === 0`

2. **Логические точки останова**

   ```javascript
   function DataGrid({ items, onSort, onFilter }) {
     // Мониторинг перерисовок компонента
     console.count('DataGrid render');

     const sortedItems = useMemo(() => {
       debugger; // Остановка при пересчете отсортированных элементов
       console.log('Calculating sorted items');
       return [...items].sort((a, b) => a.name.localeCompare(b.name));
     }, [items]);

     // Другая часть кода...
   }
   ```

3. **Временное изменение состояния для тестирования**

   В режиме отладки вы можете:

   - Изменить значения переменных через консоль
   - Вызвать функции непосредственно в консоли
   - Проверить различные условия путем вычисления выражений

### Практические советы для эффективной отладки

1. **Подготовка кода к отладке**

   - Добавляйте информативные имена переменных
   - Разбивайте сложные операции на простые шаги
   - Используйте валидацию входных параметров

2. **Стратегия отладки сложных проблем**

   - Установите точки останова в начале подозрительной функции
   - Проверьте входные данные
   - Используйте пошаговое прохождение через код
   - Анализируйте все промежуточные значения

3. **Аудит кода с помощью debugger**

   ```javascript
   function auditFormSubmission(formData) {
     debugger;

     // Валидация всех полей
     for (const [key, value] of Object.entries(formData)) {
       if (requiredFields.includes(key) && !value) {
         console.error(`Отсутствует обязательное поле: ${key}`);
       }
     }

     // Проверка типов данных
     if (typeof formData.age !== 'number') {
       console.error('Возраст должен быть числом');
     }

     // Комплексная валидация
     if (formData.password !== formData.confirmPassword) {
       console.error('Пароли не совпадают');
     }

     // Другие проверки...
   }
   ```

---

## React DevTools

React DevTools — расширение для браузера, которое добавляет две вкладки в инструменты разработчика:

### Компоненты (Components)

1. **Поиск и выбор компонентов**

   - Поисковая панель для нахождения компонентов по имени.
   - Инструмент выбора для выделения компонентов прямо на странице.

2. **Просмотр props и state**

   - Боковая панель показывает все пропсы и состояние выбранного компонента.
   - Возможность редактирования в реальном времени.

3. **Отслеживание события рендеринга**
   - Включите опцию "Highlight updates when components render" для визуализации перерендеров.

### Профилировщик (Profiler)

1. **Запись сессии**

   - Нажмите на "Record" для начала записи и выполните действия в приложении.
   - Остановите запись и анализируйте результаты.

2. **Commit-визуализация**

   - Каждый коммит — это рендеринг, вызванный изменением состояния.
   - Анализируйте время рендеринга компонентов.

3. **Рекомендации для использования**
   - Выявление частых перерендеров.
   - Идентификация медленных компонентов.
   - Оптимизация цепочек обновлений.

### Установка React DevTools

1. **Для Chrome**: [React DevTools на Chrome Web Store](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
2. **Для Firefox**: [React DevTools на Firefox Add-ons](https://addons.mozilla.org/ru/firefox/addon/react-devtools/)

---

## Отладка хуков

### useState

1. **Отслеживание обновлений состояния**

   ```javascript
   const [count, setCount] = useState(0);

   useEffect(() => {
     console.log('Значение счетчика изменилось:', count);
   }, [count]);
   ```

2. **Обнаружение проблем с асинхронными обновлениями**

   ```javascript
   // Неправильно: состояние обновляется асинхронно
   setCount(count + 1);
   console.log(count); // Выведет старое значение!

   // Правильно: используйте функциональное обновление
   setCount((prev) => {
     console.log('Текущее значение:', prev);
     return prev + 1;
   });
   ```

### useEffect

1. **Отладка зависимостей**

   ```javascript
   useEffect(() => {
     console.log('Эффект запущен с данными:', {
       id,
       items,
       otherDeps,
     });

     // Логика эффекта

     return () => {
       console.log('Очистка эффекта с данными:', { id, items });
     };
   }, [id, items, otherDeps]);
   ```

2. **Поиск пропущенных зависимостей**

   - Используйте eslint-plugin-react-hooks и правило exhaustive-deps.
   - Временно добавьте console.log в эффект для отслеживания переменных.

3. **Отладка бесконечных циклов**

   ```javascript
   // Предотвращение бесконечных циклов при отладке
   const isFirstRender = useRef(true);

   useEffect(() => {
     console.log('Эффект запущен');

     if (isFirstRender.current) {
       isFirstRender.current = false;
       return; // Пропускаем первый вызов для отладки
     }

     // Остальная логика эффекта
   }, [dependency]);
   ```

### useContext

1. **Проверка контекста**

   ```javascript
   const value = useContext(MyContext);
   console.log('Значение контекста:', value);

   // Проверка на undefined (отсутствующий Provider)
   if (value === undefined) {
     console.warn('MyContext Provider не найден выше в дереве компонентов!');
   }
   ```

### useRef

1. **Отслеживание изменений ref**

   ```javascript
   const countRef = useRef(0);

   // useRef не вызывает перерендер при изменении, поэтому логируем
   function incrementRef() {
     countRef.current += 1;
     console.log('Новое значение ref:', countRef.current);
   }
   ```

### useCallback и useMemo

1. **Проверка мемоизации**

   ```javascript
   const memoizedValue = useMemo(() => {
     console.log('Вычисление значения');
     return expensiveCalculation(a, b);
   }, [a, b]);

   // Для проверки стабильности ссылки
   useEffect(() => {
     console.log('memoizedValue изменился!');
   }, [memoizedValue]);
   ```

---

## Асинхронные операции

### Отладка Promise

1. **Цепочки Promise**

   ```javascript
   fetch('/api/data')
     .then((response) => {
       console.log('Ответ получен:', response.status);
       if (!response.ok) {
         throw new Error(`Ошибка статуса: ${response.status}`);
       }
       return response.json();
     })
     .then((data) => {
       console.log('Данные получены:', data);
       return data;
     })
     .catch((error) => {
       console.error('Произошла ошибка:', error);
     })
     .finally(() => {
       console.log('Операция завершена');
     });
   ```

2. **Отладка async/await**
   ```javascript
   async function fetchData() {
     try {
       console.log('Начало запроса');
       const response = await fetch('/api/data');

       console.log('Статус ответа:', response.status);
       if (!response.ok) {
         throw new Error(`Ошибка статуса: ${response.status}`);
       }

       const data = await response.json();
       console.log('Полученные данные:', data);
       return data;
     } catch (error) {
       console.error('Ошибка в fetchData:', error);
       throw error; // Пробрасываем ошибку дальше
     } finally {
       console.log('Операция fetchData завершена');
     }
   }
   ```

### Отладка гонок условий (Race Conditions)

1. **Проблема**

   ```javascript
   // Проблема: последующий запрос может завершиться раньше предыдущего
   function SearchComponent() {
     const [results, setResults] = useState([]);

     function handleSearch(query) {
       fetch(`/api/search?q=${query}`)
         .then((res) => res.json())
         .then((data) => {
           setResults(data); // Какой запрос вернется первым?
         });
     }
   }
   ```

2. **Решение с использованием AbortController**

   ```javascript
   function SearchComponent() {
     const [results, setResults] = useState([]);
     const abortControllerRef = useRef(null);

     async function handleSearch(query) {
       // Отменяем предыдущий запрос
       if (abortControllerRef.current) {
         abortControllerRef.current.abort();
         console.log('Предыдущий запрос отменен');
       }

       // Создаем новый контроллер
       const abortController = new AbortController();
       abortControllerRef.current = abortController;

       try {
         console.log('Запрос для:', query);
         const response = await fetch(`/api/search?q=${query}`, {
           signal: abortController.signal,
         });

         const data = await response.json();
         console.log('Результаты для:', query, data);
         setResults(data);
       } catch (error) {
         if (error.name === 'AbortError') {
           console.log('Запрос был отменен');
         } else {
           console.error('Ошибка поиска:', error);
         }
       }
     }
   }
   ```

3. **Решение с использованием идентификаторов запросов**
   ```javascript
   function SearchComponent() {
     const [results, setResults] = useState([]);
     const lastQueryRef = useRef('');

     async function handleSearch(query) {
       // Запоминаем текущий запрос
       lastQueryRef.current = query;

       try {
         const response = await fetch(`/api/search?q=${query}`);
         const data = await response.json();

         // Проверяем, что это самый последний запрос
         if (lastQueryRef.current === query) {
           console.log('Устанавливаем результаты для:', query);
           setResults(data);
         } else {
           console.log(
             'Игнорируем результаты для:',
             query,
             'так как актуальный запрос:',
             lastQueryRef.current
           );
         }
       } catch (error) {
         console.error('Ошибка поиска:', error);
       }
     }
   }
   ```

---

## Типичные ошибки React

### Ошибки рендеринга

1. **JSX синтаксис**

   ```javascript
   // Неправильно: return должен быть на той же строке или с ()
   function BadComponent() {
     return;
     <div>Текст</div>;
   }

   // Правильно
   function GoodComponent() {
     return <div>Текст</div>;
   }
   ```

2. **Множественные корневые элементы**

   ```javascript
   // Неправильно: JSX должен иметь один корневой элемент
   function BadComponent() {
     return (
       <h1>Заголовок</h1>
       <p>Параграф</p>
     );
   }

   // Правильно: используем Fragment
   function GoodComponent() {
     return (
       <>
         <h1>Заголовок</h1>
         <p>Параграф</p>
       </>
     );
   }
   ```

3. **Отсутствующие ключи в списках**

   ```javascript
   // Неправильно: отсутствуют ключи
   function TodoList({ todos }) {
     return (
       <ul>
         {todos.map((todo) => (
           <li>{todo.text}</li>
         ))}
       </ul>
     );
   }

   // Правильно: добавляем уникальные ключи
   function TodoList({ todos }) {
     return (
       <ul>
         {todos.map((todo) => (
           <li key={todo.id}>{todo.text}</li>
         ))}
       </ul>
     );
   }
   ```

### Ошибки состояния

1. **Прямое изменение состояния**

   ```javascript
   // Неправильно: прямая мутация состояния
   function BadCounter() {
     const [count, setCount] = useState(0);

     function increment() {
       count = count + 1; // Ошибка! Не изменяйте переменную состояния напрямую
     }
   }

   // Правильно: использование setCount
   function GoodCounter() {
     const [count, setCount] = useState(0);

     function increment() {
       setCount(count + 1);
     }
   }
   ```

2. **Неправильное обновление объектов**

   ```javascript
   // Неправильно: мутация объекта состояния
   function BadForm() {
     const [user, setUser] = useState({ name: '', email: '' });

     function updateEmail(newEmail) {
       user.email = newEmail; // Мутация объекта!
       setUser(user); // Ссылка на объект не изменилась!
     }
   }

   // Правильно: создание нового объекта
   function GoodForm() {
     const [user, setUser] = useState({ name: '', email: '' });

     function updateEmail(newEmail) {
       setUser({ ...user, email: newEmail }); // Новый объект
     }
   }
   ```

3. **Асинхронные обновления состояния**

   ```javascript
   // Неправильно: состояние может быть несинхронизировано
   function BadCounter() {
     const [count, setCount] = useState(0);

     function increment() {
       setCount(count + 1);
       setCount(count + 1); // Использует одно и то же значение count!
     }
   }

   // Правильно: функциональное обновление
   function GoodCounter() {
     const [count, setCount] = useState(0);

     function increment() {
       setCount((prevCount) => prevCount + 1);
       setCount((prevCount) => prevCount + 1); // Использует обновленное значение
     }
   }
   ```

### Проблемы с эффектами

1. **Бесконечный цикл в useEffect**

   ```javascript
   // Неправильно: бесконечный цикл
   function BadComponent() {
     const [data, setData] = useState([]);

     useEffect(() => {
       setData([...data, 'new item']); // Изменяет состояние, которое в зависимостях
     }, [data]); // data изменяется -> эффект перезапускается -> data изменяется...
   }

   // Правильно: функциональное обновление
   function GoodComponent() {
     const [data, setData] = useState([]);

     useEffect(() => {
       setData((prev) => [...prev, 'new item']); // Не требует data в зависимостях
     }, []); // Запуск только при монтировании
   }
   ```

2. **Отсутствие очистки в useEffect**

   ```javascript
   // Неправильно: утечка памяти
   function BadComponent() {
     useEffect(() => {
       const timer = setInterval(() => {
         console.log('Tick');
       }, 1000);
       // Отсутствует функция очистки!
     }, []);
   }

   // Правильно: добавляем очистку
   function GoodComponent() {
     useEffect(() => {
       const timer = setInterval(() => {
         console.log('Tick');
       }, 1000);

       // Возвращаем функцию очистки
       return () => clearInterval(timer);
     }, []);
   }
   ```

### Проблемы с замыканиями (closures)

1. **Устаревшие значения в замыканиях**

   ```javascript
   // Неправильно: замыкание с устаревшим значением
   function BadComponent() {
     const [count, setCount] = useState(0);

     useEffect(() => {
       const timer = setInterval(() => {
         console.log('Count in interval:', count); // Всегда будет 0
         setCount(count + 1); // Всегда будет инкрементировать от 0!
       }, 1000);

       return () => clearInterval(timer);
     }, []); // Отсутствует count в зависимостях
   }

   // Правильно: функциональное обновление
   function GoodComponent() {
     const [count, setCount] = useState(0);

     useEffect(() => {
       const timer = setInterval(() => {
         setCount((c) => c + 1); // Используем актуальное значение
       }, 1000);

       return () => clearInterval(timer);
     }, []);

     // Альтернативный подход с ref
     const countRef = useRef(0);

     useEffect(() => {
       countRef.current = count; // Обновляем ref при изменении count
     }, [count]);
   }
   ```

---

## Продвинутые техники консоли

### Группировка логов

```javascript
console.group('Действие пользователя');
console.log('Пользователь:', user.name);
console.log('Операция:', action);

console.group('Детали запроса');
console.log('URL:', requestUrl);
console.log('Метод:', method);
console.log('Данные:', payload);
console.groupEnd();

console.log('Результат:', result);
console.groupEnd();

// Сворачиваемая группа
console.groupCollapsed('Подробная информация');
console.log('Много детальной информации...');
console.groupEnd();
```

### Стилизация логов

```javascript
console.log(
  '%cВажное сообщение!',
  'color: red; font-size: 20px; font-weight: bold;'
);

console.log(
  '%cУспех%c Операция выполнена успешно',
  'background-color: green; color: white; padding: 2px 5px; border-radius: 3px;',
  'color: green; font-weight: normal;'
);
```

### Измерение производительности

```javascript
// Измерение времени выполнения
console.time('operationName');
someExpensiveOperation();
console.timeEnd('operationName'); // Выведет: operationName: 1234.56ms

// Подсчет вызовов
function handleClick() {
  console.count('Клик по кнопке');
  // Выведет: Клик по кнопке: 1, затем Клик по кнопке: 2 и т.д.
}

// Сброс счетчика
console.countReset('Клик по кнопке');
```

### Трассировка стека

```javascript
function a() {
  b();
}
function b() {
  c();
}
function c() {
  console.trace('Трассировка стека:');
  // Выведет полный стек вызовов: a -> b -> c
}

a();
```

### Условный вывод

```javascript
// Выводит сообщение только если условие ложно
console.assert(user.age >= 18, 'Пользователь несовершеннолетний:', user);

// Выводит предупреждение
console.warn('Устаревший метод, используйте newMethod() вместо него');

// Выводит ошибку (без прерывания выполнения)
console.error('Ошибка загрузки данных:', error);
```

### Табличный вывод

```javascript
// Отображение массива объектов в виде таблицы
const users = [
  { id: 1, name: 'John', age: 25, role: 'Admin' },
  { id: 2, name: 'Anna', age: 28, role: 'User' },
  { id: 3, name: 'Mike', age: 32, role: 'Editor' },
];

// Полная таблица
console.table(users);

// Выбор только определенных колонок
console.table(users, ['name', 'role']);
```

### Глобальные обработчики ошибок

```javascript
// Перехват необработанных ошибок
window.onerror = function (message, source, lineno, colno, error) {
  console.error('Необработанная ошибка:', {
    message,
    source,
    line: lineno,
    column: colno,
    error,
  });

  // Можно отправить ошибку в систему мониторинга
  // reportErrorToMonitoring(error);

  // Вернуть true, чтобы предотвратить стандартное сообщение об ошибке
  return true;
};

// Перехват отклоненных промисов
window.addEventListener('unhandledrejection', function (event) {
  console.error('Необработанное отклонение промиса:', event.reason);
  event.preventDefault(); // Предотвращение стандартной обработки
});
```

---

## Отладка React в различных фреймворках

React используется в разных фреймворках и каждый имеет свои особенности отладки. Рассмотрим специфику отладки в Next.js и Gatsby.

### Next.js

#### Особенности архитектуры Next.js для отладки

Next.js имеет несколько режимов рендеринга, каждый со своими особенностями отладки:

1. **Server-Side Rendering (SSR)**
2. **Static Site Generation (SSG)**
3. **Client-Side Rendering**
4. **Incremental Static Regeneration (ISR)**

#### Отладка серверного рендеринга (SSR)

1. **Проблема: консоль браузера не показывает ошибки серверной части**

   Решение: Проверяйте логи в терминале, где запущен Next.js сервер.

   ```javascript
   // В методах getServerSideProps, getStaticProps или getStaticPaths
   export async function getServerSideProps() {
     try {
       // Установка точки останова для отладки серверного кода
       debugger; // Это сработает только при запуске с флагом --inspect

       const res = await fetch('https://api.example.com/data');
       const data = await res.json();

       console.log('Данные с сервера:', data); // Это появится в терминале

       return { props: { data } };
     } catch (error) {
       console.error('Ошибка SSR:', error); // Логирование для отладки
       return { props: { error: error.message } };
     }
   }
   ```

2. **Отладка Node.js сервера в Next.js**

   ```bash
   # Запустите сервер с включенным отладчиком
   NODE_OPTIONS='--inspect' next dev

   # Или в package.json
   "scripts": {
     "dev:debug": "NODE_OPTIONS='--inspect' next dev"
   }
   ```

   После этого можно:

   - Открыть Chrome и перейти на `chrome://inspect`
   - Нажать на "Open dedicated DevTools for Node"
   - Установить точки останова в коде серверной части

3. **Отладка API-роутов**

   ```javascript
   // pages/api/users.js
   export default async function handler(req, res) {
     try {
       debugger; // Работает при запуске сервера с флагом --inspect

       // Проверка параметров запроса
       const { id } = req.query;
       if (!id) {
         console.error('Отсутствует ID пользователя');
         return res.status(400).json({ error: 'ID пользователя обязателен' });
       }

       const userData = await fetchUserData(id);
       return res.status(200).json(userData);
     } catch (error) {
       console.error('Ошибка API:', error);
       return res.status(500).json({ error: error.message });
     }
   }
   ```

#### Отладка гидратации

1. **Проблема: несоответствия между серверным и клиентским рендерингом**

   ```javascript
   export default function Product({ product }) {
     useEffect(() => {
       // Проверка несоответствий гидратации
       console.log('Серверные данные:', product);
       console.log(
         'DOM после гидратации:',
         document.getElementById('product-container').textContent
       );
     }, [product]);

     // Примечание: избегайте различий в рендеринге
     // Неправильно (будет ошибка гидратации):
     // const randomId = Math.random()

     return (
       <div id="product-container">
         <h1>{product.name}</h1>
         <p>{product.description}</p>
       </div>
     );
   }
   ```

2. **Обнаружение ошибок гидратации**

   Next.js выводит в консоль браузера предупреждения о несоответствиях гидратации. Ищите сообщения:

   ```
   Warning: Text content did not match. Server: "Серверный текст" Client: "Клиентский текст"
   ```

   Чтобы решить проблему, убедитесь, что компонент рендерит одинаковый контент на сервере и клиенте:

   ```javascript
   // Неправильно: разный вывод на сервере и клиенте
   function BadComponent() {
     return <div>{typeof window === 'undefined' ? 'Сервер' : 'Клиент'}</div>;
   }

   // Правильно: использование useEffect для клиентского кода
   function GoodComponent() {
     const [isClient, setIsClient] = useState(false);

     useEffect(() => {
       setIsClient(true);
     }, []);

     return <div>{isClient ? 'Клиент' : 'Загрузка...'}</div>;
   }
   ```

#### Отладка маршрутизации

```javascript
import { useRouter } from 'next/router';

export default function DebugRouting() {
  const router = useRouter();

  useEffect(() => {
    // Отслеживание изменений маршрута
    console.log('Текущий путь:', router.pathname);
    console.log('Параметры Query:', router.query);
    console.log('Готовность данных:', router.isReady);

    // Отслеживание событий маршрутизации
    const handleRouteChangeStart = (url) =>
      console.log('Начало перехода на:', url);
    const handleRouteChangeComplete = (url) =>
      console.log('Переход завершен:', url);
    const handleRouteChangeError = (err, url) =>
      console.error('Ошибка перехода на', url, err);

    router.events.on('routeChangeStart', handleRouteChangeStart);
    router.events.on('routeChangeComplete', handleRouteChangeComplete);
    router.events.on('routeChangeError', handleRouteChangeError);

    return () => {
      router.events.off('routeChangeStart', handleRouteChangeStart);
      router.events.off('routeChangeComplete', handleRouteChangeComplete);
      router.events.off('routeChangeError', handleRouteChangeError);
    };
  }, [router]);

  return (
    <div>
      <h1>Отладка маршрутизации</h1>
      <pre>{JSON.stringify(router, null, 2)}</pre>
    </div>
  );
}
```

### Gatsby

#### Особенности архитектуры Gatsby для отладки

Gatsby создает статические сайты с помощью GraphQL для получения данных во время сборки.

1. **Отладка процесса сборки**

   ```bash
   # Запуск сервера разработки с подробными логами
   GATSBY_GRAPHQL_IDE=playground gatsby develop --verbose
   ```

2. **Отладка GraphQL запросов**

   После запуска с флагом `GATSBY_GRAPHQL_IDE=playground`, вы можете открыть GraphQL playground по адресу `http://localhost:8000/___graphql` для тестирования запросов.

   ```javascript
   // В компоненте страницы
   export default function BlogPost({ data }) {
     console.log('Данные GraphQL:', data);

     // Проверка данных
     if (!data || !data.markdownRemark) {
       console.error('Отсутствуют данные из GraphQL');
       return <div>Ошибка: данные не загружены</div>;
     }

     const post = data.markdownRemark;
     return (
       <div>
         <h1>{post.frontmatter.title}</h1>
         <div dangerouslySetInnerHTML={{ __html: post.html }} />
       </div>
     );
   }

   // GraphQL запрос
   export const query = graphql`
     query ($slug: String!) {
       markdownRemark(fields: { slug: { eq: $slug } }) {
         html
         frontmatter {
           title
           date
         }
       }
     }
   `;
   ```

3. **Отладка плагинов и gatsby-node.js**

   ```javascript
   // В gatsby-node.js
   exports.createPages = async ({ graphql, actions }) => {
     const { createPage } = actions;

     console.log('Начало создания страниц'); // Появится в терминале

     try {
       const result = await graphql(`
         query {
           allMarkdownRemark {
             edges {
               node {
                 fields {
                   slug
                 }
               }
             }
           }
         }
       `);

       // Отладка результатов запроса
       console.log(
         `Получено ${result.data.allMarkdownRemark.edges.length} страниц`
       );

       // Проверка на ошибки
       if (result.errors) {
         console.error('Ошибки GraphQL:', result.errors);
         throw result.errors;
       }

       // Создание страниц
       result.data.allMarkdownRemark.edges.forEach(({ node }) => {
         // Проверка slug
         if (!node.fields || !node.fields.slug) {
           console.warn('Отсутствует slug для узла:', node);
           return;
         }

         createPage({
           path: node.fields.slug,
           component: require.resolve(`./src/templates/blog-post.js`),
           context: { slug: node.fields.slug },
         });

         console.log(`Создана страница: ${node.fields.slug}`);
       });
     } catch (error) {
       console.error('Ошибка при создании страниц:', error);
     }
   };
   ```

4. **Отладка клиентской части Gatsby**

   ```javascript
   // В компоненте
   import React, { useEffect } from 'react';

   export default function DebugClientSide() {
     useEffect(() => {
       // Проверка, что мы на клиенте
       if (typeof window !== 'undefined') {
         console.log('Информация о клиенте:');
         console.log('- User Agent:', window.navigator.userAgent);
         console.log(
           '- Viewport:',
           `${window.innerWidth}x${window.innerHeight}`
         );
         console.log('- URL:', window.location.href);
       }
     }, []);

     // Отладка событий жизненного цикла
     useEffect(() => {
       console.log('Компонент смонтирован');
       return () => console.log('Компонент размонтирован');
     }, []);

     return (
       <div>
         <h1>Отладка клиентской части</h1>
         <button onClick={() => console.log('Кнопка нажата')}>
           Тестовая кнопка
         </button>
       </div>
     );
   }
   ```

#### Советы по отладке производительности в Gatsby

1. **Использование плагина gatsby-plugin-perf-budgets**

   Установка:

   ```bash
   npm install gatsby-plugin-perf-budgets
   ```

   Конфигурация в `gatsby-config.js`:

   ```javascript
   module.exports = {
     plugins: [
       {
         resolve: 'gatsby-plugin-perf-budgets',
         options: {
           // Опции конфигурации
         },
       },
     ],
   };
   ```

2. **Анализ бандла**

   ```bash
   # Установка gatsby-plugin-webpack-bundle-analyzer
   npm install gatsby-plugin-webpack-bundle-analyzer

   # Добавление в gatsby-config.js
   module.exports = {
     plugins: [
       {
         resolve: 'gatsby-plugin-webpack-bundle-analyzer',
         options: {
           analyzerPort: 3000,
           production: true,
         },
       },
     ],
   };
   ```

3. **Отладка медленных запросов GraphQL**

   ```javascript
   // Добавление в gatsby-config.js
   module.exports = {
     plugins: [
       // Другие плагины
     ],
     flags: {
       GRAPHQL_TRACING: true, // Включает трассировку запросов
     },
   };
   ```

---

## Материалы по отладке сложных случаев

Ниже представлен список рекомендуемых текстовых материалов для углубленного изучения техник отладки React и связанных технологий.

### Базовые техники отладки React

1. **"Отладка React приложений: от основ до профессионального уровня"**

   - **Содержание**: Пошаговое руководство по использованию Chrome DevTools и React DevTools
   - **Ссылка**: [React Docs: Debugging](https://react.dev/learn/react-developer-tools)
   - **Ключевые темы**:
     - Использование React DevTools для анализа дерева компонентов
     - Отладка рендеринга и перерендеринга
     - Работа с точками останова в JSX-коде

2. **"Отладка состояния в Redux с использованием Redux DevTools"**

   - **Содержание**: Как отлаживать сложные потоки данных в Redux
   - **Ссылка**: [Redux DevTools Documentation](https://github.com/reduxjs/redux-devtools/tree/main/docs)
   - **Ключевые темы**:
     - Отслеживание изменений состояния
     - Визуализация действий и редьюсеров
     - Работа с временной шкалой (time travel debugging)

3. **"Расширенная отладка производительности React"**
   - **Содержание**: Выявление и устранение проблем производительности
   - **Ссылка**: [React Performance Optimization](https://reactjs.org/docs/optimizing-performance.html)
   - **Ключевые темы**:
     - Профилирование с React Profiler
     - Выявление ненужных перерендеров
     - Оптимизация узких мест

### Отладка сложных асинхронных операций

1. **"Глубокое погружение в асинхронные операции React"**

   - **Содержание**: Пошаговая отладка сложных асинхронных потоков
   - **Ссылка**: [GitHub Gist: Debugging Async React](https://gist.github.com/bvaughn/60a883af01716a03a1b3285a1029be0c)
   - **Ключевые темы**:
     - Отладка промисов и async/await
     - Работа с гонками условий (race conditions)
     - Отслеживание сетевых запросов и ответов

2. **"Отладка React Query и других систем кеширования данных"**
   - **Содержание**: Решение проблем с кешированием и обновлением данных
   - **Ссылка**: [TanStack Query Docs: Debugging](https://tanstack.com/query/latest/docs/react/devtools)
   - **Ключевые темы**:
     - Анализ кеша и зависимостей запросов
     - Отладка инвалидации кеша
     - Решение проблем со стейлинг данных

### Отладка серверного рендеринга

1. **"Отладка Next.js: от клиента до сервера"**

   - **Содержание**: Полное руководство по отладке Next.js приложений
   - **Ссылка**: [Next.js Docs: Debugging](https://nextjs.org/docs/advanced-features/debugging)
   - **Ключевые темы**:
     - Отладка серверных компонентов
     - Работа с ошибками гидратации
     - Отладка API-роутов и getServerSideProps

2. **"Решение проблем с рендерингом в Gatsby"**
   - **Содержание**: Отладка процесса сборки и рендеринга Gatsby
   - **Ссылка**: [Gatsby Docs: Debugging](https://www.gatsbyjs.com/docs/debugging-the-build-process/)
   - **Ключевые темы**:
     - Отладка GraphQL запросов
     - Решение проблем с плагинами
     - Отслеживание процесса создания страниц

### Отладка нативных и мобильных приложений

1. **"Отладка React Native: продвинутые техники"**

   - **Содержание**: Инструменты и методы отладки мобильных приложений
   - **Ссылка**: [React Native Docs: Debugging](https://reactnative.dev/docs/debugging)
   - **Ключевые темы**:
     - Использование React Native Debugger
     - Отладка нативного кода
     - Профилирование производительности на устройствах

2. **"Работа с Flipper для отладки мобильных приложений"**
   - **Содержание**: Использование инструмента Facebook Flipper
   - **Ссылка**: [Flipper Documentation](https://fbflipper.com/docs/getting-started/)
   - **Ключевые темы**:
     - Мониторинг сетевых запросов
     - Инспектирование хранилища данных
     - Отладка макетов и анимаций

### Практические руководства по отладке реальных проблем

1. **"Разбор кейса: отладка утечек памяти в React"**

   - **Содержание**: Пошаговое выявление и устранение утечек памяти
   - **Ссылка**: [Fixing Memory Leaks in React](https://dev.to/codylawson/how-to-find-and-fix-memory-leaks-in-your-react-app-4pj8)
   - **Ключевые темы**:
     - Использование инструментов Memory в Chrome DevTools
     - Выявление незакрытых подписок и слушателей событий
     - Исправление проблем с useEffect и очисткой ресурсов

2. **"Отладка большого Redux приложения: практический подход"**

   - **Содержание**: Демонстрация отладки сложного потока данных
   - **Ссылка**: [Redux Debugging Workflow](https://redux.js.org/usage/configuring-your-store#integrating-the-devtools-extension)
   - **Ключевые темы**:
     - Создание селекторов для отладки
     - Логирование и отслеживание критических действий
     - Стратегии изоляции проблем

3. **"Систематический подход к отладке React"**
   - **Содержание**: Методики решения различных проблем
   - **Ссылка**: [Overreacted: A Complete Guide to useEffect](https://overreacted.io/a-complete-guide-to-useeffect/)
   - **Ключевые темы**:
     - Отладка проблем с контекстом (Context)
     - Решение проблем с порядком выполнения эффектов
     - Отладка условного рендеринга

### Статьи и руководства по отладке React

1. **"Продвинутые инструменты отладки React"**

   - **Содержание**: Обзор наиболее эффективных инструментов
   - **Ссылка**: [LogRocket: React Debugging Tools](https://blog.logrocket.com/debug-react-applications-with-these-tools/)
   - **Ключевые темы**:
     - Систематический подход к отладке
     - Работа со всеми основными инструментами
     - Решение практических задач

2. **"Глубокое погружение в React DevTools"**

   - **Содержание**: Расширенное использование инструментов разработчика
   - **Ссылка**: [React DevTools - GitHub](https://github.com/facebook/react/tree/main/packages/react-devtools)
   - **Ключевые темы**:
     - Продвинутые техники работы с React DevTools
     - Профилирование и оптимизация
     - Анализ компонентов и состояний

3. **"Отладка TypeScript в проектах React"**
   - **Содержание**: Особенности отладки типизированного кода
   - **Ссылка**: [TypeScript Documentation: Debugging](https://www.typescriptlang.org/docs/handbook/debugging-typescript.html)
   - **Ключевые темы**:
     - Отладка типов в TypeScript
     - Работа с исходными картами (source maps)
     - Продвинутые техники логирования

### Ресурсы для создания собственной инфраструктуры отладки

1. **"Создание пользовательских хуков для отладки"**

   - **Содержание**: Как создать собственные инструменты отладки
   - **Ссылка**: [React Hooks for Debugging](https://usehooks.com/useDebugValue/)
   - **Ключевые темы**:
     - Создание хуков для отслеживания состояния
     - Разработка высокоуровневых абстракций для логирования
     - Интеграция с системами мониторинга

2. **"Построение системы мониторинга ошибок для React"**
   - **Содержание**: Создание комплексной системы отчетов об ошибках
   - **Ссылка**: [React Error Boundaries Documentation](https://reactjs.org/docs/error-boundaries.html)
   - **Ключевые темы**:
     - Интеграция с ErrorBoundary
     - Сбор и анализ ошибок
     - Автоматическое восстановление после сбоев

> **Примечание**: Все указанные материалы являются учебными ресурсами для самообразования и улучшения навыков отладки React-приложений. Рекомендуется изучать их последовательно, начиная с базовых техник и постепенно переходя к более сложным случаям.

---

Это руководство охватывает основные и продвинутые техники отладки в React и популярных фреймворках, а также рекомендует текстовые материалы для углубленного изучения процесса отладки сложных случаев. Используйте эти инструменты, методы и ресурсы для эффективного обнаружения и устранения проблем в ваших приложениях.
