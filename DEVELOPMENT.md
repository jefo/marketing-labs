# 🚀 Development Guide

## Краткое описание изменений

Мы кардинально улучшили UX калькулятора кешбека, сделав его более привлекательным и conversion-friendly:

### ✅ Что изменилось

1. **Компактный дизайн** - весь калькулятор теперь помещается на один экран
2. **Мгновенные результаты** - при загрузке показываются топ-3 категории с готовым расчетом
3. **Упрощенный ввод** - одна общая сумма трат вместо детализации по категориям
4. **Персональный квиз** - новая страница `/quiz` с 4 вопросами для точной персонализации
5. **Интеграция между страницами** - квиз передает параметры в калькулятор через URL
6. **Inline-редактирование трат** - можно изменить сумму прямо в заголовке, вводя только тысячи
7. **Визуализация экономии** - яркие анимированные карточки с четким акцентом на сэкономленные деньги
8. **Реалистичные пропорции** - обновлены проценты категорий по статистике Росстата 2024

### 🎯 Новая архитектура

```
/                   - Главная страница с быстрым калькулятором
/quiz              - Персональный квиз из 4 вопросов  
/?categories=x&spending=y - Калькулятор с параметрами из квиза
```

## 🛠 Техническая структура

### Frontend (Astro + Svelte)
- `pages/index.astro` - Главная страница с компактным калькулятором
- `pages/quiz.astro` - Страница персонального квиза
- `components/CashbackCalculator.svelte` - Компактный калькулятор с URL параметрами
- `components/PersonalizedQuiz.svelte` - Интерактивный квиз из 4 шагов
- `components/LoadingSpinner.svelte` - Анимации загрузки

### Логика персонализации

#### Квиз определяет:
- **Пол** → влияет на категории "Красота", "Одежда" и общий множитель трат
- **Транспорт** → добавляет категории "АЗС", "Автотовары" или "Транспорт"  
- **Город** → корректирует базовую сумму трат (Москва x1.5, СПб x1.3, и т.д.)
- **Онлайн-покупки** → влияет на категорию "Маркетплейсы" и общие траты

#### Алгоритм расчета:
```javascript
// Базовая сумма (40,000₽ среднее по России)
let monthlySpending = 40000;

// Применяем множители
monthlySpending *= cityMultiplier;    // 0.9-1.5 в зависимости от города
monthlySpending *= genderMultiplier;  // 1.0-1.2 
monthlySpending *= onlineMultiplier;  // 0.8-1.2

// Выбираем категории на основе ответов
selectedCategories = [базовые] + [дополнительные на основе ответов];

// Пересчитываем пропорции категорий чтобы сумма = 100%
// Реальные пропорции 2024: Продукты 38%, ЖКХ 16%, Рестораны 18%, и т.д.
```

#### UX улучшения v2:
```javascript
// Inline редактирование трат - пользователь вводит только тысячи
function startEditingSpending() {
  spendingInput = Math.round(totalMonthlySpending / 1000).toString(); // 45000 → "45"
}

// Визуализация экономии с анимациями
.result-card.savings {
  animation: savingsGlow 2s ease-in-out infinite alternate;
  background: linear-gradient(135deg, #4ade80 0%, #22c55e 100%);
}
```

## 💻 Команды для разработки

### Установка и запуск
```bash
cd cashback
npm install                    # Установить root dependencies
npm run install:all           # Установить все dependencies

# Разработка
npm run dev                   # Запустить frontend + backend
npm run dev:frontend         # Только frontend (порт 4321)
npm run dev:backend          # Только backend (порт 3000)

# Сборка
npm run build                # Собрать все
npm run build:frontend       # Собрать только frontend
npm run preview              # Превью production сборки
```

### Структура команд
```bash
# Утилиты
npm run clean                # Очистить dist папки
npm run lint                # Проверить код
npm test                     # Запустить тесты (пока не реализовано)
```

## 🌐 Deployment

