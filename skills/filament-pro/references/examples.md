# Filament v5 Code Examples

Complete working code examples for Filament v5 components.

## Panel Provider

### Basic Configuration

```php
<?php

namespace App\Providers\Filament;

use Filament\Panel;
use Filament\PanelProvider;
use Filament\Support\Colors\Color;

class AdminPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            ->id('admin')
            ->path('admin')
            ->colors(['primary' => Color::Amber])
            ->discoverResources(in: app_path('Filament/Resources'), for: 'App\Filament\Resources')
            ->discoverPages(in: app_path('Filament/Pages'), for: 'App\Filament\Pages')
            ->discoverWidgets(in: app_path('Filament/Widgets'), for: 'App\Filament\Widgets')
            ->middleware([
                \Illuminate\Cookie\Middleware\EncryptCookies::class,
                \Illuminate\Session\Middleware\StartSession::class,
            ])
            ->authMiddleware([
                \Illuminate\Auth\Middleware\Authenticate::class,
            ]);
    }
}
```

### With Branding

```php
public function panel(Panel $panel): Panel
{
    return $panel
        ->id('admin')
        ->path('admin')
        ->brandName('My Admin')
        ->brandLogo(asset('images/logo.svg'))
        ->favicon(asset('images/favicon.ico'))
        ->colors([
            'primary' => '#f59e0b',
            'secondary' => '#64748b',
            'success' => '#22c55e',
            'warning' => '#f59e0b',
            'danger' => '#ef4444',
        ]);
}
```

## Resources

### Complete Post Resource

```php
<?php

namespace App\Filament\Resources;

use App\Filament\Resources\PostResource\Pages;
use App\Models\Post;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;

class PostResource extends Resource
{
    protected static ?string $model = Post::class;
    protected static ?string $navigationIcon = 'heroicon-o-document-text';
    protected static ?string $navigationGroup = 'Content';

    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\TextInput::make('title')
                    ->required()
                    ->maxLength(255)
                    ->live(onBlur: true)
                    ->afterStateUpdated(fn ($state, $set) => 
                        $set('slug', \Illuminate\Support\Str::slug($state))
                    ),
                
                Forms\Components\TextInput::make('slug')
                    ->required()
                    ->unique(ignoreRecord: true)
                    ->maxLength(255),
                
                Forms\Components\RichEditor::make('content')
                    ->required()
                    ->columnSpanFull(),
                
                Forms\Components\Select::make('status')
                    ->options([
                        'draft' => 'Draft',
                        'published' => 'Published',
                        'archived' => 'Archived',
                    ])
                    ->required(),
                
                Forms\Components\DateTimePicker::make('published_at'),
                
                Forms\Components\Toggle::make('is_featured'),
                
                Forms\Components\Select::make('author_id')
                    ->relationship('author', 'name')
                    ->searchable()
                    ->preload()
                    ->required(),
                
                Forms\Components\Select::make('categories')
                    ->relationship('categories', 'name')
                    ->multiple()
                    ->preload()
                    ->searchable(),
            ])
            ->columns(2);
    }

    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('title')
                    ->searchable()
                    ->sortable(),
                
                Tables\Columns\TextColumn::make('status')
                    ->badge()
                    ->color(fn (string $state): string => match ($state) {
                        'draft' => 'gray',
                        'published' => 'success',
                        'archived' => 'warning',
                    }),
                
                Tables\Columns\TextColumn::make('author.name')
                    ->searchable()
                    ->sortable(),
                
                Tables\Columns\IconColumn::make('is_featured')
                    ->boolean(),
                
                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable(),
            ])
            ->filters([
                Tables\Filters\SelectFilter::make('status')
                    ->options([
                        'draft' => 'Draft',
                        'published' => 'Published',
                        'archived' => 'Archived',
                    ]),
                Tables\Filters\TernaryFilter::make('is_featured'),
            ])
            ->actions([
                Tables\Actions\ViewAction::make(),
                Tables\Actions\EditAction::make(),
                Tables\Actions\DeleteAction::make(),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                ]),
            ]);
    }

    public static function getPages(): array
    {
        return [
            'index' => Pages\ListPosts::class,
            'create' => Pages\CreatePost::class,
            'view' => Pages\ViewPost::class,
            'edit' => Pages\EditPost::class,
        ];
    }
}
```

## Forms

### User Registration Form

