# Testing Reference

Complete guide for testing Filament v5 resources, forms, and tables with Pest PHP.

## Setup

```bash
# Install Pest
composer require pestphp/pest --dev

# Install Livewire testing plugin
composer require pestphp/pest-plugin-livewire --dev

# Install Filament testing helpers (if separate package)
composer require filament/testing --dev
```

## Basic Test Structure

```php
<?php

use App\Filament\Resources\PostResource;
use App\Filament\Resources\PostResource\Pages\CreatePost;
use App\Filament\Resources\PostResource\Pages\EditPost;
use App\Filament\Resources\PostResource\Pages\ListPosts;
use App\Filament\Resources\PostResource\Pages\ViewPost;
use App\Models\Post;
use App\Models\User;

use function Pest\Laravel\actingAs;
use function Pest\Laravel\assertDatabaseHas;
use function Pest\Laravel\assertDatabaseMissing;
use function Pest\Livewire\livewire;

beforeEach(function () {
    $this->user = User::factory()->create();
    actingAs($this->user);
});
```

## Testing List Pages

```php
// Basic page load
describe('List Page', function () {
    it('can render the list page', function () {
        livewire(ListPosts::class)
            ->assertOk();
    });

    it('can list posts', function () {
        $posts = Post::factory()->count(5)->create();

        livewire(ListPosts::class)
            ->assertCanSeeTableRecords($posts);
    });

    it('can search posts by title', function () {
        Post::factory()->create(['title' => 'Hello World']);
        Post::factory()->create(['title' => 'Another Post']);

        livewire(ListPosts::class)
            ->searchTable('Hello')
            ->assertCanSeeTableRecords(Post::where('title', 'like', '%Hello%')->get())
            ->assertCanNotSeeTableRecords(Post::where('title', 'not like', '%Hello%')->get());
    });

    it('can sort posts', function () {
        Post::factory()->create(['title' => 'C Post']);
        Post::factory()->create(['title' => 'A Post']);
        Post::factory()->create(['title' => 'B Post']);

        livewire(ListPosts::class)
            ->sortTable('title')
            ->assertCanSeeTableRecords(Post::orderBy('title')->get(), inOrder: true);
    });

    it('can filter posts by status', function () {
        Post::factory()->create(['status' => 'published']);
        Post::factory()->create(['status' => 'draft']);

        livewire(ListPosts::class)
            ->filterTable('status', 'published')
            ->assertCanSeeTableRecords(Post::where('status', 'published')->get())
            ->assertCanNotSeeTableRecords(Post::where('status', '!=', 'published')->get());
    });

    it('can filter posts with multiple statuses', function () {
        Post::factory()->create(['status' => 'published']);
        Post::factory()->create(['status' => 'draft']);
        Post::factory()->create(['status' => 'archived']);

        livewire(ListPosts::class)
            ->filterTable('status', ['published', 'draft'])
            ->assertCanSeeTableRecords(Post::whereIn('status', ['published', 'draft'])->get())
            ->assertCanNotSeeTableRecords(Post::where('status', 'archived')->get());
    });
});
```

## Testing Table Columns

```php
it('can render table columns', function () {
    livewire(ListPosts::class)
        ->assertCanRenderTableColumn('title')
        ->assertCanRenderTableColumn('status')
        ->assertCanRenderTableColumn('created_at')
        ->assertCanNotRenderTableColumn('password');  // Hidden column
});

it('can sort by date column', function () {
    $posts = Post::factory()->count(3)->create();

    livewire(ListPosts::class)
        ->sortTable('created_at')
        ->assertCanSeeTableRecords($posts->sortBy('created_at'), inOrder: true)
        ->sortTable('created_at', 'desc')
        ->assertCanSeeTableRecords($posts->sortByDesc('created_at'), inOrder: true);
});

it('can hide columns', function () {
    livewire(ListPosts::class)
        ->assertTableColumnVisible('title')
        ->assertTableColumnHidden('deleted_at');
});
```

## Testing Table Actions

