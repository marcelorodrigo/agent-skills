# Authorization Reference

Complete guide for access control, policies, and multi-tenancy in Filament v5.

## Panel Access Control

### FilamentUser Contract

Implement the `FilamentUser` contract on your User model to control panel access:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Panel;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser
{
    // Simple check
    public function canAccessPanel(Panel $panel): bool
    {
        return $this->hasVerifiedEmail();
    }
    
    // Panel-specific checks
    public function canAccessPanel(Panel $panel): bool
    {
        return match ($panel->getId()) {
            'admin' => $this->isAdmin() && $this->hasVerifiedEmail(),
            'app' => $this->hasVerifiedEmail(),
            'vendor' => $this->isVendor() && $this->hasVerifiedEmail(),
            default => false,
        };
    }
    
    // Domain-based access
    public function canAccessPanel(Panel $panel): bool
    {
        return str_ends_with($this->email, '@company.com') && 
               $this->hasVerifiedEmail();
    }
}
```

## Laravel Policies

Filament automatically uses Laravel policies for authorization.

### Creating Policies

```bash
php artisan make:policy PostPolicy --model=Post
```

### Policy Structure

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

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

    public function update(User $user, Post $post): bool|Response
    {
        if ($post->isLocked()) {
            return Response::deny('This post is locked and cannot be edited.');
        }

        return $user->id === $post->user_id || $user->isAdmin();
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->isAdmin() || 
               ($user->id === $post->user_id && $post->status === 'draft');
    }

    public function deleteAny(User $user): bool
    {
        return $user->isAdmin();
    }

    public function restore(User $user, Post $post): bool
    {
        return $user->isAdmin();
    }

    public function forceDelete(User $user, Post $post): bool
    {
        return $user->isAdmin();
    }
    
    // Custom policy methods
    public function publish(User $user, Post $post): bool
    {
        return $user->can('publish posts') && 
               $post->status === 'draft';
    }
    
    public function feature(User $user, Post $post): bool
    {
        return $user->isAdmin() || $user->isEditor();
    }
}
```

### Registering Policies

```php
<?php

namespace App\Providers;

use App\Models\Post;
use App\Policies\PostPolicy;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    public function boot(): void
    {
        //
    }
}
```

## Resource Authorization

### Automatic Policy Enforcement

By default, Filament checks policies automatically:

```php
class PostResource extends Resource
{
    protected static bool $shouldSkipAuthorization = false; // Default
}
```

### Manual Authorization Methods

```php
class PostResource extends Resource
{
    public static function canViewAny(): bool
    {
        return auth()->user()->can('viewAny', Post::class);
    }

    public static function canCreate(): bool
    {
        return auth()->user()->can('create', Post::class);
    }

    public static function canEdit(Model $record): bool
    {
        return auth()->user()->can('update', $record);
    }

    public static function canDelete(Model $record): bool
    {
        return auth()->user()->can('delete', $record);
    }

    public static function canView(Model $record): bool
    {
        return auth()->user()->can('view', $record);
    }

    public static function canRestore(Model $record): bool
    {
        return auth()->user()->can('restore', $record);
    }

    public static function canForceDelete(Model $record): bool
    {
        return auth()->user()->can('forceDelete', $record);
    }
}
```

### Eloquent Query Scoping

```php
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()
        ->where(function (Builder $query) {
            if (! auth()->user()->isAdmin()) {
                $query->where('user_id', auth()->id());
            }
        });
}
```

## Page Authorization

### Custom Pages

```php
<?php

namespace App\Filament\Pages;

use Filament\Pages\Page;

class Reports extends Page
{
    protected static ?string $navigationIcon = 'heroicon-o-chart-bar';
    protected static string $view = 'filament.pages.reports';

    public static function canAccess(): bool
    {
        return auth()->user()->can('view reports');
    }
    
    // Or with dependencies
    public static function canAccess(array $parameters = []): bool
    {
        return auth()->user()->can('view reports') &&
               $parameters['type'] ?? false;
    }
}
```

### Settings Pages

```php
<?php

namespace App\Filament\Pages;

use Filament\Pages\Page;

class Settings extends Page
{
    protected static ?string $navigationIcon = 'heroicon-o-cog';
    protected static string $view = 'filament.pages.settings';

    public static function canAccess(): bool
    {
        return auth()->user()->isAdmin();
    }
}
```

## Action Authorization

### Table Actions

```php
->actions([
    Tables\Actions\EditAction::make()
        ->authorize('update'),
    
    Tables\Actions\DeleteAction::make()
        ->authorize('delete'),
    
    Tables\Actions\Action::make('publish')
        ->icon('heroicon-m-check-circle')
        ->authorize('publish')  // Calls PostPolicy::publish()
        ->visible(fn (Post $record): bool => 
            auth()->user()->can('publish', $record)
        )
        ->action(fn (Post $record) => $record->update(['status' => 'published'])),
])
```

### Page Actions

```php
protected function getHeaderActions(): array
{
    return [
        Action::make('export')
            ->icon('heroicon-m-arrow-down-tray')
            ->authorize('export posts')
            ->action(function () {
                // Export logic
            }),
    ];
}
```

