### What's up

This is a sandbox in which a more flexible system for displaying the columns is tested.
So far it was the case that a column was integrated into an inheritance hierarchy.
With adding a new feature, the `Diamond of Death` can occur. The code is shifting more and more
towards the base class and is too inflexible. At the moment I'm working on a system of widgets
without logic that can be added to a column. A renderer later takes the column and the widgets
and performs the rendering.

This allows the user to write his own widgets and renderers and thus determine the structure of the column.

### Setup

```bash
$ symfony new blog
$ cd blog
$ mkdir lib
$ cd lib
$ git clone https://github.com/stwe/DatatablesBundle.git -b 2.0 --single-branch
```

```bash
$ composer require annotations
$ composer require template
$ composer require profiler --dev
$ composer require debug
$ composer require symfony/asset
```

##### bundles.php

```php
// blog/config/bundles.php

return [
    // ...

    Sg\DatatablesBundle\SgDatatablesBundle::class => ['all' => true],
];
```

##### composer.json

```html
    "autoload": {
        "psr-4": {
            "Sg\\DatatablesBundle\\": "lib/DatatablesBundle/src/",
            "App\\": "src/"
        }
    },
```

```bash
$ composer dump-autoload
```

##### services.yaml

```yaml
App\Datatables\:
    resource: '../src/Datatables/'
    tags: ['datatable']
```

##### PostDatatable.php

```php
// src/Datatables/PostDatatable.php

namespace App\Datatables;

use Sg\DatatablesBundle\Datatable\AbstractDatatable;
use Sg\DatatablesBundle\Datatable\Renderer\DummyRenderer;
use Sg\DatatablesBundle\Datatable\Widget\BooleanWidget;
use Sg\DatatablesBundle\Datatable\Widget\HtmlFormatWidget;

class PostDatatable extends AbstractDatatable
{
    public function buildDatatable(): void
    {
        // create and add columns
        $this->getColumnBuilder()->addColumn('name');
        $this->getColumnBuilder()->addColumn('position');

        // add widgets
        $cols = $this->getColumnBuilder()->getColumns();

        // name column
        $currentCol0 = $cols[0];
        $currentCol0->addWidget(new BooleanWidget());
        $currentCol0->addWidget(new HtmlFormatWidget());

        // position column
        $currentCol1 = $cols[1];
        $bw = new BooleanWidget();
        $bw->setTrueLabel('custom True label');
        $currentCol1->addWidget($bw);
        $hw = new HtmlFormatWidget();
        $hw->setBold(true);
        $hw->setItalic(true);
        $currentCol1->addWidget($hw);

        // add a renderer which is compatible with added widgets
        $currentCol0->setRenderer(new DummyRenderer());
        $currentCol1->setRenderer(new DummyRenderer());
    }
    
    public function getId(): string
    {
        return 'post';
    }
}
```

##### PostController

```php
// Controller/PostController

namespace App\Controller;

use Sg\DatatablesBundle\Datatables\Datatables;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class PostController extends AbstractController
{
    /**
     * @Route("/")
     */
    public function homepage(Datatables $datatables): Response
    {
        $datatable = $datatables->getDatatableById('post');

        return $this->render(
            'post/index.html.twig',
            [
                'datatable' => $datatable
            ]
        );
    }

    /**
     * @Route("/ajax", name="table_content", methods={"GET", "POST"})
     */
    public function tabledata(Request $request, Datatables $datatables)
    {
        //$response = $datatables->handleRequest($request, 'post');

        $data =
            [
                'data' =>
                    [
                        [
                            'name' => 'Tiger',
                            'position' => false
                        ],
                        [
                            'name' => '',
                            'position' => 'Worker'
                        ]
                    ]
            ];

        $jsonResponse = $datatables->handleResponse($data, 'post');

        return $jsonResponse;
    }
}
```

##### base.html.twig

```html
<!-- base.html.twig -->

<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>
            {% block title %}
            {% endblock %}
        </title>

        {% block stylesheets %}
            <link rel="stylesheet" type="text/css" href="https://cdn.datatables.net/1.10.24/css/jquery.dataTables.min.css" />
        {% endblock %}

        {% block javascripts %}
            <script type="text/javascript" src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
            <script type="text/javascript" src="https://cdn.datatables.net/1.10.24/js/jquery.dataTables.min.js"></script>
        {% endblock %}
    </head>
    <body>
        {% block body %}{% endblock %}
    </body>
</html>
```

##### index.html.twig

```html
<!-- post/index.html.twig -->

{% extends 'base.html.twig' %}

{% block body %}
    <h2>Posts list</h2>
    {{ sg_datatables_render(datatable) }}
{% endblock %}
```
