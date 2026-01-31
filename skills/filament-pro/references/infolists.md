# Infolists Reference

Complete guide for creating read-only data displays in Filament v5.

## Overview

Infolists are components for rendering "description lists" - read-only displays of data in a label-value format. They are commonly used on view pages, custom pages, and relation managers to present record information.

## Basic Infolist

### Simple Infolist

```php
<?php

namespace App\Filament\Resources\PostResource;

use Filament\Infolists\Components\TextEntry;
use Filament\Infolists\Infolist;
use Filament\Resources\Resource;

class PostResource extends Resource
{
    public static function infolist(Infolist $infolist): Infolist
    {
        return $infolist
            ->schema([
                TextEntry::make('title'),
                TextEntry::make('content')
                    ->columnSpanFull(),
                TextEntry::make('created_at')
                    ->dateTime(),
            ]);
    }
}
```

## Entry Types

### Text Entry

```php
use Filament\Infolists\Components\TextEntry;

TextEntry::make('name')
    ->label('Full Name')
    ->weight('bold')                    // 'bold', 'semibold', 'medium'
    ->color('primary')                  // 'primary', 'success', 'danger', etc.
    ->icon('heroicon-m-user')
    ->iconColor('primary')
    ->copyable()
    ->copyMessage('Copied!')
    ->copyMessageDuration(1500)
    ->limit(50)
    ->tooltip('Full name of the user')
    ->placeholder('No name provided')
    ->prefix('Mr/Ms ')
    ->suffix(' (verified)')
    ->alignLeft()                       // alignLeft, alignCenter, alignRight
    ->columnSpanFull()
    ->hidden(fn ($record) => !$record->name)
    ->visible(fn ($record) => $record->isPublished());
```

### Text Formatting

```php
TextEntry::make('price')
    ->money('USD')                      // Currency formatting
    ->formatStateUsing(fn ($state) => number_format($state, 2));

TextEntry::make('created_at')
    ->dateTime('M j, Y g:i A')          // Custom date format
    ->since();                          // Relative time ("2 hours ago")

TextEntry::make('content')
    ->markdown()                        // Render as markdown
    ->html()                            // Render as HTML
    ->prose()                           // Apply prose styling
    ->columnSpanFull();

TextEntry::make('slug')
    ->url(fn ($record) => route('posts.show', $record))
    ->openUrlInNewTab();

TextEntry::make('status')
    ->badge()
    ->color(fn (string $state): string => match ($state) {
        'draft' => 'gray',
        'published' => 'success',
        'archived' => 'danger',
    });
```

### Icon Entry

```php
use Filament\Infolists\Components\IconEntry;

IconEntry::make('is_active')
    ->boolean()                         // Shows check/x icons
    ->trueIcon('heroicon-m-check-circle')
    ->falseIcon('heroicon-m-x-circle')
    ->trueColor('success')
    ->falseColor('danger');

IconEntry::make('status')
    ->icon(fn (string $state): string => match ($state) {
        'active' => 'heroicon-m-check-circle',
        'inactive' => 'heroicon-m-x-circle',
        default => 'heroicon-m-question-mark-circle',
    })
    ->color(fn (string $state): string => match ($state) {
        'active' => 'success',
        'inactive' => 'danger',
        default => 'gray',
    });
```

### Image Entry

```php
use Filament\Infolists\Components\ImageEntry;

ImageEntry::make('avatar')
    ->disk('public')
    ->square()                          // Square aspect ratio
    ->circular()                        // Circular (avatar style)
    ->size(100)                         // Size in pixels
    ->width(200)
    ->height(200)
    ->checkFileExistence()
    ->defaultImageUrl(fn ($record) => 'https://ui-avatars.com/api/?name=' . urlencode($record->name));

ImageEntry::make('gallery')
    ->multiple()
    ->limit(3)                          // Show only 3 images
    ->limitedRemainingText();           // Show "+X more" text
```

### Color Entry

```php
use Filament\Infolists\Components\ColorEntry;

ColorEntry::make('brand_color')
    ->copyable();
```

### Key-Value Entry

```php
use Filament\Infolists\Components\KeyValueEntry;

KeyValueEntry::make('meta')
    ->keyLabel('Property')
    ->valueLabel('Value');
```

### Repeatable Entry

```php
use Filament\Infolists\Components\RepeatableEntry;
use Filament\Infolists\Components\TextEntry;

RepeatableEntry::make('orderItems')
    ->schema([
        TextEntry::make('product.name'),
        TextEntry::make('quantity'),
        TextEntry::make('price')
            ->money('USD'),
    ])
    ->columns(3)
    ->contained(false);                 // Remove card styling
```

## Layout Components

### Section

