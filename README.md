# Лабораторная работа №3 — Разработка простой темы WordPress

**Студент:** Novac Svetlana  
**Предмет:** CMS  
**Тема:** Разработка простой темы WordPress  

---

## Цель работы

Научиться создавать собственную тему WordPress, разобраться в её минимальной структуре и принципах работы шаблонов.

---

## Среда выполнения

- **ОС:** macOS  
- **Локальный сервер:** XAMPP  
- **Путь к WordPress:** `/Applications/XAMPP/xamppfiles/htdocs/wp_lab2/`  
- **Путь к теме:** `/Applications/XAMPP/xamppfiles/htdocs/wp_lab2/wp-content/themes/usm-theme/`  
- **Редактор кода:** VS Code  

---

## Шаг 1 — Подготовка среды

1. Перешла в папку `wp-content/themes/` локальной установки WordPress.
2. Создала директорию `usm-theme` для своей темы через терминал:

```bash
cd /Applications/XAMPP/xamppfiles/htdocs/wp_lab2/wp-content/themes/
mkdir usm-theme
```
<img width="468" height="189" alt="image" src="https://github.com/user-attachments/assets/38446327-2610-45b4-900d-3569ae9892eb" />

3. Включила отладку в файле `wp-config.php`:

```php
define('WP_DEBUG', true);
```
<img width="336" height="97" alt="image" src="https://github.com/user-attachments/assets/4d350070-bdec-456d-b1a8-66f791c434cb" />

---

## Шаг 2 — Создание обязательных файлов темы

Создала все необходимые файлы через терминал:

```bash
cd /Applications/XAMPP/xamppfiles/htdocs/wp_lab2/wp-content/themes/usm-theme
touch style.css index.php functions.php header.php footer.php sidebar.php single.php page.php comments.php archive.php
```
<img width="468" height="222" alt="image" src="https://github.com/user-attachments/assets/16ea30f1-a48d-4de0-b585-07023c1cbcd9" />

### style.css

Главный файл темы. Содержит метаданные (обязательный блок комментария) и стили.

```css
/*
Theme Name: USM Theme
Theme URI: http://example.com
Author: Novac Svetlana
Author URI: http://example.com
Description: Простая тема WordPress для лабораторной работы
Version: 1.0
*/
```

### index.php

Главный шаблон темы — запасной шаблон для всех страниц.

```php
<?php get_header(); ?>

<div class="container">
    <main class="content">
        <h1>Добро пожаловать!</h1>

        <?php if ( have_posts() ) : ?>
            <?php while ( have_posts() ) : the_post(); ?>
                <article>
                    <h2><a href="<?php the_permalink(); ?>"><?php the_title(); ?></a></h2>
                    <p class="meta"><?php the_date(); ?> | <?php the_author(); ?></p>
                    <div class="excerpt"><?php the_excerpt(); ?></div>
                </article>
            <?php endwhile; ?>
        <?php else : ?>
            <p>Записей пока нет.</p>
        <?php endif; ?>
    </main>

    <?php get_sidebar(); ?>
</div>

<?php get_footer(); ?>
```

---

## Шаг 3 — Общие части шаблонов

### header.php

Содержит код шапки сайта: DOCTYPE, теги `<head>`, навигационное меню.

```php
<!DOCTYPE html>
<html <?php language_attributes(); ?>>
<head>
    <meta charset="<?php bloginfo('charset'); ?>">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title><?php wp_title('|', true, 'right'); ?><?php bloginfo('name'); ?></title>
    <?php wp_head(); ?>
</head>
<body <?php body_class(); ?>>

<header class="site-header">
    <div class="container">
        <h1 class="site-title">
            <a href="<?php echo home_url(); ?>"><?php bloginfo('name'); ?></a>
        </h1>
        <p class="site-description"><?php bloginfo('description'); ?></p>
        <nav class="main-nav">
            <?php wp_nav_menu( array( 'theme_location' => 'primary' ) ); ?>
        </nav>
    </div>
</header>
```

### footer.php

Содержит подвал сайта и закрывающие теги `</body>`, `</html>`.

