# Laravel 5 Plupload

######Laravel package for Plupload http://plupload.com.
This package uses some parts of https://github.com/jildertmiedema/laravel-plupload

## Installation
Require this package with composer:

```
composer require AntonKozytskyi/laravel-plupload
```

or add this to `composer.json`

```
"AntonKozytskyi/laravel-plupload": "^1.0"
```

After updating composer, add the ServiceProvider to the providers array in `config/app.php`
```php
'AntonKozytskyi\LaravelPlupload\PluploadServiceProvider',
// or 
AntonKozytskyi\LaravelPlupload\PluploadServiceProvider::class, // PHP 5.5
```

Add this to your facades in `config/app.php`:

```php
'Plupload' => 'AntonKozytskyi\LaravelPlupload\Facades\Plupload',
// or 
'Plupload' => AntonKozytskyi\LaravelPlupload\Facades\Plupload::class, // PHP 5.5
```

Copy the package config to your local config with the publish command:

```
php artisan vendor:publish
```
or
```
php artisan vendor:publish --provider="AntonKozytskyi\LaravelPlupload\PluploadServiceProvider"
```


## Usage


### Uploading files
##### 1. Use default plupload html

Use the [examples](http://www.plupload.com/examples) found on the plupload site. The [Getting Started](http://plupload.com/docs/Getting-Started) page is good place to start.


##### 2. Plupload builder

**make($id, $url)**

Create new uploader.
* **$id**: the unique identification for the uploader.
* **$url**: the upload url end point.
```php
{!! Plupload::make('my_uploader_id', action('MediaController@postImageUpload'))->render() !!}
```

**render($view = 'plupload::uploader')**

Renders the uploader. You can customize this by passing a view name.

##### 3. Use package js file to initialize Plupload (Optional)

If you do not want to write your own js to initialize Plupload, you can use the `upload.js` file that included with the package in `resources/views/vendor/plupload/assets/js`. Make sure that you already have `jQuery` loaded on your page.

**Initialize Plupload**

```js
<script>
$(function () {
    createUploader('my_uploader_id'); // The Id that you used to create with the builder
});
</script>
```


These following methods are useable with the `upload.js` file.

**Set Uploader options**

**setOptions(array $options)**

Set uploader options. Please visit https://github.com/moxiecode/plupload/wiki/Options to see all the options. You can set the default global options in `config/plupload.php`

```php
{!! Plupload::make('my_uploader_id', action('MediaController@postImageUpload'))
    ->setOptions([
        'filters' => [
            'max_file_size' => '2mb',
            'mime_types' => [
                ['title' => "Image files", 'extensions' => "jpg,gif,png"],
            ],
        ],
    ])
    ->render() !!}
```

**Automatically start upload when files added**

Use `setAutoStart()` in your builder before calling render() function.

**setAutoStart($bool)**

* **$bool**: `true` or `false`

```php
{!! Plupload::make('my_uploader_id', action('MediaController@postImageUpload'))
  ->setAutoStart(true)->render() !!}
```


### Receiving files


**file($name, $handler)**
* **$name**: the input name.
* **$handler**: callback handler.

Use this in your route or your controller. Feel free to modify to suit your needs.

```php
return \Plupload::file('file', function($file) {
    // Store the uploaded file
    $file->move(storage_path('upload/images'), $file->getClientOriginalName());

    // Save the record to the db
    $photo = \App\Photo::create([
        'name' => $file->getClientOriginalName(),
        'type' => 'image',
        //...
    ]);

    // This will be included in JSON response result
    return [
        'success'   => true,
        'message'   => 'Upload successful.',
        'id'        => $photo->id,
        // 'url'       => $photo->getImageUrl($filename, 'medium'),
        // 'deleteUrl' => action('MediaController@deleteDelete', [$photo->id])
    ];
});
```

If you are using the package `upload.js` file. The `url` and `deleteUrl` in the JSON payload will be used to generate preview and delete link while the `id` will be appended to the uploader as a hidden field with the following format:

`<input type="hidden" name="{uploaderId}_files[]" value="{id}" />`. 

Please note that the `deleteUrl` uses `DELETE` method.
