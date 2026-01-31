# Actions Reference

Complete guide for creating actions, modals, and notifications in Filament v5.

## Action Types

| Action | Class | Description |
|--------|-------|-------------|
| Create | `CreateAction` | Create a new record |
| Edit | `EditAction` | Edit existing record |
| View | `ViewAction` | View record details |
| Delete | `DeleteAction` | Delete a record |
| Replicate | `ReplicateAction` | Duplicate a record |
| Restore | `RestoreAction` | Restore soft-deleted record |
| Force Delete | `ForceDeleteAction` | Permanently delete |
| Import | `ImportAction` | Import from file |
| Export | `ExportAction` | Export to file |

## Basic Actions

```php
use Filament\Actions\Action;

// Simple action
Action::make('save')
    ->label('Save Changes')
    ->icon('heroicon-m-check')
    ->color('primary')
    ->action(fn () => $this->save())

// URL action
Action::make('visit')
    ->label('Visit Website')
    ->icon('heroicon-m-arrow-top-right-on-square')
    ->url(fn (Post $record) => $record->website)
    ->openUrlInNewTab()

// Hidden action
Action::make('publish')
    ->hidden(fn (Post $record): bool => $record->isPublished())

// Disabled action
Action::make('delete')
    ->disabled(fn (Post $record): bool => ! auth()->user()->can('delete', $record))

// Visible action
Action::make('edit')
    ->visible(fn (Post $record): bool => auth()->user()->can('edit', $record))
```

## Action Configuration

### Icons and Colors

```php
Action::make('approve')
    ->icon('heroicon-m-check-circle')
    ->iconPosition('before')  // or 'after'
    ->iconSize('sm')  // 'sm', 'md', 'lg'
    ->color('success')  // primary, success, danger, warning, info, gray
    ->outlined()  // Outlined button style
    ->link()  // Link style
```

### Labels and Tooltips

```php
Action::make('delete')
    ->label('Delete Record')
    ->tooltip('Permanently delete this record')
    ->helperText('This action cannot be undone')
```

### Authorization

```php
Action::make('edit')
    ->authorize('update', $this->post)
    ->authorize(fn (): bool => auth()->user()->can('update', $this->post))
```

## Modal Actions

### Basic Modal

```php
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\Textarea;

Action::make('sendEmail')
    ->icon('heroicon-m-envelope')
    ->form([
        TextInput::make('subject')
            ->required()
            ->maxLength(255),
        Textarea::make('body')
            ->required()
            ->columnSpanFull(),
    ])
    ->action(function (array $data, Post $record) {
        Mail::to($record->author->email)
            ->send(new PostNotification($data['subject'], $data['body']));
    })
    ->successNotification(
        Notification::make()
            ->title('Email sent')
            ->success()
    )
```

### Confirmation Modal

```php
Action::make('delete')
    ->color('danger')
    ->icon('heroicon-m-trash')
    ->requiresConfirmation()
    ->modalHeading('Delete post')
    ->modalDescription('Are you sure you want to delete this post? This action cannot be undone.')
    ->modalSubmitActionLabel('Yes, delete it')
    ->modalCancelActionLabel('Cancel')
    ->action(fn (Post $record) => $record->delete())
```

### Modal Customization

```php
Action::make('edit')
    ->modalHeading('Edit Customer')
    ->modalDescription('Update customer information')
    ->modalWidth(MaxWidth::Large)  // Small, Medium, Large, ExtraLarge, TwoExtraLarge, ThreeExtraLarge, FourExtraLarge, FiveExtraLarge, Screen
    ->modalIcon('heroicon-m-pencil-square')
    ->modalIconColor('primary')
    ->modalFooterActionsAlignment(Alignment::Right)
    ->modalAutofocus()
    ->closeModalByClickingAway(false)
    ->closeModalByPressingEscape(false)
```

### Wizard Modal

```php
use Filament\Forms\Components\Wizard;

Action::make('createOrder')
    ->steps([
        Wizard\Step::make('Customer')
            ->icon('heroicon-m-user')
            ->description('Select customer')
            ->schema([
                Forms\Components\Select::make('customer_id')
                    ->relationship('customer', 'name')
                    ->searchable()
                    ->required(),
            ]),
        
        Wizard\Step::make('Products')
            ->icon('heroicon-m-shopping-bag')
            ->description('Add products')
            ->schema([
                Forms\Components\Repeater::make('items')
                    ->schema([
                        Forms\Components\Select::make('product_id')
                            ->relationship('product', 'name')
                            ->searchable()
                            ->required(),
                        Forms\Components\TextInput::make('quantity')
                            ->numeric()
                            ->required()
                            ->default(1),
                    ])
                    ->columns(2),
            ]),
        
        Wizard\Step::make('Review')
            ->icon('heroicon-m-eye')
            ->description('Review order')
            ->schema([
                Forms\Components\Placeholder::make('summary')
                    ->content('Review your order before submitting.'),
            ]),
    ])
    ->action(function (array $data) {
        Order::create($data);
    })
    ->modalWidth(MaxWidth::ExtraLarge)
```

