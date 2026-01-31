# Resources Reference

Complete guide for creating and configuring Filament v5 resources (CRUD interfaces).

## Creating Resources

```bash
# Basic resource with all pages
php artisan make:filament-resource Post

# With automatic model and migration generation
php artisan make:filament-resource Post --generate

# Simple resource (modals only, no dedicated pages)
php artisan make:filament-resource Post --simple

# With view page
php artisan make:filament-resource Post --view

# With soft delete support
php artisan make:filament-resource Post --soft-deletes

# Simple with soft deletes
php artisan make:filament-resource Post --simple --soft-deletes
```

## Resource Structure

A complete resource includes:

```
app/Filament/Resources/
└── PostResource/
    ├── PostResource.php           # Main resource class
    ├── Pages/
    │   ├── CreatePost.php         # Create page
    │   ├── EditPost.php           # Edit page
    │   ├── ListPosts.php          # List page (table)
    │   └── ViewPost.php           # View page (optional)
    ├── RelationManagers/
    │   └── CommentsRelationManager.php
    └── Widgets/
        └── PostStatsWidget.php
```

## Basic Resource Class

```php
<?php

namespace App\Filament\Resources;

use App\Filament\Resources\PostResource\Pages;
use App\Filament\Resources\PostResource\RelationManagers;
use App\Models\Post;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

class PostResource extends Resource
{
    // Model configuration
    protected static ?string $model = Post::class;
    
    // UI configuration
    protected static ?string $navigationIcon = 'heroicon-o-document-text';
    protected static ?string $navigationGroup = 'Content';
    protected static ?int $navigationSort = 1;
    protected static ?string $recordTitleAttribute = 'title';
    protected static ?string $modelLabel = 'Blog Post';
    protected static ?string $pluralModelLabel = 'Blog Posts';
    
    // Authorization
    protected static bool $shouldSkipAuthorization = false;
    
    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                // Form fields
            ]);
    }
    
    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                // Table columns
            ])
            ->filters([
                // Filters
            ])
            ->actions([
                // Row actions
            ])
            ->bulkActions([
                // Bulk actions
            ]);
    }
    
    public static function getRelations(): array
    {
        return [
            RelationManagers\CommentsRelationManager::class,
        ];
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
    
    public static function getEloquentQuery(): Builder
    {
        return parent::getEloquentQuery()
            ->with(['author', 'category'])
            ->withoutGlobalScopes([
                // SoftDeletingScope::class,
            ]);
    }
}
```

## Resource Configuration Options

### Navigation

```php
protected static ?string $navigationIcon = 'heroicon-o-document-text';
protected static ?string $navigationGroup = 'Content';
protected static ?int $navigationSort = 1;
protected static ?string $navigationLabel = 'All Posts';
protected static ?string $activeNavigationIcon = 'heroicon-s-document-text';

// Hide from navigation
protected static bool $shouldRegisterNavigation = false;

// Dynamic navigation visibility
public static function shouldRegisterNavigation(): bool
{
    return auth()->user()->can('view posts');
}
```

### Labels

```php
protected static ?string $modelLabel = 'Blog Post';
protected static ?string $pluralModelLabel = 'Blog Posts';
protected static ?string $recordTitleAttribute = 'title';

// Dynamic titles
public static function getGlobalSearchResultTitle(Model $record): string
{
    return $record->title;
}

public static function getNavigationLabel(): string
{
    return 'All Posts';
}
```

### Slugs and Routes

```php
protected static ?string $slug = 'posts';

// Dynamic slug
public static function getSlug(): string
{
    return 'blog-posts';
}
```

## Form Configuration

