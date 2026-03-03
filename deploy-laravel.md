\# eTech Digital Solutions — Laravel + Inertia.js + React



This is the \*\*Laravel\*\* version of the eTech Digital Solutions website, converted from the original React/Next.js/Vite project.



\## Tech Stack



| Layer        | Technology |

|-------------|------------|

| Backend      | Laravel 12 (PHP 8.2+) |

| Frontend     | React 18 + TypeScript |

| Routing      | Inertia.js v2 (server-driven SPA) |

| Build Tool   | Vite 7 with `@vitejs/plugin-react` |

| Styling      | Tailwind CSS v4 |

| UI Components| Radix UI + shadcn/ui |

| Database     | SQLite (dev) / MySQL or PostgreSQL (production) |

| Auth         | Laravel built-in Auth guards |



\## Project Structure



```

etech-laravel/

├── app/

│   ├── Http/

│   │   ├── Controllers/

│   │   │   ├── PageController.php          # Public page routes → Inertia responses

│   │   │   ├── ContactController.php       # Contact / partnership form handling

│   │   │   └── Admin/

│   │   │       ├── AuthController.php      # Admin login/logout

│   │   │       ├── DashboardController.php

│   │   │       ├── InvoiceController.php

│   │   │       └── ClientController.php

│   │   └── Middleware/

│   │       └── HandleInertiaRequests.php   # Inertia middleware (shared props)

│   └── Models/

│       ├── Client.php

│       ├── Invoice.php

│       └── InvoiceItem.php

│

├── database/

│   ├── migrations/                         # Clients, Invoices, InvoiceItems tables

│   └── database.sqlite                     # Dev SQLite database

│

├── resources/

│   ├── css/app.css                         # Tailwind entry point

│   ├── js/

│   │   ├── app.tsx                         # Inertia React app entry

│   │   ├── Layouts/AppLayout.tsx           # Shared layout (Nav, Footer, Loading)

│   │   ├── Pages/                          # Inertia page components (one per route)

│   │   │   ├── Home.tsx

│   │   │   ├── About.tsx, Contact.tsx, ...

│   │   │   ├── Services/

│   │   │   └── Admin/

│   │   ├── Components/                     # Migrated React components

│   │   ├── app/                            # Migrated client-component wrappers

│   │   ├── contexts/                       # ThemeContext

│   │   ├── hooks/, utils/, data/, types/

│   │   └── compat/router.tsx               # react-router-dom → Inertia shim

│   └── views/app.blade.php                 # Root Blade template

│

├── routes/web.php                          # All application routes

└── vite.config.js

```



\## Getting Started



\### Prerequisites



\- PHP 8.2+

\- Composer 2+

\- Node.js 18+ \& npm



\### 1. Install dependencies



```bash

composer install

npm install

```



\### 2. Configure environment



```bash

cp .env.example .env

php artisan key:generate

```



Edit `.env` to set your database connection (SQLite is configured by default for development).



\### 3. Run migrations



```bash

php artisan migrate

```



\### 4. Build frontend assets



```bash

\# Development (with HMR)

npm run dev



\# Production build

npm run build

```



\### 5. Start the server



```bash

\# In one terminal — Vite dev server (hot reload)

npm run dev



\# In another terminal — Laravel built-in server

php artisan serve

```



Then open http://localhost:8000.



\## Routes



| Method | URL                              | Description               |

|--------|----------------------------------|---------------------------|

| GET    | `/`                              | Home                      |

| GET    | `/about`                         | About                     |

| GET    | `/services`                      | Services listing          |

| GET    | `/services/microsoft-365`        | Microsoft 365 detail      |

| GET    | `/services/hardware-procurement` | Hardware detail           |

| GET    | `/services/consulting`           | Consulting detail         |

| GET    | `/services/{serviceId}`          | Dynamic service detail    |

| GET    | `/process`                       | Our Process               |

| GET    | `/portfolio`                     | Portfolio                 |

| GET    | `/contact`                       | Contact                   |

| POST   | `/contact`                       | Contact form submit       |

| POST   | `/partnership`                   | Partnership form submit   |

| GET    | `/technology`                    | Technology stack          |

| GET    | `/terms`                         | Terms of Service          |