```php
// Single record actions
it('can delete a post', function () {
    $post = Post::factory()->create();

    livewire(ListPosts::class)
        ->callTableAction('delete', $post)
        ->assertNotified()
        ->assertCanNotSeeTableRecords([$post]);

    assertDatabaseMissing('posts', ['id' => $post->id]);
});

it('can edit a post from list', function () {
    $post = Post::factory()->create();

    livewire(ListPosts::class)
        ->callTableAction('edit', $post)
        ->assertRedirect(PostResource::getUrl('edit', ['record' => $post]));
});

// Bulk actions
it('can bulk delete posts', function () {
    $posts = Post::factory()->count(3)->create();

    livewire(ListPosts::class)
        ->selectTableRecords($posts)
        ->callTableBulkAction('delete', $posts)
        ->assertNotified()
        ->assertCanNotSeeTableRecords($posts);

    $posts->each(fn ($post) => assertDatabaseMissing('posts', ['id' => $post->id]));
});

it('can bulk update status', function () {
    $posts = Post::factory()->count(3)->create(['status' => 'draft']);

    livewire(ListPosts::class)
        ->selectTableRecords($posts)
        ->callTableBulkAction('updateStatus', $posts, data: ['status' => 'published'])
        ->assertNotified()
        ->assertHasNoTableBulkActionErrors();

    $posts->each->refresh();
    expect($posts)->each->status->toBe('published');
});

// Custom actions
it('can publish a post', function () {
    $post = Post::factory()->create(['status' => 'draft']);

    livewire(ListPosts::class)
        ->callTableAction('publish', $post)
        ->assertNotified();

    expect($post->fresh()->status)->toBe('published');
});

it('can duplicate a post', function () {
    $post = Post::factory()->create(['title' => 'Original']);

    livewire(ListPosts::class)
        ->callTableAction('duplicate', $post)
        ->assertNotified();

    assertDatabaseHas('posts', [
        'title' => 'Original (Copy)',
        'slug' => $post->slug . '-copy',
    ]);
});
```

## Testing Create Pages

```php
describe('Create Page', function () {
    it('can render the create page', function () {
        livewire(CreatePost::class)
            ->assertOk();
    });

    it('can create a post', function () {
        $newData = Post::factory()->make();

        livewire(CreatePost::class)
            ->fillForm([
                'title' => $newData->title,
                'content' => $newData->content,
                'status' => $newData->status,
            ])
            ->call('create')
            ->assertNotified()
            ->assertRedirect(PostResource::getUrl('index'));

        assertDatabaseHas('posts', [
            'title' => $newData->title,
            'content' => $newData->content,
            'status' => $newData->status,
            'user_id' => $this->user->id,
        ]);
    });

    it('validates required fields', function () {
        livewire(CreatePost::class)
            ->fillForm([
                'title' => null,
                'content' => null,
            ])
            ->call('create')
            ->assertHasFormErrors(['title' => 'required', 'content' => 'required'])
            ->assertNotNotified();
    });

    it('validates email format', function () {
        livewire(CreatePost::class)
            ->fillForm([
                'title' => 'Test',
                'content' => 'Content',
                'author_email' => 'invalid-email',
            ])
            ->call('create')
            ->assertHasFormErrors(['author_email' => 'email']);
    });

    it('validates unique fields', function () {
        $existing = Post::factory()->create();

        livewire(CreatePost::class)
            ->fillForm([
                'title' => 'Test',
                'slug' => $existing->slug,  // Must be unique
            ])
            ->call('create')
            ->assertHasFormErrors(['slug' => 'unique']);
    });

    it('validates max length', function () {
        livewire(CreatePost::class)
            ->fillForm([
                'title' => str_repeat('a', 256),  // Max 255
            ])
            ->call('create')
            ->assertHasFormErrors(['title' => 'max']);
    });

    it('validates min length', function () {
        livewire(CreatePost::class)
            ->fillForm([
                'title' => 'ab',  // Min 3
            ])
            ->call('create')
            ->assertHasFormErrors(['title' => 'min']);
    });
});
```

## Testing Edit Pages

