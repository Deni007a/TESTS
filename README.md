# 🧪 Настройка тестовой среды: Jest + React + TypeScript

Полноценная конфигурация для unit-тестов в React + TypeScript + Vite проекте.

---

## 📦 Установка зависимостей

```bash
npm install --save-dev jest ts-jest @types/jest \
  jest-environment-jsdom identity-obj-proxy \
  @testing-library/react @testing-library/jest-dom \
  @babel/preset-env @babel/preset-react ts-node

№№ Зачем:

jest, ts-jest, @types/jest — запуск тестов с TypeScript

identity-obj-proxy — мок для стилей

jest-environment-jsdom — окружение браузера (для document, window)

@testing-library/react, jest-dom — удобные матчеры и рендеринг

ts-node — позволяет Jest читать jest.config.ts

@babel/preset-react — трансформирует JSX в валидный JS

№№ ⚙️ Конфигурация jest.config.ts

```ts
export default {
  // ▶ Используем ts-jest для TypeScript
  preset: 'ts-jest',

  // ▶ Окружение как в браузере (document, window)
  testEnvironment: 'jest-environment-jsdom',

  // ▶ Расширенные матчеры (toBeInTheDocument и т.д.)
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],

  // ▶ Обработка ts/tsx файлов
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },

  // ▶ Мокаем импорты стилей
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },

  // ▶ Учитываемые файлы для покрытия
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/__tests__/**',
    '!src/index.tsx',
    '!src/setupTests.ts'
  ],

  // ▶ Минимальное покрытие — по заданию
  coverageThreshold: {
    global: {
      statements: 80,
      branches: 50,
      functions: 50,
      lines: 50
    }
  }
};
```

## 📘 tsconfig.app.json

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "target": "ESNext",
    "declaration": true,
    "jsx": "react-jsx",                   // ✅ поддержка JSX
    "esModuleInterop": true,              // ✅ корректные импорты
    "allowSyntheticDefaultImports": true,
    "moduleResolution": "Node",
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "resolveJsonModule": true,
    "skipLibCheck": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

## 🔧 Babel-конфигурация babel.config.cjs

```js
module.exports = {
  presets: [
    ['@babel/preset-env', { targets: { node: 'current' } }],
    ['@babel/preset-react', { runtime: 'automatic' }]
  ]
};
```

Зачем:

  Babel нужен для трансформации JSX в тестах

  Используется вместе с ts-jest при JSX внутри .test.tsx

## 🧪 Setup файл для Jest (jest.setup.ts)
Зачем:

  Подключает расширенные матчеры:

    toBeInTheDocument()
    toHaveTextContent()
    toBeVisible() и т.д.

## 🐶 Настройка Husky-хуков
```
npx husky install

npx husky add .husky/pre-commit "npm run format && npm run lint"
npx husky add .husky/pre-push "npm run test"
```

## 🚀 Команды тестирования
npm run test           # Запуск всех тестов
npm run test:coverage  # Показывает процент покрытия
npx jest path/to/file  # Запустить конкретный тест


✅ После настройки

  ✅ Можно писать .test.tsx рядом с компонентами
  ✅ Поддержка JSX и браузерных API
  ✅ Проверка покрытия и пропущенных участков
  ✅ Тесты запускаются автоматически перед git push

## 🧠 Частые ошибки и решения

| Ошибка | Решение |
|--------|---------|
| `Cannot use JSX unless the '--jsx' flag is...` | Установи `"jsx": "react-jsx"` в `tsconfig.app.json` |
| `Jest failed to parse a file... JSX not enabled` | Добавь `babel.config.cjs` с `@babel/preset-react` |
| `Failed to parse jest.config.ts (ts-node)` | Установи `ts-node` |
| `JSX не работает с type: module` | Используй `.cjs` для `babel.config` |


# 🧪 Testing Library React: Полный справочник

## 🔍 Основные методы поиска

| Метод                          | Лучше всего для...                          | Особенности                          |
|--------------------------------|--------------------------------------------|--------------------------------------|
| `getByText()`                  | Текстовых элементов (`<p>`, `<span>`)      | Точное совпадение или RegExp        |
| `getByRole()`                  | Интерактивных элементов (`button`, `a`)    | Поддерживает `name` для уточнения   |
| `getByLabelText()`             | Поля форм с `<label>`                      | Идеален для доступных форм          |
| `getByPlaceholderText()`       | Поля ввода без label                       | Не рекомендуется для тестов         |
| `getByAltText()`               | Изображений (`<img alt="...">`)            | Для иконок и картинок               |
| `getByDisplayValue()`          | Заполненных полей (`input`, `textarea`)    | Проверка предустановленных значений |
| `getByTestId()`                | Крайние случаи (нет семантики)             | Использовать `data-testid`          |

**Пример:**
```tsx
const input = screen.getByRole('textbox', { name: /username/i });
const button = screen.getByText('Submit');

## ⏳ Асинхронные методы
Метод	Когда использовать?
findByText()	Элемент появится после загрузки
findAllByRole()	Несколько динамических элементов
waitFor()	Кастомные асинхронные условия

```
const loading = await screen.findByText('Loading...');
await waitFor(() => expect(mockApi).toHaveBeenCalled());
```

## 🧰 События: fireEvent vs userEvent

| Метод                     | Назначение                                                                 | Разница                                                                 |
|---------------------------|---------------------------------------------------------------------------|-------------------------------------------------------------------------|
| **`fireEvent.change()`**  | Ввод текста в `<input>`, `<textarea>`                                      | Мгновенное изменение значения                                          |
| **`fireEvent.click()`**   | Имитация клика                                                            | Без задержки между событиями                                           |
| **`fireEvent.submit()`**  | Отправка формы                                                            | Не вызывает реальный submit-запрос                                    |
| **`fireEvent.focus()`**   | Элемент получает фокус                                                    | Не имитирует полный цикл фокусировки                                  |
| **`fireEvent.blur()`**    | Элемент теряет фокус                                                      | Аналогично нативному blur                                             |
| **`fireEvent.keyDown()`** | Нажатие клавиши                                                           | Можно указать `{ key: 'Enter' }`                                       |
| **`userEvent.type()`**    | Постепенный ввод текста (как у пользователя)                              | Включает `keyDown`, `keyPress`, `keyUp` + задержки между символами    |
| **`userEvent.click()`**   | Реалистичный клик (с hover/pre-click)                                     | Более близко к поведению браузера                                      |
| **`userEvent.dblClick()`**| Двойной клик                                                              | Включает задержку между кликами                                       |
| **`userEvent.tab()`**     | Переход по фокусам (`Tab`/`Shift+Tab`)                                    | Учитывает tabindex                                                    |
| **`userEvent.paste()`**   | Вставка из буфера                                                         | Эмулирует полный цикл paste-события                                   |
| **`userEvent.clear()`**   | Очистка поля ввода                                                        | Последовательность: выделение → удаление                              |
| **`userEvent.selectOptions()`** | Выбор опции в `<select>`                                           | Поддерживает multiple-выбор                                           |
| **`userEvent.upload()`**  | Загрузка файлов                                                           | Эмулирует событие `change` для `<input type="file">`                   |

### 🔹 Примеры использования

```tsx
// Простой клик (fireEvent)
fireEvent.click(submitButton);

// Реалистичный ввод текста (userEvent)
await userEvent.type(searchInput, 'React Testing Library');

// Комплексный сценарий
await userEvent.tab();
await userEvent.paste('copied text');
await userEvent.keyboard('{Enter}');

## ✅ Проверки через expect (Assertions)

### 🔍 Основные матчеры элементов

| Матчер                          | Проверяет...                              | Пример использования                  |
|----------------------------------|------------------------------------------|---------------------------------------|
| `toBeInTheDocument()`           | Присутствие элемента в DOM               | `expect(submitBtn).toBeInTheDocument()`|
| `toHaveTextContent()`           | Текст (полный или частичный)             | `expect(header).toHaveTextContent(/hello/i)` |
| `toHaveValue()`                 | Значение полей ввода                     | `expect(input).toHaveValue('test')`   |
| `toBeDisabled()`/`toBeEnabled()`| Состояние активности элемента            | `expect(button).toBeDisabled()`       |
| `toHaveClass()`                 | Наличие CSS-класса                       | `expect(div).toHaveClass('active')`   |
| `toBeVisible()`                 | Видимость (не `opacity: 0`/`hidden`)     | `expect(modal).toBeVisible()`         |
| `toHaveAttribute()`             | Атрибут с опциональным значением         | `expect(link).toHaveAttribute('href', '#')` |
| `toHaveStyle()`                 | CSS-стили (инлайновые или через классы)  | `expect(el).toHaveStyle('color: rgb(255, 0, 0)')` |
| `toBeChecked()`                 | Состояние чекбокса/радио-кнопки          | `expect(checkbox).toBeChecked()`      |
| `toBeEmptyDOMElement()`         | Элемент не содержит дочерних нод         | `expect(container).toBeEmptyDOMElement()` |
| `toContainElement()`            | Наличие дочернего элемента               | `expect(form).toContainElement(input)`|

**Пример комплексной проверки:**
```tsx
expect(screen.getByTestId('error-message'))
  .toBeInTheDocument()
  .toHaveClass('text-red-500')
  .toHaveTextContent(/invalid/i);

## 🧪 Полный справочник проверок и моков

### ✅ Проверки элементов (Assertions)

| Матчер                         | Назначение                          | Пример                          |
|---------------------------------|------------------------------------|---------------------------------|
| `toBeInTheDocument()`          | Наличие элемента в DOM            | `expect(el).toBeInTheDocument()`|
| `toHaveTextContent(text)`      | Текстовое содержимое              | `expect(el).toHaveTextContent('Hello')`|
| `toHaveValue(value)`           | Значение полей ввода              | `expect(input).toHaveValue('test')`|
| `toBeDisabled()`/`toBeEnabled()`| Активность элемента               | `expect(btn).toBeDisabled()`    |
| `toHaveClass(class)`           | CSS-класс элемента                | `expect(div).toHaveClass('active')`|
| `toBeVisible()`               | Видимость элемента               | `expect(modal).toBeVisible()`   |
| `toHaveAttribute(attr, value)`| Атрибуты элемента                | `expect(a).toHaveAttribute('href', '#')`|
| `toBeChecked()`               | Состояние чекбокса               | `expect(chk).toBeChecked()`     |
| `toHaveStyle(styles)`         | CSS-стили элемента               | `expect(el).toHaveStyle('color: red')`|
| `toContainElement(child)`     | Наличие дочернего элемента       | `expect(form).toContainElement(input)`|

### 🔬 Моки и вызовы (Jest)

| Метод                           | Назначение                          | Пример                          |
|---------------------------------|------------------------------------|---------------------------------|
| `jest.fn()`                    | Создание mock-функции             | `const mock = jest.fn()`        |
| `toHaveBeenCalled()`           | Факт вызова функции               | `expect(mock).toHaveBeenCalled()`|
| `toHaveBeenCalledTimes(n)`     | Количество вызовов                | `expect(mock).toHaveBeenCalledTimes(2)`|
| `toHaveBeenCalledWith(args)`   | Аргументы вызова                  | `expect(mock).toHaveBeenCalledWith('arg')`|
| `toHaveBeenLastCalledWith(args)`| Аргументы последнего вызова       | `expect(mock).toHaveBeenLastCalledWith(42)`|
| `jest.mock(module)`            | Мокирование модуля                | `jest.mock('../api')`           |
| `mockResolvedValue(value)`     | Успешный Promise                  | `api.get.mockResolvedValue(data)`|
| `mockRejectedValue(error)`     | Ошибочный Promise                 | `api.post.mockRejectedValue(err)`|

### 🧩 Комбинированные проверки

```tsx
// Проверка нескольких свойств
expect(button)
  .toBeInTheDocument()
  .toBeEnabled()
  .toHaveTextContent('Submit');

// Проверка частичных аргументов
expect(api.update).toHaveBeenCalledWith(
  expect.objectContaining({ id: expect.any(Number) })
);
```
📌 Примеры использования

Проверка компонента:
```tsx

test('should render button', () => {
  render(<Button>Click</Button>);
  expect(screen.getByRole('button'))
    .toBeInTheDocument()
    .toHaveTextContent('Click');
});
```
Проверка моков API:
```tsx

test('should call API', async () => {
  api.get.mockResolvedValue({ data: [] });
  await render(<Component />);
  expect(api.get).toHaveBeenCalledWith('/endpoint');
});
```
Проверка ошибок:
```tsx

test('should show error', () => {
  jest.spyOn(console, 'error').mockImplementation();
  expect(() => render(<BrokenComponent />)).toThrow('Error');
});
```
