# Widgets Reference

Complete guide for creating dashboard widgets in Filament v5.

## Creating Widgets

```bash
# Stats overview widget
php artisan make:filament-widget StatsOverview --stats-overview

# Chart widget
php artisan make:filament-widget BlogPostsChart --chart

# Table widget
php artisan make:filament-widget LatestOrders --table

# Custom widget
php artisan make:filament-widget CustomWidget
```

## Widget Types

| Type | Command | Description |
|------|---------|-------------|
| Stats Overview | `--stats-overview` | Display statistics cards |
| Chart | `--chart` | Display Chart.js charts |
| Table | `--table` | Display data tables |
| Custom | (none) | Custom Livewire component |

## Stats Overview Widget

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Order;
use App\Models\User;
use Filament\Widgets\StatsOverviewWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class StatsOverview extends StatsOverviewWidget
{
    // Widget configuration
    protected static ?string $heading = 'Dashboard Overview';
    protected ?string $description = 'Key metrics at a glance';
    
    // Layout
    protected function getColumns(): int
    {
        return 3;
    }
    
    protected function getStats(): array
    {
        return [
            Stat::make('Total Users', User::count())
                ->description(User::where('created_at', '>=', now()->subDays(30))->count() . ' this month')
                ->descriptionIcon('heroicon-m-arrow-trending-up')
                ->chart([7, 3, 4, 5, 6, 3, 5, 8])
                ->color('success'),
            
            Stat::make('Total Orders', Order::count())
                ->description(Order::where('created_at', '>=', now()->subDays(7))->count() . ' this week')
                ->color('primary'),
            
            Stat::make('Revenue', '$' . number_format(Order::sum('total'), 2))
                ->description(Order::where('created_at', '>=', now()->subDays(30))->sum('total') > Order::whereBetween('created_at', [now()->subDays(60), now()->subDays(30)])->sum('total') ? 'Increase' : 'Decrease')
                ->descriptionIcon(fn () => Order::where('created_at', '>=', now()->subDays(30))->sum('total') > Order::whereBetween('created_at', [now()->subDays(60), now()->subDays(30)])->sum('total') ? 'heroicon-m-arrow-trending-up' : 'heroicon-m-arrow-trending-down')
                ->color(fn () => Order::where('created_at', '>=', now()->subDays(30))->sum('total') > Order::whereBetween('created_at', [now()->subDays(60), now()->subDays(30)])->sum('total') ? 'success' : 'danger'),
            
            Stat::make('Pending Orders', Order::where('status', 'pending')->count())
                ->color('warning')
                ->url(OrderResource::getUrl('index', ['tableFilters[status][value]' => 'pending'])),
        ];
    }
}
```

### Stat Options

```php
Stat::make('Label', 'value')
    ->description('Description text')
    ->descriptionIcon('heroicon-m-arrow-trending-up')
    ->icon('heroicon-m-users')
    ->chart([7, 2, 10, 3, 15, 4, 17])  // Sparkline chart
    ->color('success')  // success, danger, warning, primary, secondary
    ->url('/admin/orders')  // Clickable link
    ->extraAttributes(['class' => 'col-span-2'])