```php
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\Select;
use Filament\Forms\Components\DatePicker;
use Filament\Forms\Components\FileUpload;
use Filament\Forms\Components\Toggle;
use Filament\Forms\Components\Section;
use Filament\Forms\Components\Grid;

public static function form(Form $form): Form
{
    return $form
        ->schema([
            Section::make('Personal Information')
                ->schema([
                    TextInput::make('first_name')
                        ->required()
                        ->maxLength(255),
                    TextInput::make('last_name')
                        ->required()
                        ->maxLength(255),
                    TextInput::make('email')
                        ->email()
                        ->required()
                        ->unique(ignoreRecord: true),
                    DatePicker::make('birthdate')
                        ->maxDate(now()->subYears(18)),
                ])
                ->columns(2),
            
            Section::make('Account Settings')
                ->schema([
                    TextInput::make('password')
                        ->password()
                        ->required()
                        ->minLength(8)
                        ->confirmed(),
                    TextInput::make('password_confirmation')
                        ->password()
                        ->required(),
                    Select::make('role')
                        ->options([
                            'admin' => 'Administrator',
                            'editor' => 'Editor',
                            'user' => 'User',
                        ])
                        ->required(),
                    Toggle::make('is_active')
                        ->default(true),
                ])
                ->columns(2),
            
            Section::make('Profile')
                ->schema([
                    FileUpload::make('avatar')
                        ->image()
                        ->circleCropper()
                        ->maxSize(5120),
                ]),
        ]);
}
```

### Product Form with Repeater

```php
use Filament\Forms\Components\Repeater;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\Select;

public static function form(Form $form): Form
{
    return $form
        ->schema([
            TextInput::make('name')
                ->required(),
            
            Repeater::make('variants')
                ->schema([
                    Select::make('size')
                        ->options(['S' => 'Small', 'M' => 'Medium', 'L' => 'Large'])
                        ->required(),
                    TextInput::make('sku')
                        ->required(),
                    TextInput::make('price')
                        ->numeric()
                        ->prefix('$')
                        ->required(),
                    TextInput::make('stock')
                        ->numeric()
                        ->required(),
                ])
                ->columns(4)
                ->defaultItems(1)
                ->addActionLabel('Add Variant')
                ->reorderable(),
        ]);
}
```

### Settings Form with Tabs

```php
use Filament\Forms\Components\Tabs;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\Textarea;

public static function form(Form $form): Form
{
    return $form
        ->schema([
            Tabs::make('Settings')
                ->tabs([
                    Tabs\Tab::make('General')
                        ->schema([
                            TextInput::make('site_name')
                                ->required(),
                            TextInput::make('site_email')
                                ->email()
                                ->required(),
                        ]),
                    Tabs\Tab::make('SEO')
                        ->schema([
                            TextInput::make('meta_title'),
                            Textarea::make('meta_description')
                                ->maxLength(160),
                        ]),
                    Tabs\Tab::make('Social')
                        ->schema([
                            TextInput::make('facebook_url'),
                            TextInput::make('twitter_url'),
                        ]),
                ]),
        ]);
}
```

## Tables

### Product Catalog Table

```php
public static function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\ImageColumn::make('image')
                ->square()
                ->size(50),
            
            Tables\Columns\TextColumn::make('name')
                ->searchable()
                ->sortable(),
            
            Tables\Columns\TextColumn::make('category.name')
                ->searchable()
                ->sortable(),
            
            Tables\Columns\TextColumn::make('price')
                ->money('USD')
                ->sortable(),
            
            Tables\Columns\TextColumn::make('stock_quantity')
                ->numeric()
                ->sortable()
                ->color(fn (int $state): string => match (true) {
                    $state <= 0 => 'danger',
                    $state <= 10 => 'warning',
                    default => 'success',
                }),
            
            Tables\Columns\IconColumn::make('is_active')
                ->boolean(),
        ])
        ->filters([
            Tables\Filters\SelectFilter::make('category')
                ->relationship('category', 'name'),
            
            Tables\Filters\Filter::make('low_stock')
                ->query(fn ($query) => $query->where('stock_quantity', '<=', 10))
                ->toggle(),
        ])
        ->actions([
            Tables\Actions\EditAction::make(),
            Tables\Actions\DeleteAction::make(),
        ])
        ->bulkActions([
            Tables\Actions\BulkActionGroup::make([
                Tables\Actions\DeleteBulkAction::make(),
            ]),
        ]);
}
```

## Actions

### Email Action with Modal

```php
use Filament\Actions\Action;
use Filament\Notifications\Notification;

Action::make('sendEmail')
    ->icon('heroicon-m-envelope')
    ->form([
        TextInput::make('subject')
            ->required()
            ->maxLength(255),
        RichEditor::make('body')
            ->required()
            ->columnSpanFull(),
    ])
    ->action(function (array $data, $record): void {
        Mail::to($record->email)
            ->send(new GenericEmail($data['subject'], $data['body']));
        
        Notification::make()
            ->title('Email sent successfully')
            ->success()
            ->send();
    })
    ->modalWidth(MaxWidth::Large);
```