## Notifications

### Basic Notifications

```php
use Filament\Notifications\Notification;

Notification::make()
    ->title('Saved successfully')
    ->success()
    ->send();

Notification::make()
    ->title('Error occurred')
    ->body('Unable to save the record.')
    ->danger()
    ->send();

Notification::make()
    ->title('Warning')
    ->body('This action cannot be undone.')
    ->warning()
    ->persistent()
    ->send();

Notification::make()
    ->title('Information')
    ->body('New updates are available.')
    ->info()
    ->send();
```

### Notification with Actions

```php
Notification::make()
    ->title('Order placed successfully')
    ->success()
    ->body('Your order #12345 has been confirmed.')
    ->actions([
        Notification\Action::make('view')
            ->button()
            ->url('/orders/12345')
            ->openUrlInNewTab(),
        Notification\Action::make('undo')
            ->color('gray')
            ->close()
            ->action(function () {
                // Undo logic
            }),
    ])
    ->send();
```

### Notification to Specific User

```php
Notification::make()
    ->title('New comment')
    ->body('Someone commented on your post.')
    ->sendToDatabase($user);
```

### Notification Icons

```php
Notification::make()
    ->title('Success!')
    ->icon('heroicon-o-check-circle')
    ->iconColor('success')
    ->success()
    ->send();
```

## Action Groups

```php
use Filament\Actions\ActionGroup;

ActionGroup::make([
    Action::make('view')
        ->icon('heroicon-m-eye')
        ->url(fn (Post $record): string => route('posts.show', $record)),
    
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
    ->color('gray')
    ->button()  // Render as button group
    ->tooltip('More actions')
```

## Bulk Actions

```php
use Filament\Actions\BulkAction;
use Filament\Actions\BulkActionGroup;

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
                    ->options([
                        'draft' => 'Draft',
                        'published' => 'Published',
                        'archived' => 'Archived',
                    ])
                    ->required(),
            ])
            ->action(fn (Collection $records, array $data) => 
                $records->each->update(['status' => $data['status']])
            ),
        
        BulkAction::make('export')
            ->icon('heroicon-m-arrow-down-tray')
            ->action(function (Collection $records) {
                $csv = $records->toCsv();
                return response()->streamDownload(function () use ($csv) {
                    echo $csv;
                }, 'export.csv');
            }),
    ]),
])
```

## Header Actions

```php
protected function getHeaderActions(): array
{
    return [
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
                Notification::make()
                    ->title('Import complete')
                    ->success()
                    ->send();
            }),
        
        Action::make('settings')
            ->icon('heroicon-m-cog-6-tooth')
            ->url('/admin/settings'),
    ];
}
```

## Page Actions

### In Resource Pages

```php
// Create page
protected function getHeaderActions(): array
{
    return [
        Actions\CreateAction::make(),
    ];
}

// Edit page
protected function getHeaderActions(): array
{
    return [
        Actions\ViewAction::make(),
        Actions\DeleteAction::make(),
    ];
}

// View page
protected function getHeaderActions(): array
{
    return [
        Actions\EditAction::make(),
        Actions\DeleteAction::make(),
    ];
}

// List page
protected function getHeaderActions(): array
{
    return [
        Actions\CreateAction::make(),
    ];
}
```

## Action Hooks

```php
Action::make('delete')
    ->before(function () {
        // Run before action
        Log::info('Deleting record...');
    })
    ->action(function (Post $record) {
        $record->delete();
    })
    ->after(function () {
        // Run after action
        Notification::make()
            ->title('Deleted successfully')
            ->success()
            ->send();
    })
```

## Complex Action Example

