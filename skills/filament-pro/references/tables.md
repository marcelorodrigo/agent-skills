# Table Components Reference

Complete reference for Filament v5 table columns, filters, and actions.

## Column Types

Filament provides 10+ column types in the `Filament\Tables\Columns` namespace:

| Column Type | Class | Description |
|-------------|-------|-------------|
| Text | `TextColumn` | Text display with formatting |
| Icon | `IconColumn` | Icon/boolean display |
| Image | `ImageColumn` | Image thumbnails |
| Badge | `BadgeColumn` | Colored badge |
| Color | `ColorColumn` | Color swatch |
| Select | `SelectColumn` | Editable dropdown |
| Toggle | `ToggleColumn` | Editable toggle |
| Checkbox | `CheckboxColumn` | Editable checkbox |
| Text Input | `TextInputColumn` | Editable text field |
| Summarizers | Various | Aggregate data (sum, avg, count) |

## Text Column

```php
use Filament\Tables\Columns\TextColumn;

TextColumn::make('name')
    ->label('Full Name')
    ->searchable()
    ->sortable()
    ->toggleable()
    ->toggleable(isToggledHiddenByDefault: true)
    ->weight('font-bold')
    ->color('primary')
    ->icon('heroicon-m-user')
    ->iconPosition('before')
    ->iconColor('success')
    ->url(fn ($record) => route('users.show', $record))
    ->openUrlInNewTab()
    ->copyable()
    ->copyMessage('Copied!')
    ->limit(50)
    ->tooltip(fn ($record): string => $record->description)
    ->wrap()
    ->alignLeft()
    ->alignCenter()
    ->alignRight()
    ->placeholder('N/A')
    ->prefix('$')
    ->suffix('.00')
    ->formatStateUsing(fn ($state) => strtoupper($state))
```

### Date & Number Formatting

```php
TextColumn::make('price')
    ->money('USD')
    ->sortable()

TextColumn::make('quantity')
    ->numeric(decimalPlaces: 0)
    ->suffix(' items')

TextColumn::make('percentage')
    ->numeric(decimalPlaces: 2)
    ->suffix('%')

TextColumn::make('created_at')
    ->dateTime('M j, Y H:i')
    ->sortable()

TextColumn::make('published_at')
    ->date('M j, Y')
    ->placeholder('Not published')

TextColumn::make('updated_at')
    ->since()  // "2 hours ago"

TextColumn::make('size')
    ->bytes()

TextColumn::make('download_count')
    ->numeric()
    ->summarize(Sum::make()->label('Total Downloads'))
```

## Badge Column

```php
use Filament\Tables\Columns\BadgeColumn;

BadgeColumn::make('status')
    ->color(fn (string $state): string => match ($state) {
        'draft' => 'gray',
        'pending' => 'warning',
        'published' => 'success',
        'rejected' => 'danger',
        default => 'gray',
    })
    ->icon(fn (string $state): string => match ($state) {
        'published' => 'heroicon-m-check-badge',
        'pending' => 'heroicon-m-clock',
        default => null,
    })
    ->iconPosition('before')
```

## Icon Column

```php
use Filament\Tables\Columns\IconColumn;

IconColumn::make('is_active')
    ->boolean()
    ->trueIcon('heroicon-o-check-circle')
    ->falseIcon('heroicon-o-x-circle')
    ->trueColor('success')
    ->falseColor('danger')

IconColumn::make('status')
    ->icon(fn (string $state): string => match ($state) {
        'active' => 'heroicon-m-check-circle',
        'inactive' => 'heroicon-m-x-circle',
        default => 'heroicon-m-question-mark-circle',
    })
    ->color(fn (string $state): string => match ($state) {
        'active' => 'success',
        'inactive' => 'danger',
        default => 'gray',
    })
```

## Image Column

```php
use Filament\Tables\Columns\ImageColumn;

ImageColumn::make('avatar')
    ->disk('public')
    ->square()
    ->circular()
    ->size(50)
    ->width(100)
    ->height(100)
    ->defaultImageUrl(fn ($record) => 'https://ui-avatars.com/api/?name=' . urlencode($record->name))
    ->checkFileExistence()
```

## Color Column

```php
use Filament\Tables\Columns\ColorColumn;

ColorColumn::make('color')
    ->copyable()
```

## Select Column (Editable)

```php
use Filament\Tables\Columns\SelectColumn;

SelectColumn::make('status')
    ->options([
        'draft' => 'Draft',
        'published' => 'Published',
    ])
    ->selectablePlaceholder(false)
    ->disablePlaceholderSelection()
```

## Toggle Column (Editable)

