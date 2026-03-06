# Frontend Setup

**This file is a placeholder. The user must customize the frontend design for their specific project.**

Read `FRONTEND_FRAMEWORK`, `USE_I18N`, and `USE_RTL` from `0-PROJECT_DESCRIPTION.md` before proceeding. Use those values to select which sections of this file to execute. Skip sections that do not apply.

---

## Chosen Framework

The active frontend framework for this project is defined in `0-PROJECT_DESCRIPTION.md` → Technical Requirements → Frontend → **"Sample selection for this project"**.

Supported options:

| Option                     | Best for                                                              |
| -------------------------- | --------------------------------------------------------------------- |
| **AdminLTE v4.0rc**        | Admin panels, dashboards, back-office apps with sidebar navigation    |
| **Bootstrap 5**            | General-purpose apps, minimal overhead, no predefined layout          |
| **Tailwind CSS + DaisyUI** | Utility-first styling, high design flexibility, requires a build step |

Jump to the section matching the chosen framework, then proceed to the i18n / RTL section.

---

## Option A — AdminLTE v4.0rc (Default Sample)

AdminLTE v4 is built on Bootstrap 5 and provides a full admin dashboard layout out of the box (sidebar, topbar, content area, cards, etc.).

_Note_: Project is using AdminLTE 4.0 rc (Relase Candidate) builds because it support RTL languages, and is based on Bootstrap 5.

### Install via npm

```bash
npm install admin-lte@4.0.0-rc3
npm install @fortawesome/fontawesome-free
```

Copy assets to public (or configure your build tool):

```bash
cp -r node_modules/admin-lte/dist public/vendor/adminlte
cp -r node_modules/@fortawesome/fontawesome-free/css public/vendor/fontawesome/css
cp -r node_modules/@fortawesome/fontawesome-free/webfonts public/vendor/fontawesome/webfonts
```

### Base Layout — templates/base.html.twig

```twig
<!DOCTYPE html>
<html lang="{{ app.request.locale }}"
      dir="{{ app.request.locale == 'ar' ? 'rtl' : 'ltr' }}">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="csrf-token" content="{{ csrf_token('global') }}">
    <title>{% block title %}{{ project_name }}{% endblock %}</title>

    {# AdminLTE — choose LTR or RTL build based on locale #}
    {% if app.request.locale == 'ar' %}
        <link rel="stylesheet" href="{{ asset('vendor/adminlte/css/adminlte.rtl.min.css') }}">
    {% else %}
        <link rel="stylesheet" href="{{ asset('vendor/adminlte/css/adminlte.min.css') }}">
    {% endif %}

    {# Font Awesome — Option A: self-hosted via npm #}
    <link rel="stylesheet" href="{{ asset('vendor/fontawesome/css/all.min.css') }}">

    {# Font Awesome — Option B: CDN Kit (replace XXXXXXXX with your kit ID) #}
    {# <script src="https://kit.fontawesome.com/XXXXXXXX.js" crossorigin="anonymous"></script> #}

    <link rel="stylesheet" href="{{ asset('css/app.css') }}">
    {% block stylesheets %}{% endblock %}
</head>
<body class="layout-fixed sidebar-expand-lg bg-body-tertiary">

<div class="app-wrapper">

    {# Top navigation bar #}
    <nav class="app-header navbar navbar-expand bg-body">
        <div class="container-fluid">
            <ul class="navbar-nav">
                <li class="nav-item">
                    <a class="nav-link" data-lte-toggle="sidebar" href="#" role="button">
                        <i class="bi bi-list"></i>
                    </a>
                </li>
            </ul>
            <ul class="navbar-nav ms-auto">
                {% if app.user %}
                {# Locale switcher — only shown if i18n is enabled #}
                {% if constant('USE_I18N') is defined %}
                <li class="nav-item dropdown me-2">
                    <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
                        <i class="fa-solid fa-globe"></i>
                        {{ app.request.locale|upper }}
                    </a>
                    <ul class="dropdown-menu dropdown-menu-end">
                        <li><a class="dropdown-item" href="{{ path('app_switch_locale', {locale: 'en'}) }}">English</a></li>
                        <li><a class="dropdown-item" href="{{ path('app_switch_locale', {locale: 'ar'}) }}">العربية</a></li>
                    </ul>
                </li>
                {% endif %}
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
                        <i class="fa-solid fa-user-circle"></i>
                        {{ app.user.username }}
                    </a>
                    <ul class="dropdown-menu dropdown-menu-end">
                        <li>
                            <a class="dropdown-item" href="{{ path('app_logout') }}">
                                <i class="fa-solid fa-right-from-bracket"></i>
                                {{ 'nav.logout'|trans }}
                            </a>
                        </li>
                    </ul>
                </li>
                {% endif %}
            </ul>
        </div>
    </nav>

    {# Sidebar #}
    <aside class="app-sidebar bg-body-secondary shadow" data-bs-theme="dark">
        <div class="sidebar-brand">
            <a href="{{ path('app_dashboard') }}" class="brand-link">
                <span class="brand-text fw-light">{{ project_name }}</span>
            </a>
        </div>
        <div class="sidebar-wrapper">
            <nav class="mt-2">
                <ul class="nav sidebar-menu flex-column" data-lte-toggle="treeview">
                    <li class="nav-item">
                        <a href="{{ path('app_dashboard') }}" class="nav-link">
                            <i class="nav-icon fa-solid fa-gauge"></i>
                            <p>{{ 'nav.dashboard'|trans }}</p>
                        </a>
                    </li>
                    {% block sidebar_items %}{% endblock %}
                </ul>
            </nav>
        </div>
    </aside>

    {# Main content #}
    <main class="app-main">
        <div class="app-content-header">
            <div class="container-fluid">
                <div class="row">
                    <div class="col-sm-6">
                        <h3 class="mb-0">{% block page_title %}{% endblock %}</h3>
                    </div>
                </div>
            </div>
        </div>
        <div class="app-content">
            <div class="container-fluid">

                {# Flash messages #}
                {% for type, messages in app.flashes %}
                    {% for message in messages %}
                        <div class="alert alert-{{ type == 'error' ? 'danger' : type }} alert-dismissible fade show">
                            {{ message }}
                            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                        </div>
                    {% endfor %}
                {% endfor %}

                {% block body %}{% endblock %}
            </div>
        </div>
    </main>

    <footer class="app-footer">
        <div class="float-end d-none d-sm-inline">{{ project_name }}</div>
    </footer>

</div>{# /.app-wrapper #}

{# AdminLTE JS (includes Bootstrap 5 bundle) #}
<script src="{{ asset('vendor/adminlte/js/adminlte.min.js') }}"></script>
{% block javascripts %}{% endblock %}
</body>
</html>
```

