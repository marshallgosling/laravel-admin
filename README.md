# laravel-admin
Anything good for laravel-admin

## Grid Filter using % as input, no hard code
Add below php code to Admin\bootstrap.php
```php
Encore\Admin\Grid\Filter::extend('mlike', \App\Admin\Extensions\MyLike::class);

```

MyLike.php file
```php
<?php
namespace App\Admin\Extensions;

use Encore\Admin\Grid\Filter\Like;

class MyLike extends Like
{
    protected $exprFormat = '{value}';
}

```

Usage
```php

$grid->filter(function (Grid\Filter $filter) {
    $filter->mlike('name', __('Name'))->placeholder('User % for wildcard，e.g great% 或 %good%');
});

```

## Grid components support mouse multiple selection
Add below php code to Admin\bootstrap.php

```php

Encore\Admin\Grid::init(function (Encore\Admin\Grid $grid) {
    $js = <<<JS
    var startmove = false;
    var templist = [];
    var shift = false;
    $('body').on('keydown', function(e) {
        if(e.shiftKey) shift = true;
    });
    $('body').on('keyup', function(e) {
        shift = false;
    });
    $('tbody > tr').on('mousedown', function(e) {
        if(!shift) return;
        e.preventDefault();
        let idx = $(this).data('key');
        
        startmove = true;
        $('input[data-id='+idx+']').iCheck('toggle');
    });
    $('tbody > tr').on('mouseenter', function(e) {
        if(startmove) {
            let idx = $(this).data('key');
            
            if(templist.indexOf(idx) == -1) {
                templist.push(idx);
                $('input[data-id='+idx+']').iCheck('toggle');
            }
            else {
                let t = templist.splice(templist.indexOf(idx));

                for(i=0;i<t.length;i++) {
                    $('input[data-id='+t[i]+']').iCheck('toggle');
                }
            }
        }
    });
    $('body').on('mouseup', function(e) {
        startmove = false;
        templist = [];
        
        var selected = $.admin.grid.selected().length;

        if (selected > 0) {
            $('.grid-select-all-btn').show();
        } else {
            $('.grid-select-all-btn').hide();
        }

        $('.grid-select-all-btn .selected').html("已选择 {n} 项".replace('{n}', selected));
    });
JS;
    \Encore\Admin\Facades\Admin::script($js);
}

```