```php
use Filament\Tables\Columns\ToggleColumn;

ToggleColumn::make('is_featured')
    ->onColor('success')
    ->offColor('danger')
    ->onIcon('heroicon-m-check')
    ->offIcon('heroicon-m-x-mark')
```

## Checkbox Column (Editable)

```php
use Filament\Tables\Columns\CheckboxColumn;

CheckboxColumn::make('is_approved')
```

## Text Input Column (Editable)

```php
use Filament\Tables\Columns\TextInputColumn;

TextInputColumn::make('sort_order')
    ->type('number')
    ->rules(['required', 'integer', 'min:0'])
```

## Summarizers

```php
use Filament\Tables\Columns\Summarizers\Sum;
use Filament\Tables\Columns\Summarizers\Average;
use Filament\Tables\Columns\Summarizers\Count;
use Filament\Tables\Columns\Summarizers\Range;

TextColumn::make('price')
    ->money('USD')
    ->summarize([
        Sum::make()->label('Total'),
        Average::make()->label('Average'),
    ])

TextColumn::make('quantity')
    ->numeric()
    ->summarize([
        Sum::make(),
        Range::make()->label('Range'),
    ])

TextColumn::make('id')
    ->label('Count')
    ->summarize(Count::make())
```

## Filters

### Basic Filters

```php
use Filament\Tables\Filters\Filter;
use Filament\Tables\Filters\SelectFilter;
use Filament\Tables\Filters\TernaryFilter;
use Filament\Tables\Filters\QueryBuilder;
use Illuminate\Database\Eloquent\Builder;

// Toggle filter
Filter::make('is_featured')
    ->query(fn (Builder $query) => $query->where('is_featured', true))
    ->toggle()
    ->label('Featured only')
    
// Select filter
SelectFilter::make('status')
    ->options([
        'draft' => 'Draft',
        'pending' => 'Pending',
        'published' => 'Published',
    ])
    ->multiple()
    ->searchable()
    ->preload()
    ->native(false)
    
// Ternary filter (yes/no/any)
TernaryFilter::make('email_verified_at')
    ->label('Email verified')
    ->placeholder('Any')
    ->trueLabel('Verified')
    ->falseLabel('Not verified')
    ->native(false)
    
// Relationship filter
SelectFilter::make('category')
    ->relationship('category', 'name')
    ->searchable()
    ->preload()
    ->multiple()
```

### Filter with Form

```php
use Filament\Forms\Components\DatePicker;

Filter::make('created_at')
    ->form([
        DatePicker::make('created_from'),
        DatePicker::make('created_until'),
    ])
    ->query(function (Builder $query, array $data): Builder {
        return $query
            ->when(
                $data['created_from'],
                fn (Builder $query, $date): Builder => $query->whereDate('created_at', '>=', $date),
            )
            ->when(
                $data['created_until'],
                fn (Builder $query, $date): Builder => $query->whereDate('created_at', '<=', $date),
            );
    })
```

### Filter Groups

```php
use Filament\Tables\Filters\FilterGroup;

FilterGroup::make('Status', [
    Filter::make('draft')
        ->query(fn ($query) => $query->where('status', 'draft')),
    Filter::make('published')
        ->query(fn ($query) => $query->where('status', 'published')),
    Filter::make('archived')
        ->query(fn ($query) => $query->where('status', 'archived')),
])
```

## Table Actions

### Record Actions

```php
use Filament\Actions\Action;
use Filament\Actions\EditAction;
use Filament\Actions\DeleteAction;
use Filament\Actions\ViewAction;
use Filament\Actions\ReplicateAction;
use Filament\Actions\ForceDeleteAction;
use Filament\Actions\RestoreAction;
use Filament\Notifications\Notification;

->actions([
    ViewAction::make(),
    EditAction::make(),
    DeleteAction::make(),
    
    // Custom action
    Action::make('approve')
        ->icon('heroicon-m-check-circle')
        ->color('success')
        ->requiresConfirmation()
        ->modalHeading('Approve post')
        ->modalDescription('Are you sure you want to approve this post?')
        ->modalSubmitActionLabel('Yes, approve')
        ->action(function (Post $record) {
            $record->update(['status' => 'approved']);
            
            Notification::make()
                ->title('Post approved')
                ->success()
                ->send();
        })
        ->visible(fn (Post $record): bool => $record->status === 'pending'),
        
    // Action with modal form
    Action::make('sendEmail')
        ->icon('heroicon-m-envelope')
        ->form([
            TextInput::make('subject')->required(),
            RichEditor::make('body')->required(),
        ])
        ->action(function (array $data, Post $record) {
            Mail::to($record->author->email)
                ->send(new PostNotification($data['subject'], $data['body']));
        })
        ->successNotificationTitle('Email sent'),
    
    // Open URL
    Action::make('preview')
        ->icon('heroicon-m-eye')
        ->url(fn (Post $record): string => route('posts.preview', $record))
        ->openUrlInNewTab(),
        
    // Action group
    ActionGroup::make([
        Action::make('edit')
            ->icon('heroicon-m-pencil-square')
            ->url(fn (Post $record): string => route('posts.edit', $record)),
        Action::make('duplicate')
            ->icon('heroicon-m-document-duplicate')
            ->action(fn (Post $record) => $record->replicate()->save()),
        Action::make('delete')
            ->icon('heroicon-m-trash')
            ->color('danger')
            ->requiresConfirmation()
            ->action(fn (Post $record) => $record->delete()),
    ])
        ->label('Actions')
        ->icon('heroicon-m-ellipsis-vertical')
        ->size(ActionSize::Small)
        ->color('gray'),
])
```