```

### Dynamic Stats with Database Data

```php
protected function getStats(): array
{
    $newUsersThisMonth = User::where('created_at', '>=', now()->subDays(30))->count();
    $newUsersLastMonth = User::whereBetween('created_at', [now()->subDays(60), now()->subDays(30)])->count();
    
    $userGrowth = $newUsersLastMonth > 0 
        ? round((($newUsersThisMonth - $newUsersLastMonth) / $newUsersLastMonth) * 100, 1)
        : 0;
    
    return [
        Stat::make('New Users (30 days)', $newUsersThisMonth)
            ->description($userGrowth > 0 ? "+{$userGrowth}% increase" : "{$userGrowth}% decrease")
            ->descriptionIcon($userGrowth > 0 ? 'heroicon-m-arrow-trending-up' : 'heroicon-m-arrow-trending-down')
            ->color($userGrowth > 0 ? 'success' : 'danger'),
    ];
}
```

## Chart Widget

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Order;
use Filament\Widgets\ChartWidget;

class OrdersChart extends ChartWidget
{
    protected static ?string $heading = 'Orders per Month';
    protected ?string $description = 'Monthly order volume';
    protected static string $color = 'primary';  // primary, success, danger, warning, info, gray
    protected static ?string $maxHeight = '300px';
    
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
                    'backgroundColor' => '#f59e0b',
                    'borderColor' => '#d97706',
                    'fill' => true,
                    'tension' => 0.3,
                ],
            ],
            'labels' => ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
        ];
    }
    
    protected function getType(): string
    {
        return 'line';  // line, bar, pie, doughnut, polarArea, radar, bubble, scatter
    }
    
    protected function getOptions(): array
    {
        return [
            'scales' => [
                'y' => [
                    'beginAtZero' => true,
                    'ticks' => [
                        'precision' => 0,
                    ],
                ],
            ],
            'plugins' => [
                'legend' => [
                    'display' => true,
                ],
            ],
        ];
    }
    
    // Optional: Add time range filters
    protected function getFilters(): ?array
    {
        return [
            'today' => 'Today',
            'week' => 'Last week',
            'month' => 'Last month',
            'year' => 'This year',
        ];
    }
    
    protected function getDataForFilter(string $filter): array
    {
        return match ($filter) {
            'today' => $this->getTodayData(),
            'week' => $this->getWeekData(),
            'month' => $this->getMonthData(),
            'year' => $this->getYearData(),
            default => $this->getYearData(),
        };
    }
}
```

### Chart Types

```php
// Line chart
protected function getType(): string
{
    return 'line';
}

// Bar chart
protected function getType(): string
{
    return 'bar';
}

// Pie chart
protected function getType(): string
{
    return 'pie';
}

// Doughnut chart
protected function getType(): string
{
    return 'doughnut';
}
```

### Chart with Trend Data

```php
use Flowframe\Trend\Trend;
use Flowframe\Trend\TrendValue;

protected function getData(): array
{
    $trend = Trend::model(Order::class)
        ->between(
            start: now()->startOfYear(),
            end: now()->endOfYear(),
        )
        ->perMonth()
        ->count();
    
    return [
        'datasets' => [
            [
                'label' => 'Orders',
                'data' => $trend->map(fn (TrendValue $value) => $value->aggregate),
                'backgroundColor' => '#36A2EB',
                'borderColor' => '#9BD0F5',
            ],
        ],
        'labels' => $trend->map(fn (TrendValue $value) => $value->date),
    ];
}
```

### Multiple Datasets

```php
protected function getData(): array
{
    return [
        'datasets' => [
            [
                'label' => 'Revenue',
                'data' => [1000, 1500, 1200, 2000, 1800, 2500],
                'backgroundColor' => '#22c55e',
                'borderColor' => '#16a34a',
            ],
            [
                'label' => 'Expenses',
                'data' => [800, 900, 1000, 1100, 1200, 1300],
                'backgroundColor' => '#ef4444',
                'borderColor' => '#dc2626',
            ],
        ],
        'labels' => ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
    ];
}
```

## Table Widget

```php
<?php

namespace App\Filament\Widgets;

use App\Filament\Resources\OrderResource;
use App\Models\Order;
use Filament\Tables;
use Filament\Tables\Table;
use Filament\Widgets\TableWidget;

class LatestOrders extends TableWidget
{
    protected int | string | array $columnSpan = 'full';
    protected static ?string $heading = 'Latest Orders';
    protected ?string $description = 'Most recent orders';
    protected static ?int $paginationPageSize = 5;
    
    public function table(Table $table): Table
    {
        return $table
            ->query(
                Order::query()
                    ->latest()
                    ->limit(10)
            )
            ->columns([
                Tables\Columns\TextColumn::make('order_number')
                    ->searchable(),
                
                Tables\Columns\TextColumn::make('customer.name')
                    ->searchable(),
                
                Tables\Columns\TextColumn::make('total')
                    ->money('USD'),
                
                Tables\Columns\TextColumn::make('status')
                    ->badge()
                    ->color(fn (string $state): string => match ($state) {
                        'pending' => 'warning',
                        'processing' => 'info',
                        'completed' => 'success',
                        'cancelled' => 'danger',
                        default => 'gray',
                    }),
                
                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime()
                    ->since(),
            ])
            ->actions([
                Tables\Actions\ViewAction::make()
                    ->url(fn (Order $record): string => OrderResource::getUrl('view', ['record' => $record])),
            ])
            ->paginated([5, 10, 25]);
    }
}
```

