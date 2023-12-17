# laravel-admin
Anything good for laravel-admin

## Disable tools' buttons in Show class via bootstrap init.
bootstrap.php file
```php

\Encore\Admin\Show::init(function (\Encore\Admin\Show $show) {
    $show->panel()->tools(function (\Encore\Admin\Show\Tools $tools) {
        $tools->disableDelete();
        $tools->disableEdit();
        $tools->disableList();
    });
});
```

## Customer Form for redirect with querystring after submitted.
MyForm.php file
```php
<?php

namespace App\Admin\Extensions;

use Encore\Admin\Form;

class MyForm extends Form
{
    public $queryString = '';

    public function resource($slice = -2): string
    {
        $segments = explode('/', trim(\request()->getUri(), '/'));

        if ($slice !== 0) {
            $segments = array_slice($segments, 0, $slice);
        }

        return implode('/', $segments).$this->queryString;
    }
}
```

Usage for Form function in AdminController file
```php
class MyAdminController {
    /**
     * Make a form builder.
     *
     * @return Form
     */
    protected function form()
    {
        $form = new MyForm(new TemplateRecords());

        // It will redirect to the previous page with the querystring 'parent_id=xxx'
        if(key_exists('parent_id', $_REQUEST)) $form->queryString = '?parent_id='.$_REQUEST['parent_id'];

        $form->select('parent_id', __('Parent'))->default(key_exists('parent_id', $_REQUEST) ? $_REQUEST['parent_id']:'')
                ->options([]);

        ...

        return $form;
    }
}
```

## Grid Filter using % as wildcard, no hard code
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
    $filter->mlike('name', __('Name'))->placeholder('Use % as wildcard，e.g. great% or %good%');
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

        $('.grid-select-all-btn .selected').html("Selected {n} items.".replace('{n}', selected));
    });
JS;
    \Encore\Admin\Facades\Admin::script($js);
}

```
