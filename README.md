# Filament Cursor Pagination TypeError - Reproduction Repository

This repository contains a minimal Laravel 12 and Filament installation to demonstrate a `TypeError` that occurs when using cursor pagination (`PaginationMode::Cursor`) in a Filament Table resource.

## The Bug

When using cursor pagination, Filament throws a fatal `TypeError` upon attempting to navigate to the next page. This happens because the page cursor is a **string**, but the `HasTable` contract and the `InteractsWithTable` trait specify an **int** return type for the `getTablePage()` method, causing a type mismatch.

## Prerequisites

- PHP 8.4+
- Composer
- A configured database (SQLite is sufficient)

## Setup and Reproduction Steps

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/Gusta14Dev/filament-bug-report.git
    cd filament-bug-report
    ```

2.  **Install dependencies:**
    ```bash
    composer install
    ```

3.  **Run migrations and seed the database:**
    *This command will create the necessary tables, a test user, and 100 sample records for pagination.*
    ```bash
    php artisan migrate:fresh --seed
    ```

4.  **Start the development server:**
    ```bash
    php artisan serve
    ```

57.  **Reproduce the bug:**
    a. Access the application at `http://127.0.0.1:8000/admin`.
    b. Log in using the credentials below.
    c. Navigate to the "Tests" resource in the sidebar.
    d. **Observe the fatal `TypeError`**.

## Login Credentials

-   **Email:** `test@example.com`
-   **Password:** `password`

## Technical Analysis

The root cause of the error is a type hint mismatch in the Filament Tables package.

-   Cursor pagination generates a non-integer token (a string) to identify the next set of records.
-   The `Filament\Tables\Contracts\HasTable` contract, however, strictly defines the `getTablePage()` method to return an `int`.
-   The `Filament\Tables\Concerns\CanPaginateRecords` trait follows the contract and the `getTablePage()` method to return an `int`.


**File:** `vendor/filament/tables/src/Concerns/CanPaginateRecords.php`
```php
<?php

namespace Filament\Tables\Concerns;

// ...

trait CanPaginateRecords
{
    // ...

    public function getTablePage(): int | string
    {
        return $this->getPage($this->getTablePaginationPageName());
    }

    // ...
}
```

**File:** `vendor/filament/tables/src/Contracts/HasTable.php`
```php
<?php

namespace Filament\Tables\Contracts;

// ...

interface HasTable
{
    // ...

    public function getTablePage(): int;

    // ...
}
```

When the table tries to get the current page with a string cursor, it violates this contract, leading to the TypeError.

## Proposed Solution
To accommodate both integer-based and string-based (cursor) pagination, the method signature should be updated to accept both types.

The following change should be applied to HasTable.php and any corresponding implementations:

```diff
// Em vendor/filament/tables/src/Contracts/HasTable.php

- public function getTablePage(): int;
+ public function getTablePage(): int | string;

```

This ensures type safety while allowing the flexibility required for cursor pagination to function correctly.

## Official Bug Report
This repository was created to support the issue submitted to the main Filament repository. Once the issue is created, the link will be added here for reference.

Link to Issue: [Add the link to your Filament GitHub issue here]