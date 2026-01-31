# Schemas Reference

Complete guide for understanding and using Filament v5's schema system.

## Overview

Schemas are the foundation of Filament's Server-Driven UI approach. They allow you to build user interfaces declaratively using PHP configuration objects rather than writing HTML or JavaScript. Schemas define the structure and behavior of forms, infolists, tables, and layouts.

## What Are Schemas?

A schema is a collection of components that define:
- **Form fields** and their validation rules
- **Infolist entries** for displaying data
- **Table columns** and their formatting
- **Layout containers** (grids, sections, tabs)
- **Action definitions** and their behavior

## Schema Types

### Form Schemas

Used in resources, custom pages, and actions:

```php
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\Select;
use Filament\Forms\Components\Section;
use Filament\Forms\Form;

public function form(Form $form): Form
{
    return $form
        ->schema([
            Section::make('Details')
                ->schema([
                    TextInput::make('name')
                        ->required()
                        ->maxLength(255),
                    
                    Select::make('status')
                        ->options(['draft' => 'Draft', 'published' => 'Published'])
                        ->required(),
                ])
                ->columns(2),
        ]);
}
```

### Infolist Schemas

Used for read-only data display:

```php
use Filament\Infolists\Components\TextEntry;
use Filament\Infolists\Components\Section;
use Filament\Infolists\Infolist;

public function infolist(Infolist $infolist): Infolist
{
    return $infolist
        ->schema([
            Section::make('Details')
                ->schema([
                    TextEntry::make('name'),
                    TextEntry::make('status')
                        ->badge(),
                ])
                ->columns(2),
        ]);
}
```

### Table Schemas

Define table columns and filters:

```php
use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Columns\BadgeColumn;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->columns([
            TextColumn::make('name')
                ->searchable(),
            BadgeColumn::make('status')
                ->color(fn ($state) => match ($state) {
                    'draft' => 'gray',
                    'published' => 'success',
                }),
        ])
        ->filters([
            // Filters
        ]);
}
```

## Layout Components

### Grid

Organize components in columns:

```php
use Filament\Schemas\Components\Grid;

Grid::make(2)
    ->schema([
        TextInput::make('first_name'),
        TextInput::make('last_name'),
    ])

// Responsive grid
Grid::make([
    'default' => 1,
    'sm' => 2,
    'md' => 3,
    'lg' => 4,
])
    ->schema([
        // Components
    ])
```

### Section

Group components with a heading:

```php
use Filament\Schemas\Components\Section;

Section::make('Personal Information')
    ->description('Enter your personal details')
    ->icon('heroicon-m-user')
    ->collapsible()
    ->collapsed()
    ->compact()
    ->aside()                           // Side-by-side layout
    ->schema([
        TextInput::make('name'),
        TextInput::make('email'),
    ])
    ->columns(2);
```

### Tabs

Organize into tabs:

```php
use Filament\Schemas\Components\Tabs;

Tabs::make('Settings')
    ->tabs([
        Tabs\Tab::make('General')
            ->icon('heroicon-m-cog')
            ->schema([
                TextInput::make('site_name'),
            ]),
        
        Tabs\Tab::make('SEO')
            ->icon('heroicon-m-globe')
            ->schema([
                TextInput::make('meta_title'),
            ]),
    ]);
```

### Wizard

Multi-step form:

```php
use Filament\Schemas\Components\Wizard;

Wizard::make([
    Wizard\Step::make('Account')
        ->icon('heroicon-m-user')
        ->description('Create your account')
        ->schema([
            TextInput::make('email'),
            TextInput::make('password'),
        ]),
    
    Wizard\Step::make('Profile')
        ->icon('heroicon-m-identification')
        ->description('Set up your profile')
        ->schema([
            TextInput::make('name'),
        ]),
])
    ->skippable()
    ->persistInQueryString();
```

### Fieldset

Group without card styling:

```php
use Filament\Schemas\Components\Fieldset;

Fieldset::make('Address')
    ->schema([
        TextInput::make('street'),
        TextInput::make('city'),
    ]);
```

### Split

Side-by-side layout:

```php
use Filament\Schemas\Components\Split;

Split::make([
    Section::make('Details')
        ->schema([
            TextInput::make('name'),
        ]),
    
    Section::make('Avatar')
        ->schema([
            ImageUpload::make('avatar'),
        ]),
])
    ->from('lg');                       // Breakpoint
```

### Group

Simple grouping:

```php
use Filament\Schemas\Components\Group;

Group::make()
    ->schema([
        // Components
    ])
    ->columnSpanFull()
    ->columns(2);
```

## Component States

### State Management

```php
use Filament\Schemas\Components\Utilities\Get;
use Filament\Schemas\Components\Utilities\Set;

TextInput::make('title')
    ->live(onBlur: true)
    ->afterStateUpdated(fn ($state, Set $set) => 
        $set('slug', Str::slug($state))
    );

// Accessing other field values
TextInput::make('total')
    ->formatStateUsing(fn ($state, Get $get) => 
        $get('quantity') * $get('price')
    )
    ->dehydrated(false);
```

### State Hydration

```php
TextInput::make('name')
    ->formatStateUsing(fn ($state) => strtoupper($state))
    ->dehydrateStateUsing(fn ($state) => strtolower($state));
```

### Default Values

```php
TextInput::make('status')
    ->default('draft');

Select::make('role')
    ->default('user');

Toggle::make('is_active')
    ->default(true);
```

## Conditional Logic

### Visible/Hidden

```php
TextInput::make('company_name')
    ->visible(fn (Get $get) => $get('is_company'))
    ->hidden(fn (Get $get) => !$get('is_company'));
```

### Disabled/Readonly

```php
TextInput::make('email')
    ->disabled(fn () => auth()->user()->cannot('edit_email'));

TextInput::make('created_at')
    ->readonly();
```

### Required

```php
TextInput::make('company_name')
    ->required(fn (Get $get) => $get('is_company'));
```

## Schema Validation

### Field-Level Validation

```php
TextInput::make('email')
    ->email()
    ->required()
    ->unique('users', 'email')
    ->maxLength(255);

TextInput::make('password')
    ->password()
    ->required()
    ->minLength(8)
    ->confirmed();
```

### Custom Rules

```php
TextInput::make('username')
    ->rules(['required', 'string', 'regex:/^[a-z0-9_]+$/']);

TextInput::make('code')
    ->rule(function ($state) {
        return $state === 'VALID' ? null : 'Invalid code';
    });
```

### Validation Messages

```php
TextInput::make('email')
    ->email()
    ->validationMessages([
        'email' => 'Please enter a valid email address.',
        'required' => 'Email is required.',
    ]);
```

## Schema Customization

### Columns

```php
Section::make('Details')
    ->schema([
        TextInput::make('name'),
        TextInput::make('email'),
        TextInput::make('phone'),
    ])
    ->columns(2);

// Responsive columns
Section::make('Details')
    ->columns([
        'default' => 1,
        'sm' => 2,
        'lg' => 3,
    ]);
```

### Column Span

```php
TextInput::make('title')
    ->columnSpan(2);

TextInput::make('content')
    ->columnSpanFull();

// Responsive span
TextInput::make('name')
    ->columnSpan([
        'default' => 1,
        'lg' => 2,
    ]);
```

### Extra Attributes

```php
TextInput::make('name')
    ->extraAttributes(['class' => 'custom-class'])
    ->extraInputAttributes(['autocomplete' => 'off']);
```

## Advanced Patterns

### Dynamic Schema

```php
public function form(Form $form): Form
{
    return $form
        ->schema(fn (): array => [
            TextInput::make('name'),
            
            // Conditionally include fields
            ...(auth()->user()->isAdmin() ? [
                TextInput::make('admin_notes'),
            ] : []),
        ]);
}
```

### Builder Pattern

```php
use Filament\Forms\Components\Builder;

Builder::make('content')
    ->blocks([
        Builder\Block::make('heading')
            ->schema([
                TextInput::make('content'),
                Select::make('level'),
            ]),
        
        Builder\Block::make('paragraph')
            ->schema([
                RichEditor::make('content'),
            ]),
    ]);
```

### Repeater Pattern

```php
use Filament\Forms\Components\Repeater;

Repeater::make('items')
    ->schema([
        TextInput::make('name'),
        TextInput::make('quantity'),
    ])
    ->collapsible()
    ->itemLabel(fn (array $state): ?string => $state['name'] ?? null);
```

