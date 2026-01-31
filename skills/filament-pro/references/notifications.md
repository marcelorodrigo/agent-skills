# Notifications Reference

Complete guide for sending notifications in Filament v5.

## Overview

Filament provides a powerful notification system for sending flash messages, database notifications, and broadcast notifications. Notifications can include actions, custom styling, and can be sent to specific users.

## Basic Notifications

### Simple Notifications

```php
use Filament\Notifications\Notification;

// Success notification
Notification::make()
    ->title('Saved successfully')
    ->success()
    ->send();

// Error notification
Notification::make()
    ->title('Error occurred')
    ->body('Unable to save the record.')
    ->danger()
    ->send();

// Warning notification
Notification::make()
    ->title('Warning')
    ->body('This action cannot be undone.')
    ->warning()
    ->send();

// Info notification
Notification::make()
    ->title('Information')
    ->body('New updates are available.')
    ->info()
    ->send();
```

### Notification Types

| Method | Color | Use Case |
|--------|-------|----------|
| `success()` | Green | Successful operations |
| `danger()` | Red | Errors and failures |
| `warning()` | Yellow | Warnings and cautions |
| `info()` | Blue | Informational messages |
| `secondary()` | Gray | Neutral messages |

### Notification Properties

```php
Notification::make()
    ->title('Title here')           // Main heading
    ->body('Description here')      // Detailed message
    ->icon('heroicon-o-check')      // Custom icon
    ->iconColor('success')          // Icon color
    ->duration(5000)                // Auto-dismiss after ms (default: 5000)
    ->persistent()                  // Don't auto-dismiss
    ->send();
```

## Notifications with Actions

### Basic Actions

```php
use Filament\Actions\Action;

Notification::make()
    ->title('Order placed successfully')
    ->success()
    ->body('Your order #12345 has been confirmed.')
    ->actions([
        Action::make('view')
            ->button()
            ->url(route('orders.show', $order))
            ->openUrlInNewTab(),
        
        Action::make('undo')
            ->color('gray')
            ->close()
            ->action(function () {
                // Undo logic
            }),
    ])
    ->send();
```

### Action Types

```php
Notification::make()
    ->actions([
        // Button style action
        Action::make('view')
            ->button()
            ->url('/orders/123'),
        
        // Link style action
        Action::make('dismiss')
            ->link()
            ->close(),
        
        // Action with dispatch
        Action::make('refresh')
            ->dispatch('refreshData'),
        
        // Action with color
        Action::make('delete')
            ->color('danger')
            ->requiresConfirmation()
            ->action(fn () => $record->delete()),
    ])
    ->send();
```

## Database Notifications

### Sending to Database

Database notifications are stored and displayed in the notification center:

```php
// Send to specific user
Notification::make()
    ->title('New comment on your post')
    ->body('John Doe commented on "My First Post"')
    ->actions([
        Action::make('view')
            ->url(route('posts.show', $post)),
    ])
    ->sendToDatabase($user);

// Send to multiple users
$users = User::where('role', 'admin')->get();

Notification::make()
    ->title('System maintenance scheduled')
    ->body('Maintenance will occur tonight at 2 AM.')
    ->sendToDatabase($users);
```

### Marking as Read

```php
// User marks notification as read
$user->notifications()->find($notificationId)->markAsRead();

// Mark all as read
$user->unreadNotifications->markAsRead();
```

## Broadcast Notifications

### Broadcasting to Users

Requires Laravel Echo and a broadcast driver (Pusher, Ably, etc.):

```php
// Send broadcast notification
Notification::make()
    ->title('New order received')
    ->body('Order #12345 needs processing.')
    ->broadcast($user);
```

### Using in Laravel Notifications

```php
use App\Models\User;
use Filament\Notifications\Notification;
use Illuminate\Notifications\Messages\BroadcastMessage;

class OrderReceived extends Notification
{
    public function toBroadcast(User $notifiable): BroadcastMessage
    {
        return Notification::make()
            ->title('New order received')
            ->body('Order #12345 needs processing.')
            ->getBroadcastMessage();
    }
    
    public function via($notifiable): array
    {
        return ['broadcast', 'database'];
    }
}
```

## Notification in Actions

### After Action Completion

