# Form Components Reference

Complete reference for Filament v5 form components and schemas.

## Available Form Fields

Filament v5 provides 20+ form field types in the `Filament\Forms\Components` namespace:

| Field Type | Class | Description |
|------------|-------|-------------|
| Text Input | `TextInput` | Single-line text input with validation |
| Textarea | `Textarea` | Multi-line text input |
| Select | `Select` | Dropdown selection |
| Checkbox | `Checkbox` | Boolean checkbox |
| Toggle | `Toggle` | Switch toggle |
| Checkbox List | `CheckboxList` | Multiple checkboxes |
| Radio | `Radio` | Radio button group |
| Date Picker | `DatePicker` | Date selection |
| DateTime Picker | `DateTimePicker` | Date and time selection |
| File Upload | `FileUpload` | File upload with preview |
| Rich Editor | `RichEditor` | WYSIWYG editor (TipTap) |
| Markdown Editor | `MarkdownEditor` | Markdown editing |
| Repeater | `Repeater` | Repeatable field groups |
| Builder | `Builder` | Block-based content builder |
| Tags Input | `TagsInput` | Tag creation |
| Key-value | `KeyValue` | Key-value pairs |
| Color Picker | `ColorPicker` | Color selection |
| Toggle Buttons | `ToggleButtons` | Button group selection |
| Slider | `Slider` | Range slider |
| Hidden | `Hidden` | Hidden input field |

## Text Input

```php
use Filament\Forms\Components\TextInput;

TextInput::make('name')
    ->required()
    ->maxLength(255)
    ->minLength(2)
    ->autocomplete()
    ->autofocus()
    ->placeholder('Enter name')
    ->prefix('Mr/Ms')
    ->suffix('@company.com')
    ->helperText('Your full name')
    ->hint('Required')
    ->hintIcon('heroicon-m-question-mark-circle')
    ->disabled()
    ->readonly()
    ->hidden()
    ->dehydrated(false)  // Don't save to database
    ->live()  // Real-time updates
    ->afterStateUpdated(fn ($state, $set) => $set('slug', Str::slug($state)))
```

### Validation Methods

```php
TextInput::make('email')
    ->email()
    ->required()
    ->unique('users', 'email', ignoreRecord: true)
    ->maxLength(255)
    
TextInput::make('password')
    ->password()
    ->required()
    ->minLength(8)
    ->regex('/^(?=.*[A-Z])(?=.*\d).+$/')
    ->confirmed()  // Requires password_confirmation field
    
TextInput::make('slug')
    ->required()
    ->alphaDash()
    ->unique('posts', 'slug', ignoreRecord: true)
```

## Select

```php
use Filament\Forms\Components\Select;

Select::make('status')
    ->options([
        'draft' => 'Draft',
        'published' => 'Published',
        'archived' => 'Archived',
    ])
    ->required()
    ->searchable()
    ->preload()
    ->multiple()
    ->native(false)
    ->placeholder('Select a status')
    ->noSearchResultsMessage('No status found')
    ->loadingMessage('Loading statuses...')
    ->searchPrompt('Search for a status')
```

### Relationship Selection

```php
Select::make('author_id')
    ->relationship('author', 'name')
    ->searchable()
    ->preload()
    ->createOptionForm([
        TextInput::make('name')->required(),
        TextInput::make('email')->email()->required(),
    ])
    ->editOptionForm([
        TextInput::make('name')->required(),
    ])
    ->createOptionAction(fn ($data, $set) => $set('author_id', $data['id']))
```

### Dynamic Options

```php
Select::make('city')
    ->options(fn (Get $get): array => match ($get('country')) {
        'usa' => ['nyc' => 'New York', 'la' => 'Los Angeles'],
        'uk' => ['london' => 'London', 'manchester' => 'Manchester'],
        default => [],
    })
    ->live()
    ->afterStateUpdated(fn (Set $set) => $set('zip', null))
```

## Checkbox & Toggle

```php
use Filament\Forms\Components\Checkbox;
use Filament\Forms\Components\Toggle;

Checkbox::make('is_active')
    ->label('Active')
    ->helperText('Check to activate')
    ->inline()
    
Toggle::make('is_featured')
    ->label('Featured')
    ->onColor('success')
    ->offColor('danger')
    ->inline(false)
```

## Date Pickers