### Schema Composition

```php
// Reusable schema class
class UserSchema
{
    public static function make(): array
    {
        return [
            TextInput::make('name')->required(),
            TextInput::make('email')->email()->required(),
        ];
    }
}

// Use in resource
public function form(Form $form): Form
{
    return $form
        ->schema([
            ...UserSchema::make(),
            
            TextInput::make('phone'),
        ]);
}
```

## Schema in Different Contexts

### In Resources

```php
class PostResource extends Resource
{
    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                // Form schema
            ]);
    }
    
    public static function infolist(Infolist $infolist): Infolist
    {
        return $infolist
            ->schema([
                // Infolist schema
            ]);
    }
    
    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                // Table schema (columns)
            ]);
    }
}
```

### In Custom Pages

```php
class Settings extends Page
{
    protected ?array $data = [];
    
    public function form(Form $form): Form
    {
        return $form
            ->statePath('data')
            ->schema([
                TextInput::make('site_name'),
            ]);
    }
}
```

### In Actions

```php
Action::make('edit')
    ->form([
        TextInput::make('name'),
        TextInput::make('email'),
    ])
    ->action(function (array $data) {
        // Handle data
    });
```

## Schema State Path

### State Path Configuration

```php
public function form(Form $form): Form
{
    return $form
        ->statePath('data')              // Root state path
        ->schema([
            TextInput::make('name'),      // Accesses $data['name']
            
            Section::make('Address')
                ->statePath('address')    // Nested path
                ->schema([
                    TextInput::make('street'),  // $data['address']['street']
                ]),
        ]);
}
```

### Dehydration Control

```php
TextInput::make('display_value')
    ->dehydrated(false)                // Don't save to database
    ->formatStateUsing(fn ($state, Get $get) => 
        $get('quantity') * $get('price')
    );
```

## Schema Testing

### Testing Forms

```php
it('validates form schema', function () {
    livewire(CreatePost::class)
        ->fillForm([
            'title' => null,
        ])
        ->call('create')
        ->assertHasFormErrors(['title' => 'required']);
});

it('fills form schema correctly', function () {
    livewire(EditPost::class, ['record' => $post])
        ->assertFormSet([
            'title' => $post->title,
        ]);
});
```

### Testing Conditional Logic

```php
it('shows company fields when is_company is true', function () {
    livewire(CreateUser::class)
        ->fillForm(['is_company' => true])
        ->assertFormFieldVisible('company_name');
});
```

## Best Practices

1. **Keep schemas organized** - Use sections and clear naming
2. **Extract reusable schemas** - Create schema classes for common patterns
3. **Use live sparingly** - Too many live fields hurt performance
4. **Leverage conditional logic** - Show/hide based on context
5. **Validate at field level** - Use built-in validation methods
6. **Test schema behavior** - Verify conditional logic works correctly
7. **Use responsive layouts** - Grid columns that adapt to screen size
8. **Minimize nesting** - Avoid deeply nested structures
9. **Document complex schemas** - Add comments for clarity
10. **Reuse components** - Create custom components for repetition

## Common Patterns

### Address Schema

```php
public static function addressSchema(): array
{
    return [
        Grid::make(2)
            ->schema([
                TextInput::make('street')
                    ->required()
                    ->columnSpanFull(),
                
                TextInput::make('city')
                    ->required(),
                
                TextInput::make('state')
                    ->required(),
                
                TextInput::make('zip')
                    ->required()
                    ->maxLength(10),
                
                TextInput::make('country')
                    ->required(),
            ]),
    ];
}
```

### Contact Schema

```php
public static function contactSchema(): array
{
    return [
        TextInput::make('email')
            ->email()
            ->required(),
        
        TextInput::make('phone')
            ->tel(),
        
        TextInput::make('website')
            ->url()
            ->prefix('https://'),
    ];
}
```

## Additional Resources

- [Official Schemas Documentation](https://filamentphp.com/docs/5.x/schemas)
- [Form Components](https://filamentphp.com/docs/5.x/forms)
- [Infolist Components](https://filamentphp.com/docs/5.x/infolists)
- [Layout Components](https://filamentphp.com/docs/5.x/schemas/layout)