### Table Widget with Filters

```php
public function table(Table $table): Table
{
    return $table
        ->query(
            Order::query()
                ->when(
                    $this->filter === 'pending',
                    fn ($query) => $query->where('status', 'pending')
                )
                ->when(
                    $this->filter === 'completed',
                    fn ($query) => $query->where('status', 'completed')
                )
                ->latest()
        )
        ->columns([
            // ... columns
        ]);
}

public ?string $filter = 'all';

protected function getTableFilters(): array
{
    return [
        'all' => 'All Orders',
        'pending' => 'Pending',
        'completed' => 'Completed',
    ];
}
```

## Custom Widget

```php
<?php

namespace App\Filament\Widgets;

use Filament\Widgets\Widget;

class CustomWidget extends Widget
{
    protected static string $view = 'filament.widgets.custom-widget';
    
    protected function getViewData(): array
    {
        return [
            'data' => $this->getData(),
        ];
    }
    
    protected function getData(): array
    {
        return [
            'total' => 100,
            'increase' => 20,
        ];
    }
}
```

### Custom Widget Blade Template

```blade
{{-- resources/views/filament/widgets/custom-widget.blade.php --}}
<x-filament-widgets::widget>
    <x-filament::card>
        <div class="flex items-center justify-between">
            <div>
                <h3 class="text-lg font-medium">Custom Metric</h3>
                <p class="text-3xl font-bold">{{ $data['total'] }}</p>
                <p class="text-sm text-green-600">+{{ $data['increase'] }}%</p>
            </div>
            <div class="p-3 bg-primary-100 rounded-full">
                <x-heroicon-o-chart-bar class="w-6 h-6 text-primary-600" />
            </div>
        </div>
    </x-filament::card>
</x-filament-widgets::widget>
```

## Widget Configuration

### Column Span

Control widget width on the dashboard:

```php
// Full width
protected int | string | array $columnSpan = 'full';

// Two columns
protected int | string | array $columnSpan = 2;

// Responsive column span
protected int | string | array $columnSpan = [
    'md' => 2,
    'xl' => 3,
];
```

### Sort Order

```php
protected static ?int $sort = 2;  // Lower numbers appear first
```

### Visibility

```php
public static function canView(): bool
{
    return auth()->user()->isAdmin();
}
```

### Heading and Description

```php
protected static ?string $heading = 'Dashboard Stats';
protected ?string $description = 'Overview of key metrics';
```

### Polling

Auto-refresh widget data:

```php
protected static ?string $pollingInterval = '30s';  // 30 seconds
protected static ?string $pollingInterval = '1m';   // 1 minute
protected static ?string $pollingInterval = null;   // Disable polling
```

## Dashboard Layout

Configure dashboard in your PanelProvider:

```php
use App\Filament\Widgets\StatsOverview;
use App\Filament\Widgets\OrdersChart;
use App\Filament\Widgets\LatestOrders;
use Filament\Pages\Dashboard;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->widgets([
            StatsOverview::class,
            OrdersChart::class,
            LatestOrders::class,
        ])
        ->pages([
            Dashboard::class,
        ]);
}
```

### Custom Dashboard

```php
<?php

namespace App\Filament\Pages;

use Filament\Pages\Dashboard as BaseDashboard;

class Dashboard extends BaseDashboard
{
    protected static ?string $navigationIcon = 'heroicon-o-home';
    protected static ?string $navigationLabel = 'Dashboard';
    protected static ?int $navigationSort = -2;
    
    // Custom columns
    public function getColumns(): int | array
    {
        return [
            'default' => 1,
            'sm' => 2,
            'md' => 3,
            'lg' => 4,
        ];
    }
    
    // Filter visible widgets
    public function getWidgets(): array
    {
        return [
            \App\Filament\Widgets\StatsOverview::class,
            \App\Filament\Widgets\OrdersChart::class,
            \App\Filament\Widgets\LatestOrders::class,
        ];
    }
}
```

## Complete Example: Dashboard with Multiple Widgets