### Header Actions

```php
->headerActions([
    Action::make('create')
        ->label('New Post')
        ->icon('heroicon-m-plus')
        ->url(fn (): string => route('posts.create')),
        
    Action::make('import')
        ->icon('heroicon-m-arrow-up-tray')
        ->form([
            FileUpload::make('file')
                ->acceptedFileTypes(['text/csv'])
                ->required(),
        ])
        ->action(function (array $data) {
            // Import logic
        }),
])
```

### Bulk Actions

```php
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DeleteBulkAction;
use Filament\Tables\Actions\BulkAction;

->bulkActions([
    BulkActionGroup::make([
        DeleteBulkAction::make(),
        
        BulkAction::make('publish')
            ->icon('heroicon-m-check-circle')
            ->color('success')
            ->requiresConfirmation()
            ->action(fn (Collection $records) => 
                $records->each->update(['status' => 'published'])
            ),
            
        BulkAction::make('changeStatus')
            ->icon('heroicon-m-pencil-square')
            ->form([
                Select::make('status')
                    ->options(['draft' => 'Draft', 'published' => 'Published'])
                    ->required(),
            ])
            ->action(fn (Collection $records, array $data) => 
                $records->each->update(['status' => $data['status']])
            ),
            
        BulkAction::make('export')
            ->icon('heroicon-m-arrow-down-tray')
            ->action(function (Collection $records) {
                // Export logic
                return response()->download($path);
            }),
    ]),
])
```

## Table Configuration

```php
public static function table(Table $table): Table
{
    return $table
        ->columns([
            // ... columns
        ])
        ->filters([
            // ... filters
        ])
        ->actions([
            // ... actions
        ])
        ->bulkActions([
            // ... bulk actions
        ])
        ->defaultSort('created_at', 'desc')
        ->defaultPaginationPageOption(25)
        ->paginated([10, 25, 50, 100])
        ->searchable()
        ->searchPlaceholder('Search posts...')
        ->searchDebounce(500)  // ms
        ->searchOnBlur()
        ->recordClasses(fn (Post $record) => match ($record->status) {
            'draft' => 'bg-gray-50',
            'published' => null,
            default => null,
        })
        ->recordUrl(fn (Post $record): string => route('posts.edit', $record))
        ->recordAction(EditAction::class)
        ->striped()
        ->poll('30s')
        ->emptyStateHeading('No posts yet')
        ->emptyStateDescription('Create a post to get started.')
        ->emptyStateIcon('heroicon-o-document-text')
        ->emptyStateActions([
            Action::make('create')
                ->label('Create Post')
                ->url(fn (): string => route('posts.create'))
                ->icon('heroicon-m-plus'),
        ])
        ->filtersTriggerAction(fn (Action $action) => 
            $action->button()->label('Filters')
        )
        ->filtersFormColumns(2)
        ->filtersFormMaxHeight('400px')
        ->filtersLayout(Tables\Enums\FiltersLayout::AboveContent)
        ->groups([
            Group::make('category.name')
                ->titlePrefixedWithLabel(false),
            Group::make('status')
                ->collapsible(),
        ])
        ->defaultGroup('category.name')
        ->groupingSettingsInDropdownOnDesktop()
        ->groupsInDropdownOnDesktop()
        ->groupedTriggerAction(fn (Action $action) => 
            $action->button()->label('Group')
        );
}
```

## Table Layout Options

```php
// Content layout
->contentGrid([
    'md' => 2,
    'xl' => 3,
])

// Filters layout
->filtersLayout(Tables\Enums\FiltersLayout::AboveContent)
->filtersLayout(Tables\Enums\FiltersLayout::AboveContentCollapsible)
->filtersLayout(Tables\Enums\FiltersLayout::Modal)

// Actions layout
->actionsAlignment(Alignment::Left)
->actionsAlignment(Alignment::Right)
->actionsAlignment(Alignment::Center)
->actionsColumnLabel('Actions')
```

