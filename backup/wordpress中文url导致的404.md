找到wordpress的wp-includes目录下的class-wp.php文件
```php
//wp-includes/class-wp.php
 
$pathinfo = isset( $_SERVER['PATH_INFO'] ) ? $_SERVER['PATH_INFO'] : '';
//修改为：
$pathinfo = isset( $_SERVER['PATH_INFO'] ) ? mb_convert_encoding($_SERVER['PATH_INFO'], 'utf-8', 'GBK') : '';
//wp-includes/class-wp.php
list( $req_uri ) = explode( '?', $_SERVER['REQUEST_URI'] );
//修改为：
list( $req_uri ) = explode( '?', mb_convert_encoding($_SERVER['REQUEST_URI'], 'utf-8', 'GBK') );
```