```php
describe('Edit Page', function () {
    it('can render the edit page', function () {
        $post = Post::factory()->create();

        livewire(EditPost::class, ['record' => $post->id])
            ->assertOk();
    });

    it('can retrieve post data', function () {
        $post = Post::factory()->create();

        livewire(EditPost::class, ['record' => $post->id])
            ->assertFormSet([
                'title' => $post->title,
                'content' => $post->content,
                'status' => $post->status,
            ]);
    });

    it('can update a post', function () {
        $post = Post::factory()->create();
        $newData = Post::factory()->make();

        livewire(EditPost::class, ['record' => $post->id])
            ->fillForm([
                'title' => $newData->title,
                'content' => $newData->content,
            ])
            ->call('save')
            ->assertNotified()
            ->assertRedirect(PostResource::getUrl('index'));

        assertDatabaseHas('posts', [
            'id' => $post->id,
            'title' => $newData->title,
            'content' => $newData->content,
        ]);
    });

    it('can delete a post from edit page', function () {
        $post = Post::factory()->create();

        livewire(EditPost::class, ['record' => $post->id])
            ->callAction('delete')
            ->assertNotified()
            ->assertRedirect(PostResource::getUrl('index'));

        assertDatabaseMissing('posts', ['id' => $post->id]);
    });

    it('cannot edit posts owned by other users', function () {
        $otherUser = User::factory()->create();
        $post = Post::factory()->create(['user_id' => $otherUser->id]);

        livewire(EditPost::class, ['record' => $post->id])
            ->assertForbidden();
    });
});
```

## Testing View Pages

```php
describe('View Page', function () {
    it('can render the view page', function () {
        $post = Post::factory()->create();

        livewire(ViewPost::class, ['record' => $post->id])
            ->assertOk();
    });

    it('can view post details', function () {
        $post = Post::factory()->create();

        livewire(ViewPost::class, ['record' => $post->id])
            ->assertInfolistSet([
                'title' => $post->title,
                'content' => $post->content,
            ]);
    });

    it('can navigate to edit from view', function () {
        $post = Post::factory()->create();

        livewire(ViewPost::class, ['record' => $post->id])
            ->callAction('edit')
            ->assertRedirect(PostResource::getUrl('edit', ['record' => $post]));
    });
});
```

## Testing Form Components

```php
it('can fill form fields', function () {
    livewire(CreatePost::class)
        ->fillForm([
            'title' => 'My Title',
            'content' => 'My Content',
            'is_featured' => true,
        ])
        ->assertFormSet([
            'title' => 'My Title',
            'content' => 'My Content',
            'is_featured' => true,
        ]);
});

it('can test select field', function () {
    livewire(CreatePost::class)
        ->fillForm([
            'status' => 'published',
        ])
        ->assertHasNoFormErrors();
});

it('can test date picker', function () {
    livewire(CreatePost::class)
        ->fillForm([
            'published_at' => now()->format('Y-m-d H:i:s'),
        ])
        ->assertHasNoFormErrors();
});

it('can test file upload', function () {
    Storage::fake('public');
    
    $file = UploadedFile::fake()->image('featured.jpg');
    
    livewire(CreatePost::class)
        ->fillForm([
            'title' => 'Test',
            'featured_image' => [$file],
        ])
        ->call('create')
        ->assertHasNoFormErrors();
    
    Storage::disk('public')->assertExists('posts/' . $file->hashName());
});

it('can test repeater field', function () {
    livewire(CreatePost::class)
        ->fillForm([
            'items' => [
                ['name' => 'Item 1', 'quantity' => 2],
                ['name' => 'Item 2', 'quantity' => 3],
            ],
        ])
        ->assertHasNoFormErrors();
});
```

## Testing Modal Actions

```php
it('can trigger modal action', function () {
    $post = Post::factory()->create();

    livewire(EditPost::class, ['record' => $post->id])
        ->callAction('sendEmail')
        ->assertActionHalted('sendEmail');  // Modal is open
});

it('can fill modal form and submit', function () {
    $post = Post::factory()->create();

    livewire(EditPost::class, ['record' => $post->id])
        ->callAction('sendEmail')
        ->assertActionHalted('sendEmail')
        ->fillForm([
            'subject' => 'Hello',
            'body' => 'Message body',
        ], component: 'sendEmail')
        ->callMountedAction()
        ->assertHasNoActionErrors()
        ->assertNotified();
});

it('validates modal form fields', function () {
    $post = Post::factory()->create();

    livewire(EditPost::class, ['record' => $post->id])
        ->callAction('sendEmail')
        ->fillForm([
            'subject' => '',  // Required
        ], component: 'sendEmail')
        ->callMountedAction()
        ->assertHasActionErrors(['subject' => 'required']);
});
```

