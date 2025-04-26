# Руководство по отладке в React

## Содержание
1. [Инструменты браузера](#инструменты-браузера)
2. [React DevTools](#react-devtools)
3. [Отладка хуков](#отладка-хуков)
4. [Асинхронные операции](#асинхронные-операции)
5. [Типичные ошибки React](#типичные-ошибки-react)
6. [Продвинутые техники консоли](#продвинутые-техники-консоли)

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
     { name: 'Anna', age: 28 }
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
    .then(response => {
      console.log('Статус ответа:', response.status);
      return response.json();
    })
    .then(data => {
      console.table(data); // Вывод данных в таблице
      console.dir(data);   // Интерактивное дерево для объекта
      return data;
    });
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
   setCount(prev => {
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
       otherDeps
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
     .then(response => {
       console.log('Ответ получен:', response.status);
       if (!response.ok) {
         throw new Error(`Ошибка статуса: ${response.status}`);
       }
       return response.json();
     })
     .then(data => {
       console.log('Данные получены:', data);
       return data;
     })
     .catch(error => {
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
         .then(res => res.json())
         .then(data => {
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
           signal: abortController.signal
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
           console.log('Игнорируем результаты для:', query, 
                      'так как актуальный запрос:', lastQueryRef.current);
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
     return
       <div>Текст</div>;
   }
   
   // Правильно
   function GoodComponent() {
     return (
       <div>Текст</div>
     );
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
         {todos.map(todo => (
           <li>{todo.text}</li>
         ))}
       </ul>
     );
   }
   
   // Правильно: добавляем уникальные ключи
   function TodoList({ todos }) {
     return (
       <ul>
         {todos.map(todo => (
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
       setCount(prevCount => prevCount + 1);
       setCount(prevCount => prevCount + 1); // Использует обновленное значение
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
       setData(prev => [...prev, 'new item']); // Не требует data в зависимостях
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
         setCount(c => c + 1); // Используем актуальное значение
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
function a() { b(); }
function b() { c(); }
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
  { id: 3, name: 'Mike', age: 32, role: 'Editor' }
];

// Полная таблица
console.table(users);

// Выбор только определенных колонок
console.table(users, ['name', 'role']);
```

### Глобальные обработчики ошибок

```javascript
// Перехват необработанных ошибок
window.onerror = function(message, source, lineno, colno, error) {
  console.error(
    'Необработанная ошибка:',
    {
      message,
      source,
      line: lineno,
      column: colno,
      error
    }
  );
  
  // Можно отправить ошибку в систему мониторинга
  // reportErrorToMonitoring(error);
  
  // Вернуть true, чтобы предотвратить стандартное сообщение об ошибке
  return true;
};

// Перехват отклоненных промисов
window.addEventListener('unhandledrejection', function(event) {
  console.error('Необработанное отклонение промиса:', event.reason);
  event.preventDefault(); // Предотвращение стандартной обработки
});
```

---

Это руководство охватывает основные и продвинутые техники отладки в React. Используйте эти инструменты и методы для эффективного обнаружения и устранения проблем в ваших приложениях.