```php
<footer class="site-footer">
    <div class="container">
        <p>&copy; <?php echo date('Y'); ?> <?php bloginfo('name'); ?>. Все права защищены.</p>
    </div>
</footer>

<?php wp_footer(); ?>
</body>
</html>
```

> `wp_footer()` — обязательная функция, аналогичная `wp_head()`. Нужна для корректной работы плагинов.

### Подключение в index.php

Общие части подключаются функциями:
- `get_header()` — подключает `header.php`
- `get_footer()` — подключает `footer.php`
- `get_sidebar()` — подключает `sidebar.php`

### Цикл WordPress (вывод последних записей)

```php
<?php if ( have_posts() ) : ?>
    <?php while ( have_posts() ) : the_post(); ?>
        <!-- вывод поста -->
    <?php endwhile; ?>
<?php endif; ?>
```

---

## Шаг 4 — Файл функций

### functions.php

```php
<?php

function usm_theme_scripts() {
    wp_enqueue_style(
        'usm-theme-style',
        get_stylesheet_uri(),
        array(),
        '1.0'
    );
}
add_action( 'wp_enqueue_scripts', 'usm_theme_scripts' );

function usm_theme_setup() {
    register_nav_menus( array(
        'primary' => 'Главное меню',
    ) );
    add_theme_support( 'post-thumbnails' );
    add_theme_support( 'title-tag' );
}
add_action( 'after_setup_theme', 'usm_theme_setup' );

function usm_theme_widgets_init() {
    register_sidebar( array(
        'name'          => 'Боковая панель',
        'id'            => 'sidebar-1',
        'description'   => 'Добавьте виджеты сюда',
        'before_widget' => '<div class="widget">',
        'after_widget'  => '</div>',
        'before_title'  => '<h3 class="widget-title">',
        'after_title'   => '</h3>',
    ) );
}
add_action( 'widgets_init', 'usm_theme_widgets_init' );
```

---

## Шаг 5 — Дополнительные шаблоны

### single.php — шаблон отдельного поста

Выводит полный текст поста (`the_content()`), навигацию между постами и комментарии.

### page.php — шаблон статической страницы

Выводит содержимое страницы без даты и автора (т.к. это не запись блога).

### sidebar.php — боковая панель

```php
<aside class="sidebar">
    <?php if ( is_active_sidebar('sidebar-1') ) : ?>
        <?php dynamic_sidebar('sidebar-1'); ?>
    <?php else : ?>
        <div class="widget">
            <h3 class="widget-title">О сайте</h3>
            <p>Это моя тема WordPress, созданная в рамках лабораторной работы.</p>
        </div>
    <?php endif; ?>
</aside>
```

### comments.php — комментарии

Подключается в `single.php` и `page.php` через функцию `comments_template()`.

### archive.php — архив записей

Определяет тип архива (категория, метка, дата) и выводит соответствующий заголовок и список записей.

---

## Шаг 6 — Стилизация темы

Тема оформлена в стиле **«Жар-птица»** — нежные розовые, персиковые и золотые тона с декоративными элементами.

Использованы шрифты Google Fonts:
- **Playfair Display** — для заголовков (элегантный, с засечками)
- **Lato** — для основного текста (лёгкий, читабельный)

Стилизованы следующие элементы:
- Шапка сайта — градиент от золотого к розовому, декоративное перо 🪶
- Карточки статей — белые с розовой полоской сверху и эффектом подъёма при наведении
- Боковая панель — белые виджеты с розовой полоской
- Подвал — градиент, аналогичный шапке
- Навигационное меню — полупрозрачные кнопки в шапке

---

## Шаг 7 — Скриншот темы

Добавлен файл `screenshot.png` размером 1200×900px в папку темы `usm-theme/`.

---

## Шаг 8 — Активация темы

1. В админ-панели WordPress перешла в **Внешний вид → Темы**.
2. Нашла тему **USM Theme** и нажала «Активировать».

<img width="418" height="412" alt="image" src="https://github.com/user-attachments/assets/b0591298-1959-4ebe-9b44-5b1798940d3e" />