## Field-Level Authorization

### Form Fields

```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            TextInput::make('title')
                ->required(),
            
            // Only visible to admins
            TextInput::make('internal_notes')
                ->visible(fn (): bool => auth()->user()->isAdmin()),
            
            // Only visible to users with permission
            TextInput::make('reviewer_notes')
                ->visible(fn (): bool => auth()->user()->can('review posts')),
            
            // Disabled for non-editors
            Select::make('status')
                ->disabled(fn (): bool => !auth()->user()->can('publish posts')),
            
            // Dehydrated (not saved) for certain roles
            TextInput::make('debug_info')
                ->dehydrated(fn (): bool => auth()->user()->isDeveloper()),
        ]);
}
```

### Table Columns

```php
public static function table(Table $table): Table
{
    return $table
        ->columns([
            TextColumn::make('title'),
            
            // Only visible to admins
            TextColumn::make('internal_notes')
                ->visible(fn (): bool => auth()->user()->isAdmin()),
            
            // Toggleable but hidden by default for non-admins
            TextColumn::make('cost')
                ->money('USD')
                ->toggleable(
                    isToggledHiddenByDefault: !auth()->user()->isAdmin()
                ),
        ]);
}
```

## Multi-Tenancy

### Tenant Setup

```php
<?php

namespace App\Providers\Filament;

use App\Models\Team;
use Filament\Panel;
use Filament\PanelProvider;

class AdminPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            // ...
            ->tenant(Team::class)
            ->tenantRegistration(\App\Filament\Pages\Tenancy\RegisterTeam::class)
            ->tenantProfile(\App\Filament\Pages\Tenancy\EditTeamProfile::class)
            ->tenantMenuItems([
                \Filament\Actions\Action::make('settings')
                    ->url(fn (): string => TeamSettings::getUrl())
                    ->icon('heroicon-m-cog-8-tooth'),
            ]);
    }
}
```

### User Model with Tenancy

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Models\Contracts\HasDefaultTenant;
use Filament\Models\Contracts\HasTenants;
use Filament\Panel;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Support\Collection;

class User extends Authenticatable implements FilamentUser, HasTenants, HasDefaultTenant
{
    public function teams(): BelongsToMany
    {
        return $this->belongsToMany(Team::class)
            ->withTimestamps()
            ->withPivot('role');
    }

    public function getTenants(Panel $panel): Collection
    {
        return $this->teams;
    }

    public function canAccessTenant(Model $tenant): bool
    {
        return $this->teams()
            ->whereKey($tenant->getKey())
            ->exists();
    }

    public function getDefaultTenant(Panel $panel): ?Model
    {
        return $this->latestTeam;
    }

    public function latestTeam()
    {
        return $this->belongsTo(Team::class, 'latest_team_id');
    }
}
```

### Tenant Registration Page

```php
<?php

namespace App\Filament\Pages\Tenancy;

use App\Models\Team;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Form;
use Filament\Pages\Tenancy\RegisterTenant;
use Filament\Schemas\Schema;

class RegisterTeam extends RegisterTenant
{
    public static function getLabel(): string
    {
        return 'Register team';
    }

    public function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('name')
                    ->required()
                    ->maxLength(255),
                TextInput::make('slug')
                    ->required()
                    ->unique('teams', 'slug')
                    ->maxLength(255),
            ]);
    }

    protected function handleRegistration(array $data): Team
    {
        $team = Team::create($data);
        $team->members()->attach(auth()->user(), ['role' => 'owner']);
        return $team;
    }
}
```

### Tenant Profile Page

```php
<?php

namespace App\Filament\Pages\Tenancy;

use Filament\Forms\Components\TextInput;
use Filament\Forms\Form;
use Filament\Pages\Tenancy\EditTenantProfile;
use Filament\Schemas\Schema;

class EditTeamProfile extends EditTenantProfile
{
    public static function getLabel(): string
    {
        return 'Team profile';
    }

    public function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('name')
                    ->required()
                    ->maxLength(255),
                TextInput::make('slug')
                    ->required()
                    ->unique('teams', 'slug', ignoreRecord: true)
                    ->maxLength(255),
            ]);
    }
}
```

### Tenant Subdomain Routing

```php
public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenant(Team::class, slugAttribute: 'slug')
        ->tenantDomain('{tenant:slug}.example.com');
}
```

### Tenant Path Routing

```php
public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->tenant(Team::class, slugAttribute: 'slug')
        ->tenantRoutePrefix('{tenant:slug}');
}
```

### Tenant-Aware Resources

```php
class PostResource extends Resource
{
    public static function getEloquentQuery(): Builder
    {
        return parent::getEloquentQuery()
            ->whereBelongsTo(Filament::getTenant());
    }
}
```

Or use a global scope:

```php
class Post extends Model
{
    protected static function booted(): void
    {
        static::addGlobalScope('tenant', function (Builder $query) {
            if (auth()->hasUser() && Filament::getTenant()) {
                $query->whereBelongsTo(Filament::getTenant());
            }
        });
        
        static::creating(function (Post $post) {
            if (Filament::getTenant()) {
                $post->team()->associate(Filament::getTenant());
            }
        });
    }
}
```

## Role-Based Access Control

### Using Spatie Laravel-Permission

```bash
composer require spatie/laravel-permission
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

