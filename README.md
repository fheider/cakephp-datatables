# cakephp-datatables

This plugin implements the jQuery dataTables plugin (www.datatables.net) in your CakePHP 3 application.
In addition there was added a multiple column search with request delay to minimize the ajax requests.


## Requirements

* CakePHP 3 (http://www.cakephp.org)
* jQuery (http://www.jquery.com)
* jQuery DataTables (http://www.datatables.net)
* Composer (http://getcomposer.org)


## Optional

* Twitter Bootstrap 3 (http://getbootstrap.com)
* FontAwesome 4 (http://fortawesome.github.io/Font-Awesome)

The core templates are written in Twitter Bootstrap syntax and included FontAwesome icons but can be changed easily.


## Usage

### Step 1: Installation

Use composer to install this plugin.
Add the following repository and requirement to your composer.json:

    "require": {
        "fheider/cakephp-datatables": "dev-master"
    }


### Step 2: Include CakePHP Plugin and load Component and Helper

Load plugin in ***app/bootstrap.php***:
    
    Plugin::load('DataTables', ['bootstrap' => false, 'routes' => false]);




Include component and helper: 

    class AppController extends Controller
    {
        
        public $helpers = [
            'DataTables' => [
                'className' => 'DataTables.DataTables'
            ]
        ];
        
        public function initialize()
        {
            $this->loadComponent('DataTables.DataTables');
        }
        
    }

### Step 3: Include assets

Include jQuery and jQuery DataTables scripts first and then the dataTables logic:

    echo $this->Html->script('*PATH*/jquery.min.js');
    echo $this->Html->script('*PATH*/jquery.dataTables.min.js');
    echo $this->Html->script('*PATH*/dataTables.bootstrap.min.js'); (Optional)
    echo $this->Html->script('DataTables.cakephp.dataTables.js');

Include dataTables css:

    echo $this->Html->css('PATH/dataTables.bootstrap.css');


### Step 4: Add business logic in your controller

Use it simply like find:

    $data = $this->DataTables->find('*TABLE*', [
        'contain' => []
    ]);
    
    $this->set([
        'data' => $data,
        '_serialize' => array_merge($this->viewVars['_serialize'], ['data'])
    ]);
    
The array_merge is required because the component add multiple vars to view like recordsTotal, recordsFiltered, ...
So your serialized data were added to this vars.


### Step 5: Template / View

First display your table normal, so no additional request were sended by dataTables.
The table foot is used for the multiple search fields. This could be input- or select-elements.

    <table class="table table-striped table-bordered table-hover dataTable">
        <thead>
            <tr>
                ...
            </tr>
        </thead>
        <tfoot>
        <tr class="table-search info">
            <td><input type="text" placeholder="Search ..." class="form-control input-sm input-block-level" /></td>
            <td><select><option value="">---</option>...</select></td>
            ...
        </tr>
        </tfoot>
        <tbody>
        <?php foreach($data as $item): ?>
        <tr>
            <td><?= $item->id ?></td>
            <td><?= $item->name ?></td>
            ...
        </tr>
        <?php endforeach; ?>
        </tbody>
    </table>
    
    
Then add the dataTables logic.
The options are exaxt the options you get in the dataTables reference (https://datatables.net/reference/option/).

    $this->DataTables->init([
        'ajax' => [
            'url' => Router::url(['action' => 'index']),
        ],
        'deferLoading' => $recordsTotal,
        'delay' => 600,
        'columns' => [
            [
                'name' => '*MODEL*.id',
                'data' => 'id'
                'orderable' => false
            ],
            [
                'name' => '*MODEL*.name',
                'data' => 'name'
            ],
            ...
        ]
    ])->draw('.dataTable');


In draw method you set the selector of your table. Delay is an additional option for setting the delay for processing
your search input. If delay is 0 on every key press a request will be sent.

**Notes to columns settings**

Every column contains 2 important informations: 

    name = name of your table and field like 'Customers.id'
    data = name of the field in json response

The option **name** is needed for sorting and filtering. The option **data** is needed for processing the json response.  
You also can easily add related data (e.g. a customer belongs to a customer group)

    name = Group.name
    data = group.name
    
**Please keep in mind!**  
It is important that the amount of your columns array is the same like your columns in your HTML-Table! 