| GET    | `/privacy`                       | Privacy Policy            |

| GET    | `/cookies`                       | Cookie Policy             |

| GET    | `/admin/login`                   | Admin login               |

| GET    | `/admin/dashboard`               | Admin dashboard (auth)    |

| GET    | `/admin/invoices`                | Invoices list (auth)      |

| GET    | `/admin/clients`                 | Clients list (auth)       |



\## Production Deployment



```bash

composer install --no-dev --optimize-autoloader

npm run build

php artisan config:cache

php artisan route:cache

php artisan view:cache

```



Point your web server document root to the `public/` directory.



\## Database



By default, SQLite is used for development. For production, set `DB\_CONNECTION=mysql` (or `pgsql`) and configure the appropriate `DB\_HOST`, `DB\_DATABASE`, `DB\_USERNAME`, `DB\_PASSWORD` in `.env`.



\## Supabase Integration



The original project used Supabase. The Laravel version uses Eloquent ORM with a local database. To re-integrate Supabase, you can use the Supabase REST API via Guzzle HTTP client in the controllers, or switch the `DB\_CONNECTION` to point to the Supabase Postgres database directly.





<p align="center">

<a href="https://github.com/laravel/framework/actions"><img src="https://github.com/laravel/framework/workflows/tests/badge.svg" alt="Build Status"></a>

<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>

<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>

<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>

</p>



\## About Laravel



Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel takes the pain out of development by easing common tasks used in many web projects, such as:



\- \[Simple, fast routing engine](https://laravel.com/docs/routing).

\- \[Powerful dependency injection container](https://laravel.com/docs/container).

\- Multiple back-ends for \[session](https://laravel.com/docs/session) and \[cache](https://laravel.com/docs/cache) storage.

\- Expressive, intuitive \[database ORM](https://laravel.com/docs/eloquent).

\- Database agnostic \[schema migrations](https://laravel.com/docs/migrations).

\- \[Robust background job processing](https://laravel.com/docs/queues).

\- \[Real-time event broadcasting](https://laravel.com/docs/broadcasting).



Laravel is accessible, powerful, and provides tools required for large, robust applications.



\## Learning Laravel



Laravel has the most extensive and thorough \[documentation](https://laravel.com/docs) and video tutorial library of all modern web application frameworks, making it a breeze to get started with the framework. You can also check out \[Laravel Learn](https://laravel.com/learn), where you will be guided through building a modern Laravel application.



If you don't feel like reading, \[Laracasts](https://laracasts.com) can help. Laracasts contains thousands of video tutorials on a range of topics including Laravel, modern PHP, unit testing, and JavaScript. Boost your skills by digging into our comprehensive video library.



\## Laravel Sponsors



We would like to extend our thanks to the following sponsors for funding Laravel development. If you are interested in becoming a sponsor, please visit the \[Laravel Partners program](https://partners.laravel.com).



\### Premium Partners



\- \*\*\[Vehikl](https://vehikl.com)\*\*

\- \*\*\[Tighten Co.](https://tighten.co)\*\*

\- \*\*\[Kirschbaum Development Group](https://kirschbaumdevelopment.com)\*\*

\- \*\*\[64 Robots](https://64robots.com)\*\*

\- \*\*\[Curotec](https://www.curotec.com/services/technologies/laravel)\*\*

\- \*\*\[DevSquad](https://devsquad.com/hire-laravel-developers)\*\*

\- \*\*\[Redberry](https://redberry.international/laravel-development)\*\*

\- \*\*\[Active Logic](https://activelogic.com)\*\*



\## Contributing



Thank you for considering contributing to the Laravel framework! The contribution guide can be found in the \[Laravel documentation](https://laravel.com/docs/contributions).



\## Code of Conduct



In order to ensure that the Laravel community is welcoming to all, please review and abide by the \[Code of Conduct](https://laravel.com/docs/contributions#code-of-conduct).



\## Security Vulnerabilities



If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell via \[taylor@laravel.com](mailto:taylor@laravel.com). All security vulnerabilities will be promptly addressed.



\## License



The Laravel framework is open-sourced software licensed under the \[MIT license](https://opensource.org/licenses/MIT).