## Testing Notifications

```php
it('shows success notification after create', function () {
    $data = Post::factory()->make();

    livewire(CreatePost::class)
        ->fillForm([
            'title' => $data->title,
            'content' => $data->content,
        ])
        ->call('create')
        ->assertNotified('Post created successfully');
});

it('shows custom notification message', function () {
    livewire(CreatePost::class)
        ->fillForm(['title' => ''])
        ->call('create')
        ->assertNotified(
            Notification::make()
                ->title('Error')
                ->body('Please fill all required fields')
                ->danger()
        );
});
```

## Testing Authorization

```php
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

it('cannot delete posts without permission', function () {
    $user = User::factory()->create();
    $user->revokePermissionTo('delete posts');
    actingAs($user);

    $post = Post::factory()->create();

    livewire(ListPosts::class)
        ->assertTableActionHidden('delete', $post);
});
```

## Testing Multi-Tenancy

```php
use Filament\Facades\Filament;

beforeEach(function () {
    $this->user = User::factory()->create();
    $this->team = Team::factory()->create();
    $this->user->teams()->attach($this->team);
    
    actingAs($this->user);
    Filament::setTenant($this->team);
});

it('shows only tenant posts', function () {
    $teamPost = Post::factory()->create(['team_id' => $this->team->id]);
    $otherPost = Post::factory()->create(['team_id' => Team::factory()->create()->id]);

    livewire(ListPosts::class)
        ->assertCanSeeTableRecords([$teamPost])
        ->assertCanNotSeeTableRecords([$otherPost]);
});

it('automatically sets tenant on create', function () {
    $newData = Post::factory()->make();

    livewire(CreatePost::class)
        ->fillForm([
            'title' => $newData->title,
            'content' => $newData->content,
        ])
        ->call('create')
        ->assertNotified();

    $this->assertDatabaseHas('posts', [
        'title' => $newData->title,
        'team_id' => $this->team->id,
    ]);
});
```

## Testing with Datasets

```php
it('validates input correctly', function (array $data, array $errors) {
    $newData = Post::factory()->make();

    livewire(CreatePost::class)
        ->fillForm([
            'title' => $newData->title,
            'content' => $newData->content,
            ...$data,
        ])
        ->call('create')
        ->assertHasFormErrors($errors);
})->with([
    'title is required' => [['title' => null], ['title' => 'required']],
    'title is max 255 characters' => [['title' => str_repeat('a', 256)], ['title' => 'max']],
    'email is valid' => [['author_email' => 'invalid'], ['author_email' => 'email']],
    'status is required' => [['status' => null], ['status' => 'required']],
]);

it('filters by status correctly', function (string $status, int $expectedCount) {
    Post::factory()->count(3)->create(['status' => 'published']);
    Post::factory()->count(2)->create(['status' => 'draft']);

    livewire(ListPosts::class)
        ->filterTable('status', $status)
        ->assertCanSeeTableRecords(Post::where('status', $status)->get()->slice(0, $expectedCount));
})->with([
    'published' => ['published', 3],
    'draft' => ['draft', 2],
]);
```

## Testing Custom Pages

```php
use App\Filament\Pages\Dashboard;
use App\Filament\Pages\Settings;

it('can access dashboard', function () {
    livewire(Dashboard::class)
        ->assertOk();
});

it('can access settings page', function () {
    $admin = User::factory()->create(['is_admin' => true]);
    actingAs($admin);

    livewire(Settings::class)
        ->assertOk();
});

it('can save settings', function () {
    $admin = User::factory()->create(['is_admin' => true]);
    actingAs($admin);

    livewire(Settings::class)
        ->fillForm([
            'data.site_name' => 'New Site Name',
        ])
        ->call('save')
        ->assertNotified();

    expect(setting('site_name'))->toBe('New Site Name');
});
```