```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Forms\Components\Section::make('Content')
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
                        ->unique(ignoreRecord: true),
                    
                    Forms\Components\RichEditor::make('content')
                        ->required()
                        ->columnSpanFull(),
                ]),
            
            Forms\Components\Section::make('Publishing')
                ->schema([
                    Forms\Components\Select::make('status')
                        ->options([
                            'draft' => 'Draft',
                            'published' => 'Published',
                            'archived' => 'Archived',
                        ])
                        ->required(),
                    
                    Forms\Components\DateTimePicker::make('published_at'),
                    
                    Forms\Components\Toggle::make('is_featured'),
                ])
                ->columns(2),
            
            Forms\Components\Section::make('Relationships')
                ->schema([
                    Forms\Components\Select::make('author_id')
                        ->relationship('author', 'name')
                        ->searchable()
                        ->preload()
                        ->required(),
                    
                    Forms\Components\Select::make('category_id')
                        ->relationship('category', 'name')
                        ->searchable()
                        ->preload(),
                    
                    Forms\Components\Select::make('tags')
                        ->relationship('tags', 'name')
                        ->multiple()
                        ->preload()
                        ->searchable(),
                ]),
        ])
        ->columns(1);
}
```

## Table Configuration

```php
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
                    'archived' => 'danger',
                }),
            
            Tables\Columns\TextColumn::make('author.name')
                ->searchable()
                ->sortable(),
            
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
        ])
        ->defaultSort('created_at', 'desc');
}
```

## Resource Pages

### List Page

```php
<?php

namespace App\Filament\Resources\PostResource\Pages;

use App\Filament\Resources\PostResource;
use Filament\Actions;
use Filament\Resources\Pages\ListRecords;

class ListPosts extends ListRecords
{
    protected static string $resource = PostResource::class;
    
    protected function getHeaderActions(): array
    {
        return [
            Actions\CreateAction::make(),
        ];
    }
    
    // Custom title
    protected function getHeader(): ?array
    {
        return [
            'heading' => 'Blog Posts',
            'subheading' => 'Manage your blog content',
        ];
    }
}
```

### Create Page

```php
<?php

namespace App\Filament\Resources\PostResource\Pages;

use App\Filament\Resources\PostResource;
use Filament\Resources\Pages\CreateRecord;

class CreatePost extends CreateRecord
{
    protected static string $resource = PostResource::class;
    
    // Custom redirect after creation
    protected function getRedirectUrl(): string
    {
        return $this->getResource()::getUrl('index');
    }
    
    // Mutate form data before creation
    protected function mutateFormDataBeforeCreate(array $data): array
    {
        $data['user_id'] = auth()->id();
        return $data;
    }
}
```

### Edit Page

```php
<?php

namespace App\Filament\Resources\PostResource\Pages;

use App\Filament\Resources\PostResource;
use Filament\Actions;
use Filament\Resources\Pages\EditRecord;

class EditPost extends EditRecord
{
    protected static string $resource = PostResource::class;
    
    protected function getHeaderActions(): array
    {
        return [
            Actions\ViewAction::make(),
            Actions\DeleteAction::make(),
        ];
    }
    
    protected function getRedirectUrl(): string
    {
        return $this->getResource()::getUrl('index');
    }
    
    protected function mutateFormDataBeforeSave(array $data): array
    {
        $data['updated_by'] = auth()->id();
        return $data;
    }
}
```

### View Page

```php
<?php

namespace App\Filament\Resources\PostResource\Pages;

use App\Filament\Resources\PostResource;
use Filament\Actions;
use Filament\Resources\Pages\ViewRecord;

class ViewPost extends ViewRecord
{
    protected static string $resource = PostResource::class;
    
    protected function getHeaderActions(): array
    {
        return [
            Actions\EditAction::make(),
            Actions\DeleteAction::make(),
        ];
    }
}
```

## Relation Managers

### Creating Relation Managers

```bash
php artisan make:filament-relation-manager PostResource comments id
```

