---
title: 定制化GridView
date: 2017-03-22 10:47:02
tags:
---

```php
GridView::widget([
  'dataProvider' => $dataProvider,
  'columns' => [
    ['class' => 'yii\grid\SerialColumn'],

    [
      'label' => '标题1',
      'value' => function ($model) {
        return $model->name;
      }
    ],

    [
      'class' => 'yii\grid\ActionColumn',
      'header' => '标题2',
      'template' => '{add-child} {update} {delete}',
      'buttons'=>[
        'add-child' => function($url, $model, $key) {
          return Html::a('添加商品', '#', [
            'class' => 'layerAddChild',
            'data-content-id' => $key
          ]);
        },
        'update' => function($url, $model, $key) {
          return Html::a('编辑商品', '#', [
            'class' => 'layerUpdateBrand',
            'data-content-id' => $key
          ]);
        },
        'delete' => function($url, $model, $key) {
          return Html::a('删除商品', '#', [
            'class' => 'layerDeleteBrand',
            'data-content-id' => $key
          ]);
        },
      ],
    ],
  ],
]);
```