```php
use Filament\Actions\Action;
use Filament\Notifications\Notification;

Action::make('publish')
    ->action(function (Post $record) {
        $record->update(['status' => 'published']);
        
        Notification::make()
            ->title('Post published')
            ->success()
            ->send();
    })
    ->successNotification(
        Notification::make()
            ->title('Published successfully')
            ->success()
    );
```

### Custom Notification in Actions

```php
Action::make('process')
    ->action(function () {
        try {
            $this->processData();
            
            Notification::make()
                ->title('Processing complete')
                ->success()
                ->send();
        } catch (Exception $e) {
            Notification::make()
                ->title('Processing failed')
                ->body($e->getMessage())
                ->danger()
                ->persistent()
                ->send();
        }
    });
```

## Notification Center

### Enabling Notification Center

Add to your PanelProvider:

```php
use Filament\Notifications\Livewire\DatabaseNotifications;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->databaseNotifications()
        ->databaseNotificationsPolling('30s');
}
```

### Customizing Notification Center

```php
// Change polling interval
->databaseNotificationsPolling('60s')

// Disable polling
->databaseNotificationsPolling(null)
```

## Custom Notification Views

### Blade Component

Create `resources/views/vendor/filament/components/notification.blade.php`:

```blade
<x-filament::notification
    :notification="$notification"
    class="custom-notification"
>
    {{ $slot }}
</x-filament::notification>
```

### Custom Styling

```css
/* Add to your CSS */
.filament-notification {
    @apply rounded-lg shadow-lg;
}

.filament-notification.success {
    @apply border-l-4 border-green-500;
}
```

## Advanced Examples

### Complex Notification

```php
Notification::make()
    ->title('Export completed')
    ->icon('heroicon-o-document-arrow-down')
    ->iconColor('success')
    ->body('Your data export is ready for download.')
    ->actions([
        Action::make('download')
            ->button()
            ->color('primary')
            ->icon('heroicon-o-arrow-down-tray')
            ->url(fn () => Storage::url('exports/data.csv'))
            ->openUrlInNewTab(),
        
        Action::make('dismiss')
            ->link()
            ->close()
            ->label('Dismiss'),
    ])
    ->persistent()
    ->send();
```

### Notification with Loading State

```php
Action::make('generateReport')
    ->action(function () {
        Notification::make()
            ->title('Generating report...')
            ->body('This may take a few minutes.')
            ->persistent()
            ->send();
        
        // Long running task
        $report = $this->generateReport();
        
        Notification::make()
            ->title('Report generated')
            ->success()
            ->actions([
                Action::make('download')
                    ->url($report->downloadUrl()),
            ])
            ->send();
    });
```

### Conditional Notifications

```php
public function save()
{
    $result = $this->form->save();
    
    if ($result->successful) {
        Notification::make()
            ->title('Saved successfully')
            ->success()
            ->send();
    } else {
        Notification::make()
            ->title('Save failed')
            ->body($result->errorMessage)
            ->danger()
            ->persistent()
            ->actions([
                Action::make('retry')
                    ->button()
                    ->action(fn () => $this->save()),
            ])
            ->send();
    }
}
```

## Testing Notifications

```php
use function Pest\Livewire\livewire;

it('shows success notification after create', function () {
    livewire(CreatePost::class)
        ->fillForm(['title' => 'Test'])
        ->call('create')
        ->assertNotified('Post created successfully');
});

it('shows custom notification', function () {
    livewire(CreatePost::class)
        ->callAction('publish')
        ->assertNotified(
            Notification::make()
                ->title('Published!')
                ->success()
        );
});
```

## Best Practices

1. **Keep messages concise** - Title should be brief, use body for details
2. **Use appropriate types** - Success for completions, danger for errors
3. **Add actions when helpful** - Link to affected resources
4. **Use persistent for important** - Don't auto-dismiss critical messages
5. **Send database notifications** - For actions requiring later attention
6. **Test notification flow** - Verify users receive proper feedback
7. **Don't over-notify** - Too many notifications reduce effectiveness
8. **Use icons appropriately** - Enhance recognition with relevant icons
9. **Consider mobile** - Ensure notifications work well on small screens
10. **Localize messages** - Use translation keys for multi-language support

## Additional Resources

- [Official Notifications Documentation](https://filamentphp.com/docs/5.x/notifications/overview)
- [Database Notifications](https://filamentphp.com/docs/5.x/notifications/database-notifications)
- [Broadcast Notifications](https://filamentphp.com/docs/5.x/notifications/broadcast-notifications)