### Delete with Confirmation

```php
Action::make('delete')
    ->color('danger')
    ->icon('heroicon-m-trash')
    ->requiresConfirmation()
    ->modalHeading('Delete record')
    ->modalDescription('Are you sure? This cannot be undone.')
    ->action(fn ($record) => $record->delete());
```

## Widgets

### Stats Overview

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Order;
use App\Models\User;
use Filament\Widgets\StatsOverviewWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class DashboardStats extends StatsOverviewWidget
{
    protected function getStats(): array
    {
        return [
            Stat::make('Total Users', User::count())
                ->description(User::where('created_at', '>=', now()->subDays(30))->count() . ' new this month')
                ->color('success'),
            
            Stat::make('Revenue', '$' . number_format(Order::sum('total'), 2))
                ->description('Total sales')
                ->color('primary'),
            
            Stat::make('Pending Orders', Order::where('status', 'pending')->count())
                ->color('warning'),
        ];
    }
}
```

### Chart Widget

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Order;
use Filament\Widgets\ChartWidget;

class OrdersChart extends ChartWidget
{
    protected ?string $heading = 'Orders per Month';
    
    protected function getData(): array
    {
        $orders = Order::query()
            ->selectRaw('MONTH(created_at) as month, COUNT(*) as count')
            ->whereYear('created_at', now()->year)
            ->groupBy('month')
            ->pluck('count', 'month')
            ->toArray();

        return [
            'datasets' => [
                [
                    'label' => 'Orders',
                    'data' => array_values($orders),
                ],
            ],
            'labels' => ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 
                        'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
        ];
    }

    protected function getType(): string
    {
        return 'line';
    }
}
```

## Testing

### Resource Tests

```php
<?php

use App\Filament\Resources\PostResource\Pages\CreatePost;
use App\Filament\Resources\PostResource\Pages\EditPost;
use App\Filament\Resources\PostResource\Pages\ListPosts;
use App\Models\Post;
use App\Models\User;

use function Pest\Laravel\actingAs;
use function Pest\Livewire\livewire;

beforeEach(function () {
    $this->user = User::factory()->create();
    actingAs($this->user);
});

// List Page
it('can list posts', function () {
    $posts = Post::factory()->count(5)->create();
    
    livewire(ListPosts::class)
        ->assertCanSeeTableRecords($posts);
});

it('can search posts', function () {
    Post::factory()->create(['title' => 'Hello World']);
    
    livewire(ListPosts::class)
        ->searchTable('Hello')
        ->assertCanSeeTableRecords(Post::where('title', 'like', '%Hello%')->get());
});

// Create Page
it('can create a post', function () {
    $newData = Post::factory()->make();
    
    livewire(CreatePost::class)
        ->fillForm([
            'title' => $newData->title,
            'content' => $newData->content,
        ])
        ->call('create')
        ->assertNotified()
        ->assertRedirect();
});

it('validates required fields', function () {
    livewire(CreatePost::class)
        ->fillForm(['title' => null])
        ->call('create')
        ->assertHasFormErrors(['title']);
});

// Edit Page
it('can update a post', function () {
    $post = Post::factory()->create();
    $newData = Post::factory()->make();
    
    livewire(EditPost::class, ['record' => $post->id])
        ->fillForm(['title' => $newData->title])
        ->call('save')
        ->assertNotified();
});
```

## Authorization

### User Model with Panel Access

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Panel;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser
{
    public function canAccessPanel(Panel $panel): bool
    {
        return match ($panel->getId()) {
            'admin' => $this->isAdmin(),
            'app' => $this->hasVerifiedEmail(),
            default => false,
        };
    }
}
```

### Policy

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function viewAny(User $user): bool
    {
        return true;
    }

    public function view(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->isAdmin();
    }

    public function create(User $user): bool
    {
        return $user->can('create posts');
    }

    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->isAdmin();
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->isAdmin();
    }
}
```

### Resource with Authorization

```php
class PostResource extends Resource
{
    public static function canCreate(): bool
    {
        return auth()->user()->can('create posts');
    }
    
    public static function canEdit($record): bool
    {
        return auth()->user()->can('update', $record);
    }
    
    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                TextInput::make('title')->required(),
                
                // Only visible to admins
                TextInput::make('internal_notes')
                    ->visible(fn (): bool => auth()->user()->isAdmin()),
            ]);
    }
}
```

## Multi-Tenancy

### User with Tenancy

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\HasTenants;
use Filament\Panel;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Support\Collection;

