Yii2-filter
==========
Внимание: модуль в глубокой разработке, использовать на свой страх и риск.

Модуль позволит добавлять опции для любой вашей модели в админке, а снаружи фильтровать результаты выдачи по выбранным опциям.

Функционал:

* Добавление фильтров (они же опции)
* Присвоение разных фильтров разным моделям (по значению поля)
* Управление вариантами опций
* Набор виджетов

Установка
---------------------------------
Выполнить команду

```
php composer require pistol88/yii2-filter "*"
```

Или добавить в composer.json

```
"pistol88/yii2-filter": "*",
```

И выполнить

```
php composer update
```

Далее, мигрируем базу:

```
php yii migrate --migrationPath=vendor/pistol88/yii2-filter/migrations
```

Подключение и настройка
---------------------------------
В конфигурационный файл приложения добавить модуль filter, настроив его

```php
    'modules' => [
        //...
        'filter' => [
            'class' => 'pistol88\filter\Module',
            'relationModel' => 'common\models\Product', //Модель, которой будут присвоены опции
            'relationFieldName' => 'category_id', //Наименование поля, по значению которого будут привязыватья опции
            //callback функция, которая возвращает варианты relationFieldName
            'relationFieldValues' =>
                function() {
                    //Пример с деревом:
                    $return = [];
                    $categories = \common\models\Category::find()->all();
                    foreach($categories as $category) {
                       if(empty($category->parent_id)) {
                            $return[] = $category;
                            foreach($categories as $category2) {
                                if($category2->parent_id == $category->id) {
                                    $category2->name = ' --- '.$category2->name;
                                    $return[] = $category2;
                                }
                            }
                       }
                    }
                    return \yii\helpers\ArrayHelper::map($return, 'id', 'name');
                },
        ],
        //...
    ]
```

Управление фильтрами: ?r=filter/filter

Настройка
---------------------------------
Для модели, с которой работает фильтр, добавить поведение:

```php
    function behaviors() {
        return [
            'filter' => [
                'class' => 'pistol88\filter\behaviors\AttachFilterValues',
            ],
        ];
    }
```

Чтобы иметь возможность также фильтровать результаты Find, подменяем Query:

```php
    public static function Find()
    {
        $return = new ProductQuery(get_called_class());
        return $return;
    }
```

В ProductQuery должно быть это поведение:

```php
    function behaviors()
    {
       return [
           'filter' => [
               'class' => 'pistol88\filter\behaviors\Filtered',
           ],
       ];
    }
```

Использование
---------------------------------
Получить опции и их значения из модели, в которой есть поведение AttachFilterValues:
```php

<?php if($filters = $model->getSelectedFilters()) {?>
        <?php foreach($filters as $filter_name => $filter_values) { ?>
                <p>
                        <strong><?=$filter_name;?></strong>: <?=implode(', ', $filter_values);?>
                </p>
        <?php } ?> 
<?php } ?>

```

Виджеты
---------------------------------

Блок выбора значений для опций модели $model (опция будет выведена, только если к данной модели через поле relationFieldName привязаны какие-то опции)
<?=\pistol88\filter\widgets\Choice::widget(['model' => $model]);?>

Вывод блока с фильтрами (галочки, радиобаттоны и т.д.).
<?=\pistol88\filter\widgets\FilterPanel::widget(['modelName' => 'common\models\Product', 'itemId' => $model->id]);?>

* itemId - значение relationFieldName
* modelName - модель, которая будет фильтроваться
						