```php
use Filament\Forms\Components\DatePicker;
use Filament\Forms\Components\DateTimePicker;
use Filament\Forms\Components\TimePicker;

DatePicker::make('birthdate')
    ->required()
    ->minDate(now()->subYears(100))
    ->maxDate(now()->subYears(18))
    ->native(false)
    ->displayFormat('M d, Y')
    
DateTimePicker::make('published_at')
    ->required()
    ->seconds(false)
    ->timezone('America/New_York')
    
TimePicker::make('opening_time')
    ->required()
    ->withoutSeconds()
```

## File Upload

```php
use Filament\Forms\Components\FileUpload;
use Filament\Forms\Components\ImageUpload;

FileUpload::make('attachment')
    ->required()
    ->multiple()
    ->directory('attachments')
    ->disk('s3')
    ->preserveFilenames()
    ->maxSize(10240)  // 10MB
    ->minSize(1024)   // 1MB
    ->acceptedFileTypes(['application/pdf', 'image/*'])
    ->maxFiles(5)
    ->openable()
    ->downloadable()
    ->previewable()
    ->imagePreviewHeight('250')
    
ImageUpload::make('avatar')
    ->image()
    ->imageEditor()
    ->imageEditorAspectRatios([
        null,
        '16:9',
        '4:3',
        '1:1',
    ])
    ->circleCropper()
    ->squareCropper()
```

## Rich Editor

```php
use Filament\Forms\Components\RichEditor;

RichEditor::make('content')
    ->required()
    ->columnSpanFull()
    ->toolbarButtons([
        'attachFiles',
        'blockquote',
        'bold',
        'bulletList',
        'codeBlock',
        'h2',
        'h3',
        'italic',
        'link',
        'orderedList',
        'redo',
        'strike',
        'underline',
        'undo',
    ])
    ->fileAttachmentsDirectory('posts/images')
    ->fileAttachmentsDisk('s3')
```

## Repeater

```php
use Filament\Forms\Components\Repeater;

Repeater::make('items')
    ->schema([
        TextInput::make('name')->required(),
        TextInput::make('quantity')->numeric()->required(),
        TextInput::make('price')->numeric()->prefix('$')->required(),
    ])
    ->columns(3)
    ->defaultItems(1)
    ->addActionLabel('Add Item')
    ->reorderable(true)
    ->collapsible()
    ->collapsed()
    ->cloneable()
    ->grid(2)
    ->maxItems(10)
    ->minItems(1)
    ->deleteAction(
        fn (Action $action) => $action->requiresConfirmation(),
    )
```

## Builder (Block Editor)

```php
use Filament\Forms\Components\Builder;

Builder::make('content')
    ->blocks([
        Builder\Block::make('heading')
            ->schema([
                TextInput::make('content')
                    ->label('Heading')
                    ->required(),
                Select::make('level')
                    ->options([
                        'h1' => 'Heading 1',
                        'h2' => 'Heading 2',
                        'h3' => 'Heading 3',
                    ])
                    ->required(),
            ])
            ->label('Heading'),
            
        Builder\Block::make('paragraph')
            ->schema([
                RichEditor::make('content')
                    ->label('Paragraph')
                    ->required(),
            ])
            ->label('Paragraph'),
            
        Builder\Block::make('image')
            ->schema([
                FileUpload::make('image')
                    ->image()
                    ->required(),
                TextInput::make('caption'),
            ])
            ->label('Image'),
    ])
    ->collapsible()
    ->defaultItems(0)
```

## Layout Components

### Grid

```php
use Filament\Forms\Components\Grid;

Grid::make(2)
    ->schema([
        TextInput::make('first_name'),
        TextInput::make('last_name'),
    ])

// Responsive grid
Grid::make([
    'default' => 1,
    'sm' => 2,
    'lg' => 3,
    'xl' => 4,
])->schema([
    // ...
])
```

### Section

```php
use Filament\Forms\Components\Section;

Section::make('Personal Information')
    ->description('Enter your personal details')
    ->icon('heroicon-m-user')
    ->collapsible()
    ->collapsed()
    ->compact()
    ->aside()  // Side-by-side layout
    ->schema([
        TextInput::make('name'),
        TextInput::make('email'),
    ])
    ->columns(2)
```

### Tabs

