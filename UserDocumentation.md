# Introduction #

## Installation/Requirements ##
[Download the latest MC\_Google\_Visualization](http://code.google.com/p/mc-goog-visualization/downloads/list) and extract the contents of the _lib_ folder to your PHP _include\_path_.  You should now find a top-level _MC_ directory which contains the parser and visualization compontents, and you're done!

### System Requirements ###
  * PHP 5
  * JSON extension or Zend\_Json from the Zend Framework
  * PDO (for automatic request handling and pivots)
  * magic\_quotes\_gpc must be turned off

## Handling Visualization Query Requests ##
Simple usage follows this pattern:
```
<?php
require_once 'MC/Google/Visualization.php';
$db = new PDO('sqlite:example.db');
$vis = new MC_Google_Visualization($db, 'sqlite');

$vis->addEntity('some_table', array(
    'fields' => array(
        'col1' => array('field' => 'col1', 'type' => 'text'),
        'col2' => array('field' => 'col2', 'type' => 'number')
    )
));

$vis->handleRequest();
?>
```
If you take the above code and save it to **/vis.php** on your web server, you can start making visualization requests against your database like so:
```
<html>
  <head>
    <script type="text/javascript" src="http://www.google.com/jsapi"></script>
    <script type="text/javascript">
      google.load("visualization", "1", {packages:["piechart"]});
      google.setOnLoadCallback(drawChart);
      function drawChart() {
        //Tell Google Visualization where your script is
        var query = new google.visualization.Query('/vis.php');
        query.setQuery('select col1, col2 from some_table order by col2 desc');
        query.send(function(result) {
          if(result.isError()) {
            alert(result.getDetailedMessage());
          } else {
            var chart = new google.visualization.PieChart(document.getElementById('chart_div'));
            chart.draw(result.getDataTable(), {width: 400, height: 240});
          }
        });
      }
    </script>
  </head>

  <body>
    <div id="chart_div"></div>
  </body>
</html>
```

# Mapping Your Database #
Before you can start querying against your database, you must tell MC\_Google\_Visualization which fields and tables will be allowed be queried against.  Any fields or tables in your database will **not** be exposed to visualization queries and will throw an error if attempted.
## Entities ##
You make your database tables and fields available for visualization queries by defining entities.  An entity is one or more joined database tables that expose a set of fields that can be queried against.  Fields can be simple database fields, the result of SQL functions, or use PHP callback functions to calculate the value.  Entities are defined by call `addEntity($name, $spec)` with the name of the entity (to be used in the "from" clause of queries) and a spec array defining the table, fields, and joins that make up the entity.
### Entity Spec Array ###
| **Key** | **Type** | **Description**|
|:--------|:---------|:---------------|
| table   | string   | _optional_ - The database table to map the entity against.  If this is not provided, the entity name will be used by default |
| fields  | array    | Array of $field\_name => $field\_spec pairs that define which fields will be exposed for this entity. |
| joins   | array    | Array of $join\_name => $join\_sql pairs that define the join statements that some entity fields might require.  The fields can define which join they require by providing a "join" option that matches a "join\_name" provided here. |
| where   | string   | _optional_ - an extra condition that will automatically be added to the "where" clause whenever this entity is used. |

Each field defined in the entity also has a spec array that defines the mapping for each field.
### Field Spec Array ###
| **Key** | **Type** | **Description** |
|:--------|:---------|:----------------|
| field   | string   | The SQL expression that this entity field maps to.  If this is not provided, a callback must be given. |
| callback | callback | A PHP callback that is used to generate the value instead of pulling straight from the database.  Callback fields can also include the **fields** and **extra** keys. |
| type    | string   | The data type for the field.  This must be one of "text", "number", "boolean", "date", "datetime", "timestamp", or "time" |
| join    | string   | If this field depends on fields in a joined table, this must be set to the key of the join to include in the entities "joins" list |
| fields  | array    | Callback fields can include this option to give a set of entity fields that the callback depends on to generate its data.  Use this if the callback field is transforming data in the dataset. |
| extra   | array    | Callback fields can include this option to provide extra parameters that will be passed along to the callback. |
| sort\_field | string   | Use this option to delegate sorting to another entity field.  This option can be used to make call back fields sortable. |

## More Information ##
  * [API Documentation](http://www.mailchimp.com/labs/visualization/api/)
  * [Usage Examples](http://www.mailchimp.com/labs/visualization/examples/)
  * [Google Visualization Query Language Reference](http://code.google.com/apis/visualization/documentation/querylanguage.html)
  * [PHP PDO Reference](http://www.php.net/pdo)
  * [Query Language Differences](QueryLanguageDifferences.md)