```php
use Filament\Infolists\Components\Section;

Section::make('Personal Information')
    ->description('User details and contact information')
    ->icon('heroicon-m-user')
    ->collapsible()
    ->collapsed()
    ->compact()
    ->aside()                           // Side-by-side label/value
    ->schema([
        TextEntry::make('name'),
        TextEntry::make('email'),
        TextEntry::make('phone'),
    ])
    ->columns(2);
```

### Grid

```php
use Filament\Infolists\Components\Grid;

Grid::make(2)
    ->schema([
        TextEntry::make('first_name'),
        TextEntry::make('last_name'),
    ]);

// Responsive grid
Grid::make([
    'default' => 1,
    'sm' => 2,
    'lg' => 3,
])
    ->schema([
        // ...
    ]);
```

### Tabs

```php
use Filament\Infolists\Components\Tabs;

Tabs::make('User Details')
    ->tabs([
        Tabs\Tab::make('General')
            ->icon('heroicon-m-user')
            ->schema([
                TextEntry::make('name'),
                TextEntry::make('email'),
            ]),
        
        Tabs\Tab::make('Professional')
            ->icon('heroicon-m-briefcase')
            ->schema([
                TextEntry::make('job_title'),
                TextEntry::make('company'),
            ]),
        
        Tabs\Tab::make('Social')
            ->icon('heroicon-m-share')
            ->schema([
                TextEntry::make('twitter'),
                TextEntry::make('linkedin'),
            ]),
    ]);
```

### Split

```php
use Filament\Infolists\Components\Split;

Split::make([
    Section::make('Details')
        ->schema([
            TextEntry::make('name'),
            TextEntry::make('email'),
        ]),
    
    Section::make('Avatar')
        ->schema([
            ImageEntry::make('avatar')
                ->circular()
                ->size(150),
        ]),
])
    ->from('lg');                       // Split on large screens
```

### Fieldset

```php
use Filament\Infolists\Components\Fieldset;

Fieldset::make('Address')
    ->schema([
        TextEntry::make('street'),
        TextEntry::make('city'),
        TextEntry::make('zip'),
    ]);
```

## Complete Examples

### User Profile Infolist

```php
public static function infolist(Infolist $infolist): Infolist
{
    return $infolist
        ->schema([
            Section::make('Profile')
                ->schema([
                    Split::make([
                        Grid::make(2)
                            ->schema([
                                TextEntry::make('name')
                                    ->weight('bold')
                                    ->size(TextEntry\Size::Large),
                                
                                TextEntry::make('email')
                                    ->icon('heroicon-m-envelope')
                                    ->copyable(),
                                
                                TextEntry::make('phone')
                                    ->icon('heroicon-m-phone'),
                                
                                TextEntry::make('role.name')
                                    ->badge()
                                    ->color('primary'),
                                
                                IconEntry::make('is_active')
                                    ->boolean()
                                    ->label('Status'),
                            ]),
                        
                        ImageEntry::make('avatar')
                            ->circular()
                            ->size(150)
                            ->hiddenLabel(),
                    ])
                    ->from('md'),
                ]),
            
            Section::make('Professional Information')
                ->collapsible()
                ->schema([
                    TextEntry::make('job_title'),
                    TextEntry::make('company'),
                    TextEntry::make('bio')
                        ->markdown()
                        ->columnSpanFull(),
                ])
                ->columns(2),
            
            Section::make('Metadata')
                ->collapsed()
                ->schema([
                    TextEntry::make('created_at')
                        ->dateTime()
                        ->since(),
                    
                    TextEntry::make('updated_at')
                        ->dateTime()
                        ->since(),
                ])
                ->columns(2),
        ]);
}
```

### Order Details Infolist

```php
public static function infolist(Infolist $infolist): Infolist
{
    return $infolist
        ->schema([
            Section::make('Order Summary')
                ->schema([
                    TextEntry::make('order_number')
                        ->label('Order #')
                        ->weight('bold'),
                    
                    TextEntry::make('status')
                        ->badge()
                        ->color(fn (string $state): string => match ($state) {
                            'pending' => 'warning',
                            'processing' => 'info',
                            'completed' => 'success',
                            'cancelled' => 'danger',
                        }),
                    
                    TextEntry::make('total')
                        ->money('USD')
                        ->weight('bold')
                        ->size(TextEntry\Size::Large),
                    
                    TextEntry::make('created_at')
                        ->dateTime(),
                ])
                ->columns(4),
            
            Section::make('Customer')
                ->schema([
                    TextEntry::make('customer.name')
                        ->label('Name'),
                    
                    TextEntry::make('customer.email')
                        ->label('Email')
                        ->copyable(),
                    
                    TextEntry::make('customer.phone')
                        ->label('Phone'),
                ])
                ->columns(3),
            
            Section::make('Order Items')
                ->schema([
                    RepeatableEntry::make('items')
                        ->hiddenLabel()
                        ->schema([
                            TextEntry::make('product.name')
                                ->label('Product'),
                            
                            TextEntry::make('quantity')
                                ->label('Qty'),
                            
                            TextEntry::make('price')
                                ->label('Price')
                                ->money('USD'),
                            
                            TextEntry::make('total')
                                ->label('Total')
                                ->money('USD')
                                ->weight('bold'),
                        ])
                        ->columns(4)
                        ->contained(false),
                ]),
        ]);
}
```