### Relation Manager Class

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
    
    protected static ?string $title = 'Comments';
    
    protected static ?string $recordTitleAttribute = 'content';
    
    public function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\Textarea::make('content')
                    ->required()
                    ->maxLength(1000)
                    ->columnSpanFull(),
                
                Forms\Components\Toggle::make('is_approved')
                    ->default(true),
            ]);
    }
    
    public function table(Table $table): Table
    {
        return $table
            ->recordTitleAttribute('content')
            ->columns([
                Tables\Columns\TextColumn::make('content')
                    ->limit(50),
                
                Tables\Columns\TextColumn::make('user.name'),
                
                Tables\Columns\IconColumn::make('is_approved')
                    ->boolean(),
                
                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime(),
            ])
            ->filters([
                Tables\Filters\TernaryFilter::make('is_approved'),
            ])
            ->headerActions([
                Tables\Actions\CreateAction::make(),
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
}
```

### Registering Relations

```php
public static function getRelations(): array
{
    return [
        RelationManagers\CommentsRelationManager::class,
        RelationManagers\TagsRelationManager::class,
    ];
}
```

## Global Search

Enable global search for a resource:

```php
protected static ?string $recordTitleAttribute = 'title';

public static function getGlobalSearchResultTitle(Model $record): string
{
    return $record->title;
}

public static function getGlobalSearchResultDetails(Model $record): array
{
    return [
        'Author' => $record->author->name,
        'Status' => $record->status,
    ];
}

public static function getGlobalSearchEloquentQuery(): Builder
{
    return parent::getGlobalSearchEloquentQuery()
        ->with(['author']);
}

public static function getGloballySearchableAttributes(): array
{
    return ['title', 'content', 'slug'];
}
```

## Sub-Navigation

Add tabs to the resource:

```php
public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        Pages\ViewPost::class,
        Pages\EditPost::class,
        Pages\ManageComments::class,
    ]);
}
```

## Custom Pages

Create custom resource pages:

```bash
php artisan make:filament-page ManagePostComments \
    --resource=PostResource \
    --type=ManageRelatedRecords
```

```php
<?php

namespace App\Filament\Resources\PostResource\Pages;

use App\Filament\Resources\PostResource;
use Filament\Resources\Pages\ManageRelatedRecords;
use Filament\Tables;

class ManageComments extends ManageRelatedRecords
{
    protected static string $resource = PostResource::class;
    
    protected static string $relationship = 'comments';
    
    protected static ?string $navigationIcon = 'heroicon-o-chat-bubble-left-ellipsis';
    
    public function table(Tables\Table $table): Tables\Table
    {
        return $table
            ->recordTitleAttribute('content')
            ->columns([
                Tables\Columns\TextColumn::make('content'),
                Tables\Columns\TextColumn::make('user.name'),
            ]);
    }
}
```

## Soft Deletes

Enable soft delete support:

```bash
php artisan make:filament-resource Post --soft-deletes
```

```php
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()
        ->withoutGlobalScopes([
            SoftDeletingScope::class,
        ]);
}

// Add actions in table
->actions([
    Tables\Actions\EditAction::make(),
    Tables\Actions\DeleteAction::make(),
    Tables\Actions\ForceDeleteAction::make(),
    Tables\Actions\RestoreAction::make(),
])

// Add filter
->filters([
    Tables\Filters\TrashedFilter::make(),
])
```

## Authorization

```php
public static function canViewAny(): bool
{
    return auth()->user()->can('view posts');
}