## Testing Widgets

```php
use App\Filament\Widgets\StatsOverview;
use App\Filament\Widgets\OrdersChart;

it('displays stats correctly', function () {
    User::factory()->count(5)->create();

    livewire(StatsOverview::class)
        ->assertSee('5');  // User count
});

it('displays chart data', function () {
    Order::factory()->count(10)->create();

    livewire(OrdersChart::class)
        ->assertOk();
});
```

## Testing Relation Managers

```php
use App\Filament\Resources\PostResource\RelationManagers\CommentsRelationManager;

it('can render relation manager', function () {
    $post = Post::factory()->create();

    livewire(CommentsRelationManager::class, [
        'ownerRecord' => $post,
        'pageClass' => EditPost::class,
    ])
        ->assertOk();
});

it('can create related record', function () {
    $post = Post::factory()->create();

    livewire(CommentsRelationManager::class, [
        'ownerRecord' => $post,
        'pageClass' => EditPost::class,
    ])
        ->callTableAction('create')
        ->assertTableActionHalted('create')
        ->fillForm([
            'content' => 'Test comment',
        ])
        ->callMountedTableAction()
        ->assertHasNoTableActionErrors();

    assertDatabaseHas('comments', [
        'post_id' => $post->id,
        'content' => 'Test comment',
    ]);
});
```

## Best Practices

1. **Test all CRUD operations** - Create, Read, Update, Delete
2. **Test validation rules** - Required, format, unique, etc.
3. **Test authorization** - Access control and permissions
4. **Test table features** - Search, sort, filter, actions
5. **Test edge cases** - Empty data, large datasets, errors
6. **Use factories** - Create realistic test data
7. **Group related tests** - Use describe() blocks
8. **Use datasets** - Test multiple scenarios efficiently
9. **Test notifications** - Verify user feedback
10. **Test redirects** - After create/update/delete
11. **Test file uploads** - With storage fakes
12. **Test multi-tenancy** - Scoped data access
13. **Keep tests fast** - Avoid unnecessary DB calls
14. **Use beforeEach** - Set up common state
15. **Assert on database** - Verify state changes

## Helper Methods Reference

### Form Testing
- `fillForm(array $data)` - Fill form fields
- `assertFormSet(array $data)` - Assert form values
- `assertHasFormErrors(array $errors)` - Assert validation errors
- `assertHasNoFormErrors()` - Assert no errors
- `call(string $method)` - Call form method (e.g., 'create', 'save')

### Table Testing
- `assertCanSeeTableRecords($records)` - Assert records visible
- `assertCanNotSeeTableRecords($records)` - Assert records hidden
- `searchTable(string $query)` - Search table
- `sortTable(string $column, string $direction = 'asc')` - Sort table
- `filterTable(string $filter, $value)` - Apply filter
- `selectTableRecords($records)` - Select records for bulk actions
- `callTableAction(string $action, $record)` - Call row action
- `callTableBulkAction(string $action, $records)` - Call bulk action
- `assertTableColumnVisible(string $column)` - Assert column visible
- `assertTableColumnHidden(string $column)` - Assert column hidden

### Action Testing
- `callAction(string $action)` - Call page action
- `assertActionHalted(string $action)` - Assert modal opened
- `callMountedAction()` - Submit modal form
- `assertHasActionErrors(array $errors)` - Assert modal errors
- `assertHasNoActionErrors()` - Assert no modal errors

### Notification Testing
- `assertNotified(string $message = null)` - Assert notification shown
- `assertNotNotified()` - Assert no notification

### Authorization Testing
- `assertOk()` - Assert 200 status
- `assertForbidden()` - Assert 403 status
- `assertRedirect(string $url)` - Assert redirect

## Additional Resources

- [Pest PHP Documentation](https://pestphp.com/)
- [Livewire Testing](https://livewire.laravel.com/docs/testing)
- [Filament Testing](https://filamentphp.com/docs/5.x/testing/overview)