## Advanced Features

### Entry Groups

```php
use Filament\Infolists\Components\EntryGroup;

Section::make('Contact Information')
    ->schema([
        EntryGroup::make([
            TextEntry::make('email')
                ->icon('heroicon-m-envelope'),
            TextEntry::make('phone')
                ->icon('heroicon-m-phone'),
            TextEntry::make('website')
                ->icon('heroicon-m-globe')
                ->url()
                ->openUrlInNewTab(),
        ])
            ->label('Contact Details')
            ->inlineLabel(),
    ]);
```

### Custom Entries

```php
use Filament\Infolists\Components\Entry;

Entry::make('custom')
    ->label('Custom Data')
    ->view('filament.infolists.entries.custom')
    ->viewData([
        'extra' => 'data',
    ]);
```

### Relationship Data

```php
TextEntry::make('author.name')
    ->label('Author')
    ->url(fn ($record) => UserResource::getUrl('view', ['record' => $record->author]))
    ->openUrlInNewTab();

TextEntry::make('tags.name')
    ->badge()
    ->separator(',');
```

### Conditional Visibility

```php
Section::make('Premium Features')
    ->visible(fn ($record) => $record->isPremium())
    ->schema([
        TextEntry::make('premium_feature_1'),
        TextEntry::make('premium_feature_2'),
    ]);

TextEntry::make('admin_notes')
    ->visible(fn () => auth()->user()->isAdmin());
```

### State Formatting

```php
TextEntry::make('price')
    ->formatStateUsing(fn ($state) => '$' . number_format($state, 2));

TextEntry::make('tags')
    ->formatStateUsing(function ($state) {
        return collect($state)->pluck('name')->join(', ');
    });

TextEntry::make('status')
    ->formatStateUsing(fn ($state) => ucfirst($state));
```

## Using Infolists in Resources

### Resource Infolist

```php
class PostResource extends Resource
{
    public static function infolist(Infolist $infolist): Infolist
    {
        return $infolist
            ->schema([
                // ... entries
            ]);
    }
    
    public static function getPages(): array
    {
        return [
            'index' => Pages\ListPosts::class,
            'create' => Pages\CreatePost::class,
            'view' => Pages\ViewPost::class,  // Uses infolist
            'edit' => Pages\EditPost::class,
        ];
    }
}
```

### Custom Page Infolist

```php
<?php

namespace App\Filament\Pages;

use Filament\Infolists\Components\TextEntry;
use Filament\Infolists\Infolist;
use Filament\Pages\Page;

class Dashboard extends Page
{
    protected static string $view = 'filament.pages.dashboard';
    
    public ?array $data = [];
    
    public function mount(): void
    {
        $this->data = [
            'total_users' => User::count(),
            'total_orders' => Order::count(),
        ];
    }
    
    public function infolist(Infolist $infolist): Infolist
    {
        return $infolist
            ->state($this->data)
            ->schema([
                TextEntry::make('total_users')
                    ->label('Total Users'),
                TextEntry::make('total_orders')
                    ->label('Total Orders'),
            ]);
    }
}
```

### Infolist in Actions

```php
Action::make('viewDetails')
    ->infolist([
        TextEntry::make('name'),
        TextEntry::make('email'),
    ])
    ->record($user)
    ->modalWidth(MaxWidth::Large);
```

## Best Practices

1. **Use sections to group related data** - Organize logically
2. **Leverage icons** - Enhance visual recognition
3. **Format dates consistently** - Use `since()` or custom formats
4. **Copyable fields** - Enable for emails, IDs, URLs
5. **Use badges for statuses** - Visual state indicators
6. **Collapsible sections** - For less important data
7. **Responsive layouts** - Use responsive grids
8. **Hide empty fields** - Use `hidden()` or `placeholder()`
9. **Link related resources** - Connect to other pages
10. **Keep it scannable** - Use bold labels, clear hierarchy

## Additional Resources

- [Official Infolists Documentation](https://filamentphp.com/docs/5.x/infolists)
- [Entry Components](https://filamentphp.com/docs/5.x/infolists/entries)
- [Layout Components](https://filamentphp.com/docs/5.x/infolists/layout)