public static function canCreate(): bool
{
    return auth()->user()->can('create posts');
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
```

## Infolist (View Page Details)

```php
use Filament\Infolists\Components\TextEntry;
use Filament\Infolists\Components\ImageEntry;
use Filament\Infolists\Infolist;

public static function infolist(Infolist $infolist): Infolist
{
    return $infolist
        ->schema([
            TextEntry::make('title'),
            TextEntry::make('content')
                ->html()
                ->columnSpanFull(),
            TextEntry::make('author.name'),
            ImageEntry::make('featured_image'),
        ]);
}
```

## Complete Resource Example

```php
<?php

namespace App\Filament\Resources;

use App\Filament\Resources\PostResource\Pages;
use App\Filament\Resources\PostResource\RelationManagers;
use App\Models\Post;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Infolists\Components\ImageEntry;
use Filament\Infolists\Components\TextEntry;
use Filament\Infolists\Infolist;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletingScope;
use Illuminate\Support\Str;

class PostResource extends Resource
{
    protected static ?string $model = Post::class;
    protected static ?string $navigationIcon = 'heroicon-o-document-text';
    protected static ?string $navigationGroup = 'Content';
    protected static ?int $navigationSort = 1;
    protected static ?string $recordTitleAttribute = 'title';
    
    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\Section::make('Basic Information')
                    ->schema([
                        Forms\Components\TextInput::make('title')
                            ->required()
                            ->maxLength(255)
                            ->live(onBlur: true)
                            ->afterStateUpdated(fn ($state, $set) => 
                                $set('slug', Str::slug($state))
                            ),
                        
                        Forms\Components\TextInput::make('slug')
                            ->required()
                            ->unique(ignoreRecord: true),
                        
                        Forms\Components\RichEditor::make('content')
                            ->required()
                            ->columnSpanFull(),
                    ]),
                
                Forms\Components\Section::make('Publishing')
                    ->schema([
                        Forms\Components\Select::make('status')
                            ->options([
                                'draft' => 'Draft',
                                'published' => 'Published',
                                'archived' => 'Archived',
                            ])
                            ->required(),
                        
                        Forms\Components\DateTimePicker::make('published_at'),
                        
                        Forms\Components\Toggle::make('is_featured'),
                    ])
                    ->columns(2),
                
                Forms\Components\Section::make('Relationships')
                    ->schema([
                        Forms\Components\Select::make('author_id')
                            ->relationship('author', 'name')
                            ->searchable()
                            ->preload()
                            ->required(),
                        
                        Forms\Components\Select::make('category_id')
                            ->relationship('category', 'name')
                            ->searchable()
                            ->preload(),
                    ])
                    ->columns(2),
            ])
            ->columns(1);
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
                        'archived' => 'danger',
                    }),
                
                Tables\Columns\TextColumn::make('author.name')
                    ->searchable()
                    ->sortable(),
                
                Tables\Columns\TextColumn::make('published_at')
                    ->dateTime()
                    ->sortable(),
                
                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable()
                    ->toggleable(isToggledHiddenByDefault: true),
            ])
            ->filters([
                Tables\Filters\SelectFilter::make('status')
                    ->options([
                        'draft' => 'Draft',
                        'published' => 'Published',
                        'archived' => 'Archived',
                    ]),
                
                Tables\Filters\TrashedFilter::make(),
            ])
            ->actions([
                Tables\Actions\ViewAction::make(),
                Tables\Actions\EditAction::make(),
                Tables\Actions\DeleteAction::make(),
                Tables\Actions\ForceDeleteAction::make(),
                Tables\Actions\RestoreAction::make(),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                    Tables\Actions\ForceDeleteBulkAction::make(),
                    Tables\Actions\RestoreBulkAction::make(),
                ]),
            ])
            ->defaultSort('created_at', 'desc');
    }
    
    public static function infolist(Infolist $infolist): Infolist
    {
        return $infolist
            ->schema([
                TextEntry::make('title'),
                TextEntry::make('slug'),
                TextEntry::make('content')
                    ->html()
                    ->columnSpanFull(),
                TextEntry::make('author.name'),
                TextEntry::make('status'),
                TextEntry::make('published_at'),
                ImageEntry::make('featured_image'),
            ]);
    }
    
    public static function getRelations(): array
    {
        return [
            RelationManagers\CommentsRelationManager::class,
        ];
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
    
    public static function getEloquentQuery(): Builder
    {
        return parent::getEloquentQuery()
            ->withoutGlobalScopes([
                SoftDeletingScope::class,
            ]);
    }
}
```

## Best Practices

1. **Use eager loading** in `getEloquentQuery()` to prevent N+1
2. **Organize forms with Sections** for better UX
3. **Add search and sort** to frequently accessed columns
4. **Use colors on status badges** for visual feedback
5. **Implement authorization** with policies
6. **Add relation managers** for connected data
7. **Enable soft deletes** for data safety
8. **Use toggleable columns** for wide tables
9. **Add global search** for better discoverability
10. **Keep forms focused** - use multiple sections or tabs
11. **Validate all input** with appropriate rules
12. **Test with large datasets** to ensure performance
13. **Use infolists** on view pages for read-only data
14. **Add helpful actions** like duplicate or export
15. **Document your resources** with comments

## Additional Resources

- [Official Resources Documentation](https://filamentphp.com/docs/5.x/resources/overview)
- [Relation Managers](https://filamentphp.com/docs/5.x/resources/relation-managers)
- [Global Search](https://filamentphp.com/docs/5.x/resources/global-search)