## Eloquent Query Customization

```php
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()
        ->with(['author', 'category'])  // Eager load
        ->where('status', '!=', 'archived');  // Default filter
}
```

## Selection and Records per Page

```php
// Enable record selection
->selectable()
->selectCurrentPageOnly()

// Pagination options
->paginated([10, 25, 50, 100, 'all'])
->defaultPaginationPageOption(25)
->extremePaginationLinks()  // Show first/last page links

// Simple pagination
->pagination(false)
```

## Complete Example: Product Table

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
                ->sortable()
                ->weight('font-bold'),
                
            Tables\Columns\TextColumn::make('category.name')
                ->searchable()
                ->sortable(),
                
            Tables\Columns\TextColumn::make('price')
                ->money('USD')
                ->sortable()
                ->summarize(Sum::make()->label('Total')),
                
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
                
            Tables\Columns\TextColumn::make('created_at')
                ->dateTime()
                ->sortable()
                ->toggleable(isToggledHiddenByDefault: true),
        ])
        ->filters([
            Tables\Filters\SelectFilter::make('category')
                ->relationship('category', 'name')
                ->searchable()
                ->preload(),
                
            Tables\Filters\Filter::make('low_stock')
                ->label('Low Stock')
                ->query(fn (Builder $query): Builder => $query->where('stock_quantity', '<=', 10))
                ->toggle(),
                
            Tables\Filters\Filter::make('price_range')
                ->form([
                    Forms\Components\TextInput::make('min_price')
                        ->numeric()
                        ->prefix('$'),
                    Forms\Components\TextInput::make('max_price')
                        ->numeric()
                        ->prefix('$'),
                ])
                ->query(function (Builder $query, array $data): Builder {
                    return $query
                        ->when(
                            $data['min_price'],
                            fn (Builder $query, $price): Builder => $query->where('price', '>=', $price),
                        )
                        ->when(
                            $data['max_price'],
                            fn (Builder $query, $price): Builder => $query->where('price', '<=', $price),
                        );
                }),
        ])
        ->actions([
            Tables\Actions\ViewAction::make(),
            Tables\Actions\EditAction::make(),
            
            Tables\Actions\Action::make('duplicate')
                ->icon('heroicon-m-document-duplicate')
                ->color('warning')
                ->requiresConfirmation()
                ->action(function (Product $record): void {
                    $newProduct = $record->replicate();
                    $newProduct->name = $record->name . ' (Copy)';
                    $newProduct->sku = $record->sku . '-COPY';
                    $newProduct->save();
                }),
                
            Tables\Actions\DeleteAction::make(),
        ])
        ->bulkActions([
            Tables\Actions\BulkActionGroup::make([
                Tables\Actions\DeleteBulkAction::make(),
                
                Tables\Actions\BulkAction::make('updateStock')
                    ->icon('heroicon-m-archive-box')
                    ->form([
                        Forms\Components\TextInput::make('quantity')
                            ->numeric()
                            ->required(),
                    ])
                    ->action(function ($records, array $data): void {
                        foreach ($records as $record) {
                            $record->increment('stock_quantity', $data['quantity']);
                        }
                    }),
                    
                Tables\Actions\BulkAction::make('activate')
                    ->icon('heroicon-m-check-circle')
                    ->color('success')
                    ->action(fn ($records) => $records->each->update(['is_active' => true])),
            ]),
        ])
        ->defaultSort('created_at', 'desc')
        ->poll('30s');
}
```

## Tips & Best Practices

1. **Use eager loading** in `getEloquentQuery()` to avoid N+1
2. **Add searchable()** to frequently filtered columns
3. **Use toggleable()** for less important columns
4. **Implement summarizers** for numeric data
5. **Use color()** on status columns for visual feedback
6. **Add record actions** for common operations
7. **Group related bulk actions** in BulkActionGroup
8. **Use filters** to help users find data
9. **Set defaultSort()** for consistent ordering
10. **Use poll()** for real-time data updates
11. **Add empty state actions** for better UX
12. **Format dates consistently** across the table
13. **Use icon columns** for boolean values
14. **Add tooltip()** for truncated text
15. **Test with large datasets** to ensure performance

## Additional Resources

- [Official Tables Documentation](https://filamentphp.com/docs/5.x/tables/columns)
- [Filters](https://filamentphp.com/docs/5.x/tables/filters)
- [Actions](https://filamentphp.com/docs/5.x/tables/actions)