### Global CSS — public/css/app.css

```css
/* app.css — overrides on top of AdminLTE v4 */

.sidebar-menu .nav-link.active {
    background-color: rgba(255, 255, 255, 0.1);
    border-radius: 4px;
}

.content-card {
    border: none;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

/* RTL — table alignment fix */
[dir="rtl"] .table th,
[dir="rtl"] .table td {
    text-align: right;
}

/* RTL — button group direction */
[dir="rtl"] .btn-group {
    direction: rtl;
}
```

---

## Option B — Bootstrap 5

Use this section if `FRONTEND_FRAMEWORK = Bootstrap 5`. No admin layout shell — plain Bootstrap only.

### Install

```bash
npm install bootstrap @fortawesome/fontawesome-free
```

Or use CDN (no build step needed):

```twig
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
{# RTL — swap the above for: #}
{# <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.rtl.min.css"> #}
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
```

### Base Layout

Follow the same structure as Option A but without the AdminLTE wrapper classes — use a standard Bootstrap navbar + container layout. The `dir` and `lang` attributes, locale switcher, and flash message block remain identical.

---

## Option C — Tailwind CSS + DaisyUI

Use this section if `FRONTEND_FRAMEWORK = Tailwind + DaisyUI`. Requires a build step.

### Install

```bash
npm install -D tailwindcss @tailwindcss/vite daisyui
npx tailwindcss init
```

Configure `tailwind.config.js`:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
    content: ["./templates/**/*.{html,twig}", "./assets/**/*.js"],
    plugins: [require("daisyui")],
    daisyui: {
        themes: ["light", "dark"],
    },
};
```

Configure Vite (`vite.config.js`) or Webpack Encore for the build pipeline, then reference compiled assets in `base.html.twig`.

**RTL note:** Tailwind does not ship a separate RTL build. Use the `tailwindcss-rtl` plugin or set `dir` on the `<html>` element and rely on CSS logical properties (`ms-`, `me-`, `ps-`, `pe-` instead of `ml-`, `mr-`, `pl-`, `pr-`). DaisyUI components respect the `dir` attribute natively.

---

## i18n and RTL Frontend Setup

**Only execute this section if `USE_I18N = Yes`.** If not, skip entirely.

### 1. Set lang and dir on the html element

All layout templates must set `lang` and `dir` dynamically based on the active Symfony locale:

```twig
<html lang="{{ app.request.locale }}"
      dir="{{ app.request.locale in ['ar', 'he', 'fa', 'ur'] ? 'rtl' : 'ltr' }}">
```

Extend this list if you add more RTL languages.

### 2. Serve RTL Stylesheets Conditionally

For AdminLTE and Bootstrap, swap the stylesheet based on direction:

```twig
{% set is_rtl = app.request.locale in ['ar', 'he', 'fa', 'ur'] %}

