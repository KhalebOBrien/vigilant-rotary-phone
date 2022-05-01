# vigilant-rotary-phone

The following are best practices that you should follow while working with Laravel 9.

### **1. Keep Business Logic in Service Class**

Controllers are not meant to handle business logic, so keep them clean by putting logics in service classes.

Do these:

```php
public function store(Request $request)
{
    $this->DownlineService->shareCommissionToDownlines($user, $commission);

    ....
}

class DownlineService
{
    public function shareCommissionToDownlines($user, $commission): void
    {
        if (!empty($user), !is_null($commission)) {
            # code to share commission to downlines
        }
    }
}
```

### **2. Use Helper Functions**

Don't reinvent the wheel, use the helper methods provided in Illuminate\Support\Str

Don't do these:

```php
public function newId()
{
    $str_result = '23456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghjklmnpqrstuvwxyz';

    $id = substr(str_shuffle($str_result), 0, 24); 
}
```

Do these:

```php
public function newId()
{
    ....
    $id = Str::random(24);
    ....
}

```

### **3. Obey Single Responsibility Principle (SRP)**

A class and a method should have one and only one responsibility.

Don't do these:

```php
public function getTransactionAttribute()
{
    if ($transaction && ($transaction->type == 'withdrawal') && $transaction->isVerified()) {
        return ['reference'=>$this->transaction->reference, 'status'=>'verified'];
    } else {
        return ['link'=>$this->transaction->paymentLink, 'status'=>'not verified'];
    }
}
```

Do these:

```php
public function getTransactionAttribute(): bool
{
    return $this->isVerified() ? $this->getReference() : $this->getPaymentLink();
}

public function isVerified(): bool
{
    return $this->transaction && ($transaction->type == 'withdrawal') && $this->transaction->isVerified();
}

public function getReference(): string
{
    return ['reference'=>$this->transaction->reference, 'status'=>'verified'];
}

public function getPaymentLink(): string
{
    return ['link'=>$this->transaction->paymentLink, 'status'=>'not verified'];
}
```
### **4. Do Validation in Request Classes**

All form validations should be done in Request classes.

Don't do these:

```php
public function store(Request $request)
{
    $request->validate([
        'slug' => 'required',
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'flImage' => ['image', 'required', 'mimes:png,jpg,jpeg,gif']
    ]);
}
```

Do these:

```php
public function store(CreateArticleRequest $request)
{    
    ....
}

class CreateArticleRequest extends Request
{
    public function rules(): array
    {
        return [
            'slug' => 'required',
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'flImage' => ['image', 'required', 'mimes:png,jpg,jpeg,gif']
        ];
    }
}
```

### **5. Timeout HTTP Requests**

Prevent timeout errors by using the `timeout` method on the http client. By default, Laravel will now timeout after 30 seconds.

Good example:

```php
public function makeRequest(CreateArticleRequest $request)
{
    ....
    $response = Http::timeout(120)->get(...);
    ....
}
```

### **6. Prefer Mass assignment**
Mass assignments helps prevent Mass Assignment Vulnerabilities. 

Don't do these:

```php
$product = new Product;
$product->name = $request->name;
$product->price = $request->price;
$product->qty = $request->qty;
$product->save();
```

Do this:

```php
Product::create($request->validated());
```

### **7. Chunk data for heavy data tasks**

Don't do these:

```php
$articles = $this->get();

foreach ($articles as $article) {
    ...
}
```

Do this:

```php
$this->chunk(100, function ($articles) {
    foreach ($articles as $article) {
        ...
    }
});
```

### **8. Maintain Laravel naming conventions**

Follow PSR standards as stated in http://www.php-fig.org/psr/psr-2/

Also, use naming conventions accepted by the Laravel community:

What | How | Good | Bad
------------ | ------------- | ------------- | -------------
Model | singular | User | ~~Users~~
hasOne or belongsTo relationship | singular | articleComment | ~~articleComments, article_comment~~
All other relationships | plural | articleComments | ~~articleComment, article_comments~~
Table | plural | article_comments | ~~article_comment, articleComments~~
Route | plural | articles/1 | ~~article/1~~
Pivot table | singular model names in alphabetical order | article_user | ~~user_article, articles_users~~
Table column | snake_case without model name | meta_title | ~~MetaTitle; article_meta_title~~
Model property | snake_case | $model->created_at | ~~$model->createdAt~~
Controller | singular | ArticleController | ~~ArticlesController~~
Contract (interface) | adjective or noun | AuthenticationInterface | ~~Authenticatable, IAuthentication~~
Trait | adjective | Notifiable | ~~NotificationTrait~~
Foreign key | singular model name with _id suffix | article_id | ~~ArticleId, id_article, articles_id~~
Method | camelCase | getAll | ~~get_all~~
Variable | camelCase | $articlesWithAuthor | ~~$articles_with_author~~
View | kebab-case | show-filtered.blade.php | ~~showFiltered.blade.php, show_filtered.blade.php~~
Collection | descriptive, plural | $activeUsers = User::active()->get() | ~~$active, $data~~
Named route | snake_case with dot notation | users.show_active | ~~users.show-active, show-active-users~~
Object | descriptive, singular | $activeUser = User::active()->first() | ~~$users, $obj~~
Config and language files index | snake_case | articles_enabled | ~~ArticlesEnabled; articles-enabled~~
Config | snake_case | google_calendar.php | ~~googleCalendar.php, google-calendar.php~~

### **9. Do not use environment variables (`.env`) directly in your code**

Environment variables should only be used in `config()` helper function. Use `config()` to access `.env` variables.

Don't do these:

```php
$apiKey = env('API_KEY');
```

Do these:

```php
// config/api.php
'key' => env('API_KEY'),

// Use the data
$apiKey = config('api.key');
```

### **10. Use Eloquent instead of Query Builder and Raw SQL Queries**

Eloquent allows you to write readable and maintainable code. Also, Eloquent has great built-in tools like soft deletes, events, scopes etc. It is also easier to manage relationships between models.

Don't do these:

```sql
SELECT *
FROM `products`
WHERE EXISTS (SELECT *
              FROM `categories`
              WHERE `products`.`category_id` = `categories`.`id`
              AND `categories`.`deleted_at` IS NULL)
AND `is_available` = '1'
ORDER BY `created_at` DESC
```

Do these:

```php
Product::has('category')->isAvailable()->latest()->get();
```