class User extends Authenticatable implements HasTenants
{
    public function teams(): BelongsToMany
    {
        return $this->belongsToMany(Team::class);
    }
    
    public function getTenants(Panel $panel): Collection
    {
        return $this->teams;
    }
    
    public function canAccessTenant(Model $tenant): bool
    {
        return $this->teams()->whereKey($tenant)->exists();
    }
}
```

### Panel with Tenancy

```php
public function panel(Panel $panel): Panel
{
    return $panel
        ->tenant(Team::class)
        ->tenantProfile(\App\Filament\Pages\TeamProfile::class)
        ->tenantRegistration(\App\Filament\Pages\TeamRegistration::class);
}
```

## Custom Pages

### Settings Page

```php
<?php

namespace App\Filament\Pages;

use Filament\Actions\Action;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Form;
use Filament\Notifications\Notification;
use Filament\Pages\Page;

class Settings extends Page
{
    protected static ?string $navigationIcon = 'heroicon-o-cog';
    protected static ?string $navigationGroup = 'System';
    protected static string $view = 'filament.pages.settings';
    
    public ?array $data = [];
    
    public function mount(): void
    {
        $this->form->fill([
            'site_name' => config('app.name'),
        ]);
    }
    
    public static function canAccess(): bool
    {
        return auth()->user()->isAdmin();
    }
    
    public function form(Form $form): Form
    {
        return $form
            ->schema([
                TextInput::make('site_name')->required(),
            ])
            ->statePath('data');
    }
    
    protected function getHeaderActions(): array
    {
        return [
            Action::make('save')
                ->action('save'),
        ];
    }
    
    public function save(): void
    {
        $data = $this->form->getState();
        setting(['site_name' => $data['site_name']]);
        
        Notification::make()
            ->title('Settings saved')
            ->success()
            ->send();
    }
}
```

## Relation Managers

### Comments Relation Manager

```php
<?php

namespace App\Filament\Resources\PostResource\RelationManagers;

use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\RelationManagers\RelationManager;
use Filament\Tables;
use Filament\Tables\Table;

class CommentsRelationManager extends RelationManager
{
    protected static string $relationship = 'comments';
    
    public function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\Textarea::make('content')
                    ->required()
                    ->columnSpanFull(),
            ]);
    }
    
    public function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('content')->limit(50),
                Tables\Columns\TextColumn::make('user.name'),
            ])
            ->headerActions([
                Tables\Actions\CreateAction::make(),
            ])
            ->actions([
                Tables\Actions\EditAction::make(),
                Tables\Actions\DeleteAction::make(),
            ]);
    }
}
```

## Complete Blog Example

### Category Resource

```php
<?php

namespace App\Filament\Resources;

use App\Models\Category;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;
use Illuminate\Support\Str;

class CategoryResource extends Resource
{
    protected static ?string $model = Category::class;
    protected static ?string $navigationIcon = 'heroicon-o-tag';
    protected static ?string $navigationGroup = 'Blog';
    
    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\TextInput::make('name')
                    ->required()
                    ->live(onBlur: true)
                    ->afterStateUpdated(fn ($state, $set) => 
                        $set('slug', Str::slug($state))
                    ),
                
                Forms\Components\TextInput::make('slug')
                    ->required()
                    ->unique(ignoreRecord: true),
                
                Forms\Components\ColorPicker::make('color'),
            ]);
    }
    
    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\ColorColumn::make('color'),
                Tables\Columns\TextColumn::make('name')->searchable(),
                Tables\Columns\TextColumn::make('posts_count')->counts('posts'),
            ]);
    }
}
```

### Post Resource with Relations

```php
<?php

namespace App\Filament\Resources;

use App\Models\Post;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;

class PostResource extends Resource
{
    protected static ?string $model = Post::class;
    protected static ?string $navigationIcon = 'heroicon-o-document-text';
    protected static ?string $navigationGroup = 'Blog';
    
    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\TextInput::make('title')->required(),
                
                Forms\Components\RichEditor::make('content')
                    ->required()
                    ->columnSpanFull(),
                
                Forms\Components\Select::make('category_id')
                    ->relationship('category', 'name')
                    ->searchable(),
                
                Forms\Components\Select::make('tags')
                    ->relationship('tags', 'name')
                    ->multiple()
                    ->preload(),
                
                Forms\Components\Toggle::make('is_published'),
            ])
            ->columns(2);
    }
    
    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('title')->searchable(),
                Tables\Columns\TextColumn::make('category.name'),
                Tables\Columns\IconColumn::make('is_published')->boolean(),
            ]);
    }
    
    public static function getRelations(): array
    {
        return [
            RelationManagers\CommentsRelationManager::class,
        ];
    }
}
```
