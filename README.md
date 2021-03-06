# Smush.it PHP Library

A simple PHP library for accessing the [Yahoo! Smush.it™](http://www.smushit.com/ysmush.it/) lossless image compressor.

## Requirements

- PHP version 5.3 ([closures](http://php.net/manual/en/functions.anonymous.php) in SmushIt::clean())
- PHP [cURL extension](http://php.net/manual/en/book.curl.php) (Client URL Library)

## Basic usage

Using Yahoo! Smush.it™ to compress a single local file.

	$smushit = new SmushIt('/path/to/image.png');

Using Yahoo! Smush.it™ to compress a single remote file.

    $smushit = new SmushIt('http://example.org/image.jpg');

Using Yahoo! Smush.it™ to compress multiple files at once.

    $smushit = new SmushIt(array(
        '/path/to/image.png',
        'http://example.org/image.jpg'
    ));

Get "Smushed images".

    $smushit->get();
    // Sample result:
    // array (size=1)
    //   0 =>
    //     object(SmushIt)[4]
    //       public 'error' => null
    //       public 'source' => string 'http://example.org/image.jpg' (length=35)
    //       public 'destination' => string 'http://ysmushit.zenfs.com/results/image.jpg' (length=74)
    //       public 'sourceSize' => int 444808
    //       public 'destinationSize' => int 227097
    //       public 'savings' => float 48.94

## Advanced usage

Smush.it PHP Library removes SmushIt objects from get() method result when an error occured during compression process (including "no savings", "image size exceeds 1MB", etc). You can keep these results in get() array by passing `SmushIt::KEEP_ERRORS` flag to SmushIt constructor.

```php
<?php

$images = array(
    'http://example.org/image.jpg',
    'http://example.org/broken-link.jpg'
);

$default = new SmushIt($images);
$keepErr = new SmushIt($images, SmushIt::KEEP_ERRORS);

count($default->get()); // Value: 1
count($keepErr->get()); // Value: 2

?>
```

PHP SmushIt Library can throw exception when an error occured.

```php
<?php

$smushit = new SmushIt(array(
    'http://example.org/image.jpg',
    // InvalidArgumentException when calling SmushIt::__construct() with empty $path argument.
    '',
    // This image will never be subjected to the API
    'http://example.org/another-image.jpg'
), SmushIt::THROW_EXCEPTION);

?>
```

## Code Sample

This code sample demonstrates how to compress all `.jpg` files from a directory named `images/` and replace original files by their compressed version.

```php
<?php

// Processing may take a while
set_time_limit(0);

// Include Smush.it PHP Library
require 'SmushIt.class.php';

// Get all .jpg files from images/ directory
$files = glob(__DIR__ . '/images/*.jpg');

// Make batches of 3 images
$files = array_chunk($files, 3);

// Take a batch of three files
foreach($files as $batch) {
    try {
        // Compress the batch
        $smushit = new SmushIt($batch);
        // And finaly, replace original files by their compressed version
        foreach($smushit->get() as $file) {
            // Sometimes, Smush.it convert files. We don't want that to happen.
            $src = pathinfo($file->source, PATHINFO_EXTENSION);
            $dst = pathinfo($file->destination, PATHINFO_EXTENSION);
            if ($src == $dst AND copy($file->destination, $file->source)) {
                // Success !
            }
        }
    } catch(Exception $e) {
        continue;
    }
}

?>
```

## License

Copyright (C) 2012 Ghislain PHU <contact@ghislainphu.fr>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