```php
Action::make('processRefund')
    ->icon('heroicon-m-arrow-uturn-left')
    ->color('warning')
    ->requiresConfirmation()
    ->modalHeading('Process Refund')
    ->modalDescription('Are you sure you want to refund this order?')
    ->modalSubmitActionLabel('Yes, refund')
    ->modalWidth(MaxWidth::Large)
    ->form([
        Section::make('Refund Details')
            ->schema([
                TextInput::make('amount')
                    ->numeric()
                    ->required()
                    ->prefix('$')
                    ->maxValue(fn (Order $record) => $record->total),
                
                Select::make('reason')
                    ->options([
                        'customer_request' => 'Customer Request',
                        'damaged_item' => 'Damaged Item',
                        'wrong_item' => 'Wrong Item',
                        'other' => 'Other',
                    ])
                    ->required(),
                
                Textarea::make('notes')
                    ->columnSpanFull(),
            ]),
    ])
    ->action(function (array $data, Order $record) {
        // Process refund
        $refund = $record->refunds()->create([
            'amount' => $data['amount'],
            'reason' => $data['reason'],
            'notes' => $data['notes'],
            'processed_by' => auth()->id(),
        ]);
        
        // Update order status
        $record->update([
            'status' => 'refunded',
            'refunded_amount' => $record->refunded_amount + $data['amount'],
        ]);
        
        // Send notification
        Notification::make()
            ->title('Refund processed')
            ->body("Refund #{$refund->id} for {$data['amount']} has been processed.")
            ->success()
            ->actions([
                Notification\Action::make('view')
                    ->button()
                    ->url("/admin/refunds/{$refund->id}"),
            ])
            ->send();
    })
    ->visible(fn (Order $record): bool => 
        $record->status === 'completed' && 
        $record->refunded_amount < $record->total
    )
    ->authorize(fn (Order $record): bool => 
        auth()->user()->can('process refunds')
    )
```

## Import Action

```php
use Filament\Actions\ImportAction;
use Filament\Forms\Components\FileUpload;

Action::make('import')
    ->icon('heroicon-m-arrow-up-tray')
    ->form([
        FileUpload::make('file')
            ->acceptedFileTypes([
                'application/vnd.ms-excel',
                'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
                'text/csv',
            ])
            ->required(),
    ])
    ->action(function (array $data) {
        $import = new ProductsImport();
        Excel::import($import, $data['file']);
        
        Notification::make()
            ->title('Import complete')
            ->body("{$import->getRowCount()} products imported successfully.")
            ->success()
            ->send();
    })
```

## Export Action

```php
use Filament\Actions\ExportAction;

Action::make('export')
    ->icon('heroicon-m-arrow-down-tray')
    ->form([
        Select::make('format')
            ->options([
                'csv' => 'CSV',
                'xlsx' => 'Excel',
                'pdf' => 'PDF',
            ])
            ->default('csv'),
        
        Toggle::make('include_headers')
            ->default(true),
    ])
    ->action(function (array $data) {
        $format = $data['format'];
        $filename = "products-{$format}." . ($format === 'xlsx' ? 'xlsx' : $format);
        
        return match ($format) {
            'csv' => response()->streamDownload(fn () => Product::toCsv(), $filename),
            'xlsx' => Excel::download(new ProductsExport(), $filename),
            'pdf' => PDF::loadView('exports.products', ['products' => Product::all()])->download($filename),
        };
    })
```

## Best Practices

1. **Use appropriate colors** - Danger for destructive, Success for positive
2. **Always confirm destructive actions** - Use requiresConfirmation()
3. **Provide clear labels** - Action should describe what it does
4. **Use icons** - Enhance visual recognition
5. **Handle errors gracefully** - Catch exceptions and show notifications
6. **Authorize actions** - Check permissions before allowing
7. **Give feedback** - Always send notifications after actions
8. **Use action groups** - Group related actions together
9. **Customize modals** - Set appropriate headings and descriptions
10. **Optimize bulk actions** - Use database transactions for efficiency
11. **Show loading states** - For long-running actions
12. **Log important actions** - For audit trails
13. **Test edge cases** - Empty data, large datasets, errors
14. **Use success notifications** - Confirm actions completed
15. **Keep actions focused** - One action should do one thing

## Tips & Tricks

### Action with Loading State

```php
Action::make('sync')
    ->action(function () {
        // Long-running operation
        $this->syncData();
    })
    ->requiresConfirmation()
    ->modalHeading('Sync Data')
    ->modalDescription('This may take a few minutes...')
```

### Dynamic Action Label

```php
Action::make('toggleStatus')
    ->label(fn (Post $record): string => $record->is_published ? 'Unpublish' : 'Publish')
    ->icon(fn (Post $record): string => $record->is_published ? 'heroicon-m-x-circle' : 'heroicon-m-check-circle')
    ->color(fn (Post $record): string => $record->is_published ? 'danger' : 'success')
    ->action(fn (Post $record) => $record->update(['is_published' => ! $record->is_published]))
```

### Conditional Confirmation

```php
Action::make('delete')
    ->requiresConfirmation(fn (Post $record): bool => $record->comments()->count() > 0)
    ->modalHeading('Delete Post')
    ->modalDescription(fn (Post $record): string => 
        $record->comments()->count() > 0 
            ? "This post has {$record->comments()->count()} comments. Are you sure?"
            : 'Are you sure you want to delete this post?'
    )
```

## Additional Resources

- [Official Actions Documentation](https://filamentphp.com/docs/5.x/actions/overview)
- [Notifications](https://filamentphp.com/docs/5.x/notifications/overview)