### User Model

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Panel;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable implements FilamentUser
{
    use HasRoles;

    public function canAccessPanel(Panel $panel): bool
    {
        return $this->hasAnyRole(['admin', 'editor', 'author']);
    }
    
    public function isAdmin(): bool
    {
        return $this->hasRole('admin');
    }
    
    public function isEditor(): bool
    {
        return $this->hasRole('editor');
    }
}
```

### Resource with Role Checks

```php
class PostResource extends Resource
{
    public static function canCreate(): bool
    {
        return auth()->user()->hasAnyRole(['admin', 'editor', 'author']);
    }

    public static function canEdit(Model $record): bool
    {
        $user = auth()->user();
        
        if ($user->hasRole('admin')) {
            return true;
        }
        
        if ($user->hasRole('editor')) {
            return true;
        }
        
        return $user->id === $record->user_id;
    }

    public static function canDelete(Model $record): bool
    {
        return auth()->user()->hasRole('admin');
    }
}
```

### Permission-Based Checks

```php
public static function canPublish(): bool
{
    return auth()->user()->can('publish posts');
}

public static function canFeature(): bool
{
    return auth()->user()->can('feature posts');
}
```

## Navigation Visibility

### Resource Navigation

```php
class PostResource extends Resource
{
    // Hide from navigation entirely
    protected static bool $shouldRegisterNavigation = false;
    
    // Dynamic visibility
    public static function shouldRegisterNavigation(): bool
    {
        return auth()->user()->can('view posts');
    }
}
```

### Page Navigation

```php
class Reports extends Page
{
    public static function shouldRegisterNavigation(): bool
    {
        return auth()->user()->can('view reports');
    }
}
```

## Advanced Authorization

### Custom Authorization Logic

```php
class PostResource extends Resource
{
    public static function canEdit(Model $record): bool
    {
        $user = auth()->user();
        
        // Admin can edit anything
        if ($user->isAdmin()) {
            return true;
        }
        
        // Owner can edit their own posts
        if ($user->id === $record->user_id) {
            // But not if locked
            if ($record->isLocked()) {
                return false;
            }
            
            // And not if published (only drafts editable by owner)
            if ($record->status === 'published') {
                return false;
            }
            
            return true;
        }
        
        // Editors can edit any draft
        if ($user->isEditor() && $record->status === 'draft') {
            return true;
        }
        
        return false;
    }
}
```

### Time-Based Restrictions

```php
public static function canEdit(Model $record): bool
{
    // Can't edit posts older than 30 days (unless admin)
    if ($record->created_at->diffInDays(now()) > 30 && !auth()->user()->isAdmin()) {
        return false;
    }
    
    return auth()->user()->can('update', $record);
}
```

### Feature Flags

```php
public static function canCreate(): bool
{
    // Check if feature is enabled
    if (!Feature::active('new-posts')) {
        return false;
    }
    
    return auth()->user()->can('create posts');
}
```

## Best Practices

1. **Use policies for model-level authorization**
2. **Implement FilamentUser for panel access**
3. **Check permissions in actions, not just visibility**
4. **Use custom policy methods for specific actions**
5. **Implement multi-tenancy at the query level**
6. **Cache expensive permission checks**
7. **Test authorization thoroughly**
8. **Use roles for broad access, permissions for specific actions**
9. **Keep authorization logic in policies, not controllers**
10. **Use Response::deny() with messages for clarity**
11. **Document your authorization rules**
12. **Regularly audit permissions**
13. **Use middleware for route-level protection**
14. **Implement proper tenant isolation**
15. **Log sensitive authorization failures**

## Testing Authorization

```php
use function Pest\Laravel\actingAs;

it('denies access to guests', function () {
    auth()->logout();

    livewire(ListPosts::class)
        ->assertRedirect('/login');
});

it('denies access to unauthorized users', function () {
    $user = User::factory()->create(['role' => 'user']);
    actingAs($user);

    livewire(ListPosts::class)
        ->assertForbidden();
});

it('allows access to authorized users', function () {
    $admin = User::factory()->create(['role' => 'admin']);
    actingAs($admin);

    livewire(ListPosts::class)
        ->assertOk();
});

it('hides actions without permission', function () {
    $user = User::factory()->create();
    $user->revokePermissionTo('delete posts');
    actingAs($user);

    $post = Post::factory()->create();

    livewire(ListPosts::class)
        ->assertTableActionHidden('delete', $post);
});
```

## Additional Resources

- [Laravel Authorization](https://laravel.com/docs/authorization)
- [Spatie Laravel-Permission](https://spatie.be/docs/laravel-permission)
- [Filament Multi-Tenancy](https://filamentphp.com/docs/5.x/panels/tenancy)