### Статическая сборка (рекомендуется)
Проект собирается в статические HTML/CSS/JS файлы:

```bash
cd frontend
npm run build
# Файлы в frontend/dist/ готовы к деплою
```

### Варианты хостинга
1. **Netlify/Vercel** - автодеплой из GitHub
2. **GitHub Pages** - бесплатный хостинг для статики
3. **CDN** - AWS CloudFront, Cloudflare Pages
4. **Nginx** - для собственного сервера

### Пример конфига Nginx
```nginx
server {
    listen 80;
    server_name cashback-calculator.com;
    
    location / {
        root /var/www/cashback/frontend/dist;
        try_files $uri $uri/ /index.html;
    }
    
    location /quiz {
        root /var/www/cashback/frontend/dist;
        try_files $uri $uri/ /quiz/index.html;
    }
}
```

## 📊 SEO и аналитика

### Уже реализовано
- ✅ Structured data (JSON-LD) для поисковиков
- ✅ Оптимизированные meta теги 
- ✅ robots.txt и sitemap.xml
- ✅ Семантическая разметка HTML
- ✅ Адаптивность и Core Web Vitals

### Для внедрения аналитики
```html
<!-- Google Analytics 4 -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>

<!-- Яндекс.Метрика -->
<script>
   (function(m,e,t,r,i,k,a){...});
</script>
```

## 🔧 Настройка окружений

### Development
```javascript
// astro.config.mjs
export default defineConfig({
  integrations: [svelte()],
  server: {
    port: 4321,
    host: true
  }
});
```

### Production
```javascript
// astro.config.mjs  
export default defineConfig({
  integrations: [svelte()],
  output: 'static',
  build: {
    inlineStylesheets: 'auto'
  }
});
```

## 📝 TODO для продакшена

### Критичные
- [ ] Настроить домен и SSL сертификат
- [ ] Внедрить аналитику (GA4 + Яндекс.Метрика) с отслеживанием конверсий
- [ ] A/B тест анимированных vs статичных карточек экономии
- [ ] Добавить тесты для inline-редактирования и расчетов
- [ ] Настроить мониторинг производительности

### Улучшения
- [ ] Добавить больше вариантов товаров для визуализации экономии
- [ ] Интеграция с реальными API банков и актуальными ценами маркетплейсов
- [ ] PWA функциональность с push-уведомлениями о новых предложениях
- [ ] Темная тема с сохранением ярких акцентов на экономии
- [ ] Мультиязычность (приоритет - регионы с высоким банковским проникновением)

## 🐛 Известные ограничения

1. **JavaScript обязателен** - без JS калькулятор не работает
2. **Локальные данные** - настройки не сохраняются между сессиями
3. **Статичные данные** - кешбек проценты и товары захардкожены, не из API банков/маркетплейсов
4. **Мобильные анимации** - на слабых устройствах могут тормозить (добавить prefers-reduced-motion)
5. **Inline-редактирование** - работает только с клавиатуры, нет touch-оптимизации для мобильных

## 📞 Поддержка

Если нужно внести изменения или есть вопросы по коду:

1. **Изменить категории и пропорции** → `cashback/frontend/src/components/CashbackCalculator.svelte` (строки 20-39)
2. **Модифицировать квиз** → `cashback/frontend/src/components/PersonalizedQuiz.svelte`  
3. **Настроить анимации экономии** → inline `<style>` блоки, поиск по "animation:"
4. **Обновить товары для демонстрации** → массив `purchaseExamples` в калькуляторе
5. **Изменить логику inline-редактирования** → функции `startEditingSpending` и `finishEditingSpending`
6. **SEO/мета-теги** → `cashback/frontend/src/pages/*.astro`

### 🎯 Ключевые метрики для отслеживания:
- Конверсия из квиза в калькулятор
- Время взаимодействия с inline-редактированием трат
- Клики по карточкам товаров (показывает привлекательность визуализации)
- Процент пользователей, которые меняют категории после квиза

Проект готов к продакшену! 🚀