```php
use Filament\Forms\Components\Tabs;

Tabs::make('Settings')
    ->tabs([
        Tabs\Tab::make('General')
            ->icon('heroicon-m-cog')
            ->schema([
                TextInput::make('site_name'),
                TextInput::make('site_email'),
            ]),
            
        Tabs\Tab::make('SEO')
            ->icon('heroicon-m-globe')
            ->schema([
                TextInput::make('meta_title'),
                Textarea::make('meta_description'),
            ]),
            
        Tabs\Tab::make('Social')
            ->icon('heroicon-m-share')
            ->schema([
                TextInput::make('facebook_url'),
                TextInput::make('twitter_url'),
            ]),
    ])
```

### Wizard

```php
use Filament\Forms\Components\Wizard;

Wizard::make([
    Wizard\Step::make('Account')
        ->icon('heroicon-m-user')
        ->description('Create your account')
        ->schema([
            TextInput::make('email')
                ->email()
                ->required(),
            TextInput::make('password')
                ->password()
                ->required()
                ->confirmed(),
        ]),
        
    Wizard\Step::make('Profile')
        ->icon('heroicon-m-identification')
        ->description('Set up your profile')
        ->schema([
            TextInput::make('name')->required(),
            FileUpload::make('avatar')->image(),
        ]),
        
    Wizard\Step::make('Preferences')
        ->icon('heroicon-m-cog')
        ->description('Customize your experience')
        ->schema([
            Toggle::make('newsletter'),
            Select::make('timezone'),
        ]),
])
->skippable()
->persistInQueryString()
->submitAction(
    Action::make('createAccount')
        ->label('Create Account')
)
```

## Fieldset

```php
use Filament\Forms\Components\Fieldset;

Fieldset::make('Address')
    ->schema([
        TextInput::make('street'),
        TextInput::make('city'),
        TextInput::make('zip'),
    ])
```

## Groups

```php
use Filament\Forms\Components\Group;

Group::make()
    ->schema([
        // Fields that should be grouped together
    ])
    ->columnSpanFull()
    ->columns(2)
```

## Placeholder

```php
use Filament\Forms\Components\Placeholder;

Placeholder::make('summary')
    ->content(fn ($record): string => "Created: {$record->created_at}")
    ->columnSpanFull()
```

## KeyValue

```php
use Filament\Forms\Components\KeyValue;

KeyValue::make('meta')
    ->keyLabel('Property')
    ->valueLabel('Value')
    ->addActionLabel('Add Property')
    ->keyPlaceholder('Property name')
    ->valuePlaceholder('Property value')
    ->reorderable()
```

## Tags Input

```php
use Filament\Forms\Components\TagsInput;

TagsInput::make('skills')
    ->label('Skills')
    ->placeholder('Add a skill')
    ->suggestions(['PHP', 'Laravel', 'JavaScript', 'Vue.js', 'React'])
    ->splitKeys(['Tab', ',', 'Enter'])
    ->reorderable()
```

## Color Picker

```php
use Filament\Forms\Components\ColorPicker;

ColorPicker::make('color')
    ->label('Brand Color')
    ->hex()
    ->rgb()
    ->rgba()
    ->hsl()
    ->hsv()
    ->default('#f59e0b')
```

## Toggle Buttons

```php
use Filament\Forms\Components\ToggleButtons;

ToggleButtons::make('size')
    ->options([
        'sm' => 'Small',
        'md' => 'Medium',
        'lg' => 'Large',
    ])
    ->icons([
        'sm' => 'heroicon-m-sun',
        'md' => 'heroicon-m-moon',
        'lg' => 'heroicon-m-star',
    ])
    ->default('md')
    ->inline()
    ->grouped()
```

## Slider

```php
use Filament\Forms\Components\Slider;

Slider::make('volume')
    ->min(0)
    ->max(100)
    ->step(10)
    ->default(50)
    ->suffix('%')
```

## Validation

### Built-in Rules

```php
TextInput::make('email')
    ->required()
    ->email()
    ->unique('users', 'email', ignoreRecord: true)
    ->maxLength(255)
    ->minLength(5)

TextInput::make('password')
    ->required()
    ->minLength(8)
    ->regex('/^(?=.*[A-Z])(?=.*\d).+$/')
    ->confirmed()  // Requires password_confirmation

TextInput::make('age')
    ->numeric()
    ->minValue(18)
    ->maxValue(100)
    ->integer()

TextInput::make('website')
    ->url()
    ->activeUrl()

TextInput::make('ip')
    ->ip()

TextInput::make('mac')
    ->macAddress()
```

### Custom Rules

```php
TextInput::make('username')
    ->rules(['required', 'string', 'min:3', 'max:20', 'regex:/^[a-zA-Z0-9_]+$/'])
    
TextInput::make('code')
    ->rule(function ($state) {
        return $state === 'valid-code' ? null : 'Invalid code';
    })
```

### Conditional Rules

```php
TextInput::make('company_name')
    ->required(fn (Get $get): bool => $get('is_company'))
    
TextInput::make('phone')
    ->requiredWithout('email')
```

## Conditional Visibility

```php
TextInput::make('company_name')
    ->visible(fn (Get $get): bool => $get('is_company'))
    ->hidden(fn (Get $get): bool => ! $get('is_company'))
    ->disabled(fn (Get $get): bool => $get('is_locked'))
    ->readonly(fn (): bool => auth()->user()->cannot('edit'))
```

## State Management

```php
use Filament\Forms\Components\Utilities\Get;
use Filament\Forms\Components\Utilities\Set;

Select::make('country')
    ->live()
    ->afterStateUpdated(fn (Set $set) => $set('city', null))

TextInput::make('name')
    ->live(onBlur: true)
    ->afterStateUpdated(fn ($state, Set $set) => $set('slug', Str::slug($state)))

// Access other field values
TextInput::make('total')
    ->formatStateUsing(fn ($state, Get $get): string => 
        $get('quantity') * $get('price')
    )
    ->dehydrated(false)
```

## Complete Example: Product Form

```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Section::make('Basic Information')
                ->schema([
                    TextInput::make('name')
                        ->required()
                        ->maxLength(255)
                        ->live(onBlur: true)
                        ->afterStateUpdated(fn ($state, $set) => 
                            $set('slug', Str::slug($state))
                        ),
                    
                    TextInput::make('slug')
                        ->required()
                        ->unique(ignoreRecord: true)
                        ->disabled()
                        ->dehydrated(),
                    
                    Select::make('category_id')
                        ->relationship('category', 'name')
                        ->searchable()
                        ->preload()
                        ->required(),
                    
                    Textarea::make('description')
                        ->required()
                        ->maxLength(1000)
                        ->columnSpanFull(),
                ])
                ->columns(2),
            
            Section::make('Pricing & Inventory')
                ->schema([
                    TextInput::make('price')
                        ->required()
                        ->numeric()
                        ->prefix('$')
                        ->maxValue(999999.99),
                    
                    TextInput::make('sale_price')
                        ->numeric()
                        ->prefix('$')
                        ->lte('price', 'Must be less than or equal to regular price'),
                    
                    TextInput::make('stock_quantity')
                        ->required()
                        ->numeric()
                        ->integer()
                        ->minValue(0),
                    
                    Toggle::make('track_inventory')
                        ->default(true),
                ])
                ->columns(2),
            
            Section::make('Media')
                ->schema([
                    FileUpload::make('images')
                        ->multiple()
                        ->image()
                        ->maxFiles(10)
                        ->directory('products')
                        ->columnSpanFull(),
                ]),
            
            Section::make('SEO')
                ->schema([
                    TextInput::make('meta_title')
                        ->maxLength(70),
                    
                    Textarea::make('meta_description')
                        ->maxLength(160)
                        ->columnSpanFull(),
                ])
                ->collapsible()
                ->collapsed(),
        ]);
}
```

## Tips & Best Practices

1. **Use sections to group related fields**
2. **Leverage live() for real-time updates**
3. **Always validate input with built-in methods**
4. **Use afterStateUpdated to compute derived fields**
5. **Disable fields that shouldn't be edited instead of hiding them**
6. **Use placeholders for computed/display-only values**
7. **Enable reorderable() on repeaters for better UX**
8. **Use builder blocks for flexible content structures**
9. **Add helper text and hints for complex fields**
10. **Test forms with various input combinations**

## Additional Resources

- [Official Forms Documentation](https://filamentphp.com/docs/5.x/schemas/forms)
- [Form Layout](https://filamentphp.com/docs/5.x/schemas/layout)
- [Validation](https://filamentphp.com/docs/5.x/schemas/validation)