```php
<?php

// app/Filament/Widgets/OrderStats.php
namespace App\Filament\Widgets;

use App\Models\Order;
use Filament\Widgets\StatsOverviewWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class OrderStats extends StatsOverviewWidget
{
    protected function getStats(): array
    {
        $todayRevenue = Order::whereDate('created_at', today())->sum('total');
        $weekRevenue = Order::where('created_at', '>=', now()->subDays(7))->sum('total');
        $monthRevenue = Order::where('created_at', '>=', now()->subDays(30))->sum('total');
        
        $pendingOrders = Order::where('status', 'pending')->count();
        
        return [
            Stat::make("Today's Revenue", '$' . number_format($todayRevenue, 2))
                ->description('12 orders today')
                ->color('success'),
            
            Stat::make('This Week', '$' . number_format($weekRevenue, 2))
                ->description('+15% from last week')
                ->color('primary'),
            
            Stat::make('This Month', '$' . number_format($monthRevenue, 2))
                ->description('328 total orders')
                ->chart([65, 59, 80, 81, 56, 55, 40])
                ->color('info'),
            
            Stat::make('Pending Orders', $pendingOrders)
                ->description('Require attention')
                ->color('warning')
                ->url('/admin/orders?tableFilters[status][value]=pending'),
        ];
    }
}
```

```php
<?php

// app/Filament/Widgets/RevenueChart.php
namespace App\Filament\Widgets;

use App\Models\Order;
use Filament\Widgets\ChartWidget;

class RevenueChart extends ChartWidget
{
    protected static ?string $heading = 'Revenue Overview';
    protected static ?int $sort = 2;
    protected int | string | array $columnSpan = 2;
    
    protected function getData(): array
    {
        $revenue = Order::query()
            ->selectRaw('DATE(created_at) as date, SUM(total) as revenue')
            ->where('created_at', '>=', now()->subDays(30))
            ->groupBy('date')
            ->orderBy('date')
            ->pluck('revenue', 'date')
            ->toArray();
        
        return [
            'datasets' => [
                [
                    'label' => 'Revenue',
                    'data' => array_values($revenue),
                    'backgroundColor' => 'rgba(34, 197, 94, 0.2)',
                    'borderColor' => '#22c55e',
                    'borderWidth' => 2,
                    'fill' => true,
                    'tension' => 0.4,
                ],
            ],
            'labels' => array_keys($revenue),
        ];
    }
    
    protected function getType(): string
    {
        return 'line';
    }
    
    protected function getOptions(): array
    {
        return [
            'scales' => [
                'y' => [
                    'beginAtZero' => true,
                    'ticks' => [
                        'callback' => 'function(value) { return "$" + value; }',
                    ],
                ],
            ],
        ];
    }
}
```

## Best Practices

1. **Keep widgets focused** - One widget per metric type
2. **Use appropriate chart types** - Line for trends, pie for proportions
3. **Add context** - Include descriptions and comparisons
4. **Use colors effectively** - Green for positive, red for negative
5. **Enable polling** only for real-time data needs
6. **Limit table widget rows** - Use pagination for large datasets
7. **Add clickable links** to stats for easy navigation
8. **Use sparklines** on stats for trend visualization
9. **Cache expensive queries** - Use Laravel's cache for heavy calculations
10. **Test on mobile** - Ensure widgets are responsive
11. **Use columnSpan** effectively - Balance the dashboard layout
12. **Sort widgets logically** - Most important first
13. **Add filters** to charts for different time ranges
14. **Show empty states** - Handle cases with no data gracefully
15. **Optimize queries** - Use aggregates and avoid N+1

## Tips & Tricks

### Query Optimization

```php
protected function getStats(): array
{
    // Cache expensive queries
    $stats = cache()->remember('dashboard_stats', 300, function () {
        return [
            'users' => User::count(),
            'orders' => Order::count(),
            'revenue' => Order::sum('total'),
        ];
    });
    
    return [
        Stat::make('Users', $stats['users']),
        Stat::make('Orders', $stats['orders']),
        Stat::make('Revenue', '$' . number_format($stats['revenue'], 2)),
    ];
}
```

### Conditional Display

```php
public static function canView(): bool
{
    return auth()->user()->can('view dashboard stats');
}
```

### Dynamic Headings

```php
protected function getHeading(): ?string
{
    return 'Sales: ' . now()->format('F Y');
}
```

## Additional Resources

- [Official Widgets Documentation](https://filamentphp.com/docs/5.x/widgets/stats-overview)
- [Chart.js Documentation](https://www.chartjs.org/docs/)
- [Trend Package](https://github.com/flowframe/laravel-trend) for time series data
