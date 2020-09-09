# raveren\mysql
Minimal and useful MYSQL/MariaDb class used in production for more than a decade!

`composer require raveren/mysql`

## Usage:

```php
use raveren\mysql;

mysql::config(
    [
        'host'     => '...',
        'database' => '...',
        'username' => '...',
        'password' => '...',
        'init_command' => 'SET NAMES utf8mb4',
    ]
);


// and just use it anywhere now, it is all very self-explanatory:
$my_pies = mysql::getAll("SELECT * FROM user_pies WHERE users_id = 42");
// result format is: 
[
    [ 'id' => 1, 'users_id' => 42, 'pie_name' => 'apple pie' ]
    [ 'id' => 2, 'users_id' => 42, 'pie_name' => 'cherry pie' ]
];
```

### Set your own custom error handler!
```php
mysql::setErrorHandler(
    function (PDOException $e, $query, $boundValues, $boundQuery) {
        !Kint::dump(
            'You just got a mysql error, son',
            $e->getMessage(),
            $query,
            $boundValues,
            $boundQuery
        );
    }
);
```

###  Effortless parameter binding! All of these work as you'd expect:
```php
$my_pies = mysql::getAll("SELECT * FROM user_pies WHERE users_id = ?", 42 );
$my_pies = mysql::getAll("SELECT * FROM user_pies WHERE users_id = ?", [42] );
$my_pies = mysql::getAll( 
    "SELECT * FROM user_pies WHERE users_id = ? AND pie_name = ?", 
    [42, 'cherry pie']
);
$my_pies = mysql::getAll( // named parameters supported of course!
    "SELECT * FROM user_pies WHERE users_id = :user_id AND pie_name = :pie", 
    ['user_id' => 42, 'pie' => 'cherry pie']
);
```

#### THE RESULT IS ALWAYS ARRAY IN THE FORMAT YOU EXPECT IT TO BE!


### Fetch data in your usual and more exotic formats
```php
mysql::getOne("SELECT pie_name FROM user_pies WHERE user_id = ?", 1);
mysql::getRow("SELECT * FROM user_pies WHERE user_id = ?", 1);
mysql::getSingleColumn("SELECT DISTINCT pie_name FROM user_pies WHERE user_id = ?", 1);

// advanced, but you'll wonder how can anyone live without it: 
mysql::getAsAssoc("SELECT * FROM user_pies", 'id=>*'); // second parameter is format of result
$result = [
    [ 1 => ['users_id' => 42, 'pie_name' => 'apple pie']]
    [ 2 => ['users_id' => 42, 'pie_name' => 'cherry pie']]
];

mysql::getAsAssoc("SELECT * FROM user_pies", 'pie_name[]=>*');
$result = [
        'cherry pie' => [
            ['id' => 1, 'users_id' => 42],
        ],
        'apple pie' => [
            ['id' => 2, 'users_id' => 42],
        ],
];

mysql::getAsAssoc("SELECT * FROM user_pies", 'users_id[pie_name]=>pie_name;id'); 
// you should be able deduct the idea by now
$result = [
    [ 42 => [
            'cherry pie' => ['cherry pie', 1]
            'apple pie' => ['apple pie', 2]
        ]
    ]
];
```


### You've got your data manipulation functions, some examples follow
```php
mysql::updateTable(
    'user_has_pies',
    ['has_pie' => mysql::literal('NOT(has_pie)')],
    'id=2'
);
mysql::deleteRows('pies', ['id' => 23]);
mysql::addRow('bakers', [ 'baker_name' => 'Chad', 'pie_level' => 100 ]); 

// third parameter for mysql::addRow can INSERT IGNORE, REPLACE or ON DUPLICATE KEY UPDATE
// this class's got it all baby. Want to optimize chunk inserts for huge amounts of rows? Use

foreach ($data as $row) {
    mysql::addRows('table_name', array('id'=>$row['id']));
}
mysql::addRows('table_name');
```

### Want to just execute a statement?
```php
mysql::exec('SET NAMES utf8mb4');
```


## License

MIT or whatever, just use it in any way you please