3. Настроила навигационное меню: **Внешний вид → Меню** — добавила пункты «Главная», «Контакты» и другие страницы, назначила меню на область «Главное меню».

<img width="1468" height="216" alt="Снимок экрана 2026-03-22 в 12 05 14" src="https://github.com/user-attachments/assets/853e7dc9-77da-440c-affd-7a7662811094" />

4. Проверила отображение сайта по адресу `http://localhost/wp_lab2`.

<img width="1469" height="876" alt="Снимок экрана 2026-03-22 в 12 06 01" src="https://github.com/user-attachments/assets/db0c9343-d3af-4448-81c1-46b692861f83" />

**Контакты:**

<img width="827" height="790" alt="Снимок экрана 2026-03-22 в 12 06 47" src="https://github.com/user-attachments/assets/1e462d4f-3e72-408f-abcf-bc710f5cca17" />

**Пример свежей записи:**

<img width="780" height="785" alt="Снимок экрана 2026-03-22 в 12 07 14" src="https://github.com/user-attachments/assets/18c9b0d4-209b-4e4b-85d4-9367d12e8206" />

**Боковая панель:**

<img width="329" height="742" alt="Снимок экрана 2026-03-22 в 12 14 26" src="https://github.com/user-attachments/assets/03478d3b-c118-4393-a6bb-a740debc0f1c" />

**Раздел для комментариев, нижняя часть боковой панели и футтер:**

<img width="1470" height="453" alt="Снимок экрана 2026-03-22 в 12 15 20" src="https://github.com/user-attachments/assets/a160e578-6b27-41f7-85ce-038ba2a7a47e" />

---

## Структура файлов темы

```
usm-theme/
├── style.css         — метаданные + стили (обязательный)
├── index.php         — главный шаблон (обязательный)
├── functions.php     — функции темы
├── header.php        — шапка сайта
├── footer.php        — подвал сайта
├── sidebar.php       — боковая панель
├── single.php        — шаблон отдельного поста
├── page.php          — шаблон статической страницы
├── comments.php      — блок комментариев
├── archive.php       — архив записей
└── screenshot.png    — превью темы (1200×900px)
```

<img width="297" height="278" alt="Снимок экрана 2026-03-22 в 12 07 51" src="https://github.com/user-attachments/assets/6ea409ac-a811-4bd1-9e25-c63433a0808d" />

---

## Ответы на контрольные вопросы

### 1. Какие два файла являются обязательными для любой темы WordPress?

**`style.css`** и **`index.php`**.

`style.css` должен содержать блок метаданных в комментарии — без него WordPress не распознает папку как тему. `index.php` является главным запасным шаблоном, который используется когда нет более конкретного шаблона.

### 2. Как подключаются общие части шаблонов (header, footer, sidebar)?

С помощью встроенных функций WordPress:

```php
<?php get_header(); ?>   // подключает header.php
<?php get_footer(); ?>   // подключает footer.php
<?php get_sidebar(); ?>  // подключает sidebar.php
```

WordPress автоматически находит соответствующие файлы в папке активной темы.

### 3. Чем отличаются index.php, single.php и page.php?

| Файл | Назначение | Когда используется |
|------|-----------|-------------------|
| `index.php` | Главный запасной шаблон | Когда нет подходящего шаблона; список записей |
| `single.php` | Шаблон записи блога | При открытии отдельного поста |
| `page.php` | Шаблон статической страницы | При открытии страницы («О нас», «Контакты») |

Главное отличие: `single.php` показывает дату, автора, навигацию между постами; `page.php` — только содержимое без этих элементов.

### 4. Зачем нужен файл functions.php в теме?

`functions.php` — это «мозг» темы. В нём:

- подключаются стили и скрипты через `wp_enqueue_style()` и `wp_enqueue_scripts()`
- регистрируются области навигационных меню через `register_nav_menus()`
- регистрируются сайдбары через `register_sidebar()`
- добавляется поддержка функций WordPress через `add_theme_support()` (миниатюры постов, тег title и др.)

Без `functions.php` стили темы не будут подключены правильно, а меню и виджеты не будут работать.

---

*Лабораторная работа выполнена. Тема активирована и проверена в браузере.*
