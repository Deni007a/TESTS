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