{% if is_rtl %}
    <link rel="stylesheet" href="{{ asset('vendor/adminlte/css/adminlte.rtl.min.css') }}">
{% else %}
    <link rel="stylesheet" href="{{ asset('vendor/adminlte/css/adminlte.min.css') }}">
{% endif %}
```

### 3. Add a Locale Switcher in the Navbar

Include a language dropdown in the top navigation. Add a link for every language defined in `I18N_LANGUAGES`:

```twig
<li class="nav-item dropdown">
    <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
        <i class="fa-solid fa-globe"></i> {{ app.request.locale|upper }}
    </a>
    <ul class="dropdown-menu dropdown-menu-end">
        <li>
            <a class="dropdown-item" href="{{ path('app_switch_locale', {locale: 'en'}) }}">
                English
            </a>
        </li>
        <li>
            <a class="dropdown-item" href="{{ path('app_switch_locale', {locale: 'ar'}) }}">
                العربية
            </a>
        </li>
    </ul>
</li>
```

### 4. Use trans in All User-Facing Strings

Never hard-code UI text. Always use Symfony's translation filter:

```twig
{# Simple key #}
{{ 'nav.dashboard'|trans }}

{# With parameters #}
{{ 'greeting.user'|trans({'%name%': app.user.realName}) }}

{# In attributes #}
<input placeholder="{{ 'form.search_placeholder'|trans }}">
```

### 5. RTL CSS Conventions

Write CSS that works in both directions. Use logical properties instead of physical ones:

| Avoid (physical)   | Use instead (logical) |
| ------------------ | --------------------- |
| `margin-left`      | `margin-inline-start` |
| `padding-right`    | `padding-inline-end`  |
| `float: left`      | `float: inline-start` |
| `text-align: left` | `text-align: start`   |
| `border-left`      | `border-inline-start` |

Bootstrap 5 utility classes already use logical equivalents: `ms-` (margin-start), `me-` (margin-end), `ps-` (padding-start), `pe-` (padding-end). Use these in Twig templates.

### 6. Arabic Typography

Add to `public/css/app.css`:

```css
/* Arabic font stack — loads Google Fonts Noto Sans Arabic for RTL content */
[dir="rtl"] body {
    font-family: "Noto Sans Arabic", "Segoe UI", Tahoma, sans-serif;
    font-size: 1rem;
    line-height: 1.7; /* Arabic text benefits from more line height */
}
```

Optionally load Noto Sans Arabic from Google Fonts (only when locale is Arabic):

```twig
{% if app.request.locale == 'ar' %}
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+Arabic:wght@400;600;700&display=swap"
      rel="stylesheet">
{% endif %}
```

---

## Component Conventions

These conventions apply regardless of which framework is chosen.

### Buttons

```twig
{# Primary submit #}
<button type="submit" class="btn btn-primary">
    <i class="fa-solid fa-floppy-disk"></i> {{ 'action.save'|trans }}
</button>

{# Edit #}
<a href="{{ path('app_edit', {id: item.id}) }}" class="btn btn-sm btn-secondary">
    <i class="fa-solid fa-pen-to-square"></i> {{ 'action.edit'|trans }}
</a>

{# Delete — always a form, never a link #}
<form method="post" action="{{ path('app_delete', {id: item.id}) }}" style="display:inline">
    <input type="hidden" name="_token" value="{{ csrf_token('delete-' ~ item.id) }}">
    <button type="submit" class="btn btn-sm btn-danger"
            onclick="return confirm('{{ 'confirm.delete'|trans }}')">
        <i class="fa-solid fa-trash"></i> {{ 'action.delete'|trans }}
    </button>
</form>
```

### Icon Reference

| Action           | Icon class                       |
| ---------------- | -------------------------------- |
| Edit             | `fa-solid fa-pen-to-square`      |
| Delete           | `fa-solid fa-trash`              |
| View             | `fa-solid fa-eye`                |
| Add              | `fa-solid fa-plus`               |
| Save             | `fa-solid fa-floppy-disk`        |
| Search           | `fa-solid fa-magnifying-glass`   |
| Dashboard        | `fa-solid fa-gauge`              |
| Settings         | `fa-solid fa-gear`               |
| User             | `fa-solid fa-user`               |
| Logout           | `fa-solid fa-right-from-bracket` |
| Globe / Language | `fa-solid fa-globe`              |

---

## Customization Checklist

- [ ] Confirm `FRONTEND_FRAMEWORK` choice and delete unused Option sections above
- [ ] Install chosen framework assets (npm or CDN)
- [ ] Create `templates/base.html.twig` from the appropriate option
- [ ] Create `public/css/app.css`
- [ ] Implement `app_dashboard` route and template
- [ ] Add project-specific sidebar navigation items (AdminLTE) or navbar items (Bootstrap/Tailwind)
- [ ] If `USE_I18N = Yes`: verify `lang` + `dir` on `<html>`, RTL stylesheet swap, locale switcher, `trans` filter on all strings
- [ ] If `USE_RTL = Yes`: test Arabic layout, check table alignment, check form field directions, verify font
- [ ] Add project-specific Twig macros to `templates/macros.html.twig`

Use the `frontend-design` skill if available to generate polished